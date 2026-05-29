## The Trio Working Together

### Setup

You run an LLM API. 4 users send requests simultaneously. GPU has 12 memory pages available.

```
User A → short request, will generate 10 tokens
User B → long request,  will generate 80 tokens
User C → medium request, will generate 40 tokens
User D → waiting in queue
```

---

### Step 1 — Requests arrive

**PagedAttention** allocates pages on demand:

```
User A → assigned Page 1
User B → assigned Page 2
User C → assigned Page 3

No upfront reservation. 9 pages still free.
```

**Continuous Batching** fills GPU:

```
Active batch: [A, B, C]
User D waits in queue
```

---

### Step 2 — Tokens start generating

**KV Cache** kicks in:

```
Token 1 generated for A, B, C
→ K/V computed and stored in their pages
→ saved in pages 1, 2, 3

Token 2 generated
→ reuse stored K/V from token 1
→ only compute new token's K/V
→ append to their pages
```

As sequences grow, PagedAttention adds pages:

```
After token 16:
User A → [Page 1]
User B → [Page 2, Page 5]   ← new page added automatically
User C → [Page 3, Page 6]   ← new page added automatically
```

---

### Step 3 — User A finishes at token 10

**Continuous Batching** acts immediately:

```
User A done → slot freed
User D inserted instantly
Batch: [D, B, C]  ← no waiting
```

**PagedAttention** frees A's memory instantly:

```
Page 1 released → back to free pool
User D → assigned Page 1 immediately
```

**KV Cache** starts fresh for D:

```
User D token 1 → compute K/V, store in Page 1
User D token 2 → reuse Page 1, append new K/V
```

---

### Step 4 — User C finishes at token 40

Same thing repeats:

```
Pages 3, 6 released
Next user in queue inserted immediately
Their pages allocated from free pool
KV Cache starts for them
```

---

### Full picture

```
KV Cache          → each token reuses past computation
                     decode is fast

PagedAttention    → memory freed and allocated instantly
                     no fragmentation, no waste
                     makes insertion of new requests possible

Continuous        → slots never idle
Batching            new request in the moment old one finishes
                     GPU always at 90%+ utilization
```

---

### One line each

- **KV Cache** → don't recompute what you already know
- **PagedAttention** → don't waste memory on what you might need
- **Continuous Batching** → don't waste time waiting for slow requests

---

They don't work without each other. That's why vLLM became the standard — it implemented all three together.