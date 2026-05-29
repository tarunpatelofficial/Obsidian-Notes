## KV Cache — Deep Dive

### What is it?

During inference, transformer attention computes **Key** and **Value** matrices for every token.

Without cache: every new token regenerates K/V for **all previous tokens** — O(n²) compute.

With cache: you store K/V from previous tokens, only compute for the **new token**.

```
Prompt: "What is the capital of France?"
Token 1 → compute K/V, store it
Token 2 → reuse Token 1's K/V, compute only Token 2's
Token 3 → reuse 1+2, compute only Token 3's
...
```

### Why it matters

- **Without it**: generating 500 tokens = 500 full attention passes
- **With it**: generating 500 tokens = 1 prefill pass + 500 cheap decode steps
- Speed difference: **10x–50x faster** decode phase

### Two phases to understand

|Phase|What happens|KV Cache role|
|---|---|---|
|**Prefill**|Process entire prompt at once|K/V computed + stored|
|**Decode**|Generate one token at a time|K/V reused from cache|

TTFT (time to first token) = prefill cost TBT (time between tokens) = decode cost — this is where cache saves you

### The catch — memory

- KV cache lives in GPU VRAM
- Size = `layers × heads × seq_len × dtype_bytes`
- Long contexts eat VRAM fast — this is exactly why **PagedAttention** was invented

### Prompt caching in APIs

Anthropic + OpenAI both expose this:

- Send same system prompt repeatedly → cached → cheaper + faster
- Anthropic charges ~10% of normal input cost on cache hits

### Rule of thumb

> KV cache is why decode is fast. VRAM is why it's limited. PagedAttention is the fix for the limitation.

