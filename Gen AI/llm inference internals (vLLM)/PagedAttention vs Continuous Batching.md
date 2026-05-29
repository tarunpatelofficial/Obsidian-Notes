
---

**PagedAttention** → solves **memory**

- Problem: VRAM being wasted/fragmented
- Fix: allocate memory in small blocks on demand

**Continuous Batching** → solves **scheduling**

- Problem: GPU slots sitting idle waiting for slow requests
- Fix: insert new requests the moment a slot frees up

---

### They're a team

PagedAttention makes continuous batching **possible**.

When User A finishes and you want to insert User E immediately — you need memory available instantly for E's KV cache.

Without PagedAttention → no free contiguous memory block → can't insert E → back to waiting.

With PagedAttention → grab a few free pages instantly → insert E immediately.

---

### Analogy

- **PagedAttention** = making sure there's always a free table in the restaurant
- **Continuous Batching** = seating the next customer the moment someone leaves, not waiting for the whole group to finish

Same restaurant, two different jobs.

---
