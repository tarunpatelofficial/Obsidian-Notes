## PagedAttention — Deep Dive

### The problem it solves

KV cache is pre-allocated in **contiguous GPU memory** blocks.

```
Request A: needs 2048 tokens of KV cache → reserve 2048 upfront
Request B: needs 512 tokens → reserve 512 upfront
Request C: needs 4096 tokens → reserve 4096 upfront
```

Problem: you don't know sequence length upfront → over-allocate → **60-80% GPU memory wasted** (memory fragmentation)

This is exactly like RAM fragmentation in operating systems.

---

### What PagedAttention does

Borrows the **virtual memory + paging** concept from OS design.

Instead of one contiguous block:

- KV cache split into fixed-size **pages** (blocks), e.g. 16 tokens per block
- Pages allocated **on demand** as sequence grows
- Non-contiguous in memory — a block table maps logical → physical

```
Request A tokens 1-16   → Page 3
Request A tokens 17-32  → Page 7   (not contiguous, doesn't matter)
Request A tokens 33-48  → Page 1
```

---

### Why this is a big deal

||Old approach|PagedAttention|
|---|---|---|
|Memory waste|60-80%|<5%|
|Max batch size|Small|3-10x larger|
|Throughput|Baseline|2-4x higher|
|Memory fragmentation|High|Near zero|

Bigger batch = more requests served per GPU = lower cost per request.

---

### Bonus: enables memory sharing

**Parallel sampling** (generating N outputs for same prompt):

- Without PA: copy KV cache N times
- With PA: all N sequences **share the same pages** for the prompt portion — copy-on-write only when sequences diverge

Same applies to **system prompts** shared across requests.

---

### Where it lives

- **vLLM** invented and implemented this
- Now standard in most serious inference engines (TGI, SGLang, TensorRT-LLM)

---

### Mental model

> PagedAttention : KV Cache = Virtual Memory : RAM Same problem, same solution, different domain.

---

## PagedAttention — From Scratch

### Start with the real problem

You have a GPU with 40GB VRAM. You want to serve 100 users simultaneously.

Each user's conversation needs KV cache stored in VRAM. More users in VRAM at once = more throughput = cheaper to run.

---

### Why it was broken before

Imagine VRAM as a parking lot with 100 spots.

Old system: when a request arrives, you **reserve a full row of spots upfront** — even if the car only needs 2 spots right now.

```
User A arrives → reserve 50 spots (might need that much)
User B arrives → reserve 50 spots
Parking lot full. Only 2 users served.
```

But User A only used 10 spots. 40 spots **sit empty but are blocked**.

That's memory fragmentation. GPU is 80% empty but thinks it's full.

---

### What PagedAttention does

Stop reserving upfront. **Allocate one small block at a time, as needed.**

```
User A arrives → give 1 block (16 tokens)
User A grows → give another block, wherever there's space
User A grows → give another block, wherever there's space
```

Blocks don't need to be next to each other. A **block table** tracks where each user's blocks are.

```
User A → [Block 3, Block 7, Block 1]  ← scattered but tracked
User B → [Block 2, Block 9]
User C → [Block 4, Block 5, Block 8]
```

GPU memory is now packed tight. Almost zero waste.

---

### Result

Same 40GB VRAM:

- Before: serve 10 users
- After: serve 40-50 users

More users per GPU = higher throughput = lower cost.

---

### The sharing bonus

100 users, same system prompt.

Before: store that system prompt's KV cache **100 times**.

After: store it **once**, point all 100 users to the same blocks. Only create a separate copy when their responses diverge.

---

### One line summary

> GPU memory was being wasted like a badly packed suitcase. PagedAttention packs it properly.

---
