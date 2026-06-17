### Foundation (must know cold)

- Tokenization, embeddings, attention mechanism
- KV Cache — what it is, why it matters for inference speed
- Context window vs effective context (they're not the same)

### Inference & Serving

- **PagedAttention** — how vLLM uses it to maximize GPU memory
- **Continuous batching** — vs static batching, why it matters at scale
- **Quantization** — INT4/INT8/GPTQ/AWQ tradeoffs
- **Speculative decoding** — draft model + verification
- Tensor parallelism vs pipeline parallelism

### Evaluation (deep)

- Perplexity — what it measures and its limits
- RAGAS — RAG-specific eval framework (faithfulness, answer relevancy, context precision)
- LLM-as-judge — bias, calibration, rubric design
- Evals for hallucination specifically — FactScore, SelfCheckGPT
- Latency profiling — TTFT (time to first token) vs TBT (time between tokens)

### RAG (beyond basics)

- Chunking strategies — fixed, semantic, recursive
- Dense vs sparse retrieval (BM25 vs embeddings vs hybrid)
- Reranking — cross-encoders, Cohere Rerank
- HyDE — hypothetical document embeddings
- Context stuffing vs summarization tradeoffs
- Indexing strategies — HNSW, IVF-flat in vector DBs

### Advanced RAG patterns

- Agentic RAG — query routing, self-reflection loops
- Multi-hop retrieval
- Corrective RAG (CRAG)
- Chunk + full doc retrieval (small-to-big)

### Fine-tuning

- When NOT to fine-tune (most people do this wrong)
- LoRA / QLoRA — how it works, when to use
- DPO vs RLHF — practical difference
- Catastrophic forgetting

### Production concerns

- Prompt caching (Anthropic/OpenAI both support this — saves cost)
- Cold start latency
- GPU memory math — can you fit the model?
- Observability — tracing, logging, eval in prod

---

**Learning order:** Inference → RAG internals → Eval → Fine-tuning → Prod
