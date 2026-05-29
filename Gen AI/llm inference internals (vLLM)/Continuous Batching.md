## Continuous Batching — From Scratch

### Old way: Static Batching

GPU processes requests in batches.

```
Batch: [User A, User B, User C]

User A finishes at token 50
User B finishes at token 200
User C finishes at token 180
```

Old system: **entire batch waits for the slowest request.**

User A done at token 50 — GPU sits idle for User A's slot until B and C finish.

That's wasted compute. GPU is never fully utilized.

---

### Continuous Batching fix

As soon as one request finishes, **immediately insert a new one.**

```
Step 1: [User A, User B, User C]
Step 2: [User A, User B, User C]
...
User A finishes →
Step 51: [User D, User B, User C]  ← D inserted immediately
```

GPU never waits. Always full. Always computing.

---

### Why this matters in prod

Real traffic is unpredictable:

- Some prompts generate 50 tokens
- Some generate 2000 tokens
- They arrive at random times

Static batching → GPU utilization 40-60% Continuous batching → GPU utilization 90%+

Same hardware, 2x throughput.

---

### How it works with PagedAttention

They're designed together:

- **PagedAttention** manages memory — new request gets blocks instantly
- **Continuous batching** manages scheduling — new request inserted instantly

Without PagedAttention, you can't do continuous batching efficiently — no memory available for the incoming request.

---

### Mental model

```
Static batching   = a bus that waits for everyone before leaving
Continuous batching = a metro that leaves every 2 minutes
```

---

### Where it lives

vLLM, TGI, SGLang all implement this by default. It's now table stakes for any serious inference engine.

---

## Static vs Continuous Batching

### Setup

You run an LLM API. 6 users send requests.

Each request generates different number of tokens:

```
User A → 10 tokens
User B → 10 tokens
User C → 10 tokens
User D → 80 tokens
User E → 80 tokens
User F → 80 tokens
```

---

### Static Batching

GPU takes batch of 3:

```
Batch 1: [A, B, C]
```

A, B, C all finish at token 10. GPU is done with Batch 1.

```
Batch 2: [D, E, F]
```

D, E, F need 80 tokens each. GPU runs all 80 steps.

**Total time = 10 steps + 80 steps = 90 steps**

---

### Now make it realistic

```
Batch 1: [A, B, D]

A finishes at step 10 ✓
B finishes at step 10 ✓
D still going...
D still going...
D still going...
D finishes at step 80 ✓
```

A and B finished at step 10. GPU spent steps 11–80 computing **only D**. A and B's slots = **idle for 70 steps**.

Meanwhile E, F, C are **waiting in queue** even though 2 GPU slots are free.

That's the problem. Free slots, wasted time, users waiting.

---

### Continuous Batching

Same scenario:

```
Step 1-10:  [A, B, D]
A finishes → E inserted immediately
Step 11-20: [E, B, D]  ← wait, B also finished
B finishes → C inserted immediately  
Step 11-20: [E, C, D]
...
```

No slot ever sits idle. As soon as one request finishes, next one jumps in.

```
Static:     90 steps, users waiting
Continuous: ~80 steps, no waiting, higher throughput
```

---

### One line

> Static batching = slowest person in the group holds everyone up. Continuous batching = you leave the moment your seat is free, don't wait for anyone.

