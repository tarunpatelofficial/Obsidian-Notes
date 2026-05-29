## Evaluating an LLM Model

LLM evaluation covers several dimensions depending on your use case:

---

### 1. **Benchmark Performance**

Standardized tests give a quick apples-to-apples comparison:

- **Reasoning**: MMLU, HellaSwag, ARC, BIG-Bench
- **Math**: GSM8K, MATH
- **Code**: HumanEval, SWE-Bench
- **Instruction following**: IFEval, MT-Bench

These are useful for baselines but don't always reflect real-world performance.

---

### 2. **Task-Specific Evaluation**

Test the model on _your_ actual use case:

- Build a **golden dataset** — a set of inputs with expected outputs
- Score outputs on correctness, completeness, format compliance
- Run regression tests when switching models

---

### 3. **Human Evaluation**

- **Preference testing (A/B)**: Show two model outputs, ask which is better
- **Likert scale ratings**: Rate quality 1–5 on dimensions like helpfulness, clarity, accuracy
- Best for nuanced tasks (creative writing, tone, empathy)

---

### 4. **LLM-as-Judge**

Use a stronger model (e.g. Claude Opus, GPT-4) to score outputs automatically:

- Scales well for large eval sets
- Define a rubric: accuracy, groundedness, conciseness, safety
- Watch for bias — judge models tend to favor verbose or their own outputs

---

### 5. **Key Quality Dimensions**

|Dimension|What to measure|
|---|---|
|**Accuracy**|Factual correctness, hallucination rate|
|**Instruction following**|Does it do exactly what was asked?|
|**Reasoning**|Multi-step logic, chain-of-thought quality|
|**Groundedness**|Does it stay within context (RAG use cases)?|
|**Safety**|Refusal of harmful prompts, bias|
|**Calibration**|Does it know when it doesn't know?|
|**Latency/Cost**|Tokens/sec, price per 1M tokens|

---

### 6. **Practical Tips**

- **Red-teaming**: Adversarially probe for failure modes, jailbreaks, edge cases
- **Consistency testing**: Run the same prompt multiple times — are outputs stable?
- **Length/format compliance**: Does it follow output structure instructions reliably?
- **Context window stress tests**: How does quality degrade at long contexts?

---

### Tools to help

- **Promptfoo** — open-source LLM eval framework
- **LangSmith** — tracing + eval for LangChain apps
- **Braintrust** — eval + logging platform
- **Evals by OpenAI / Anthropic's model card benchmarks** — reference points

The gold standard is combining **automated benchmarks + task-specific evals + human review** — no single method captures the full picture.


---

## Real-World LLM Eval (What Actually Matters)

**Skip generic benchmarks** — MMLU scores mean nothing for your use case.

### What works:

**1. Build your own eval set**

- 50–200 real inputs from your use case
- Define pass/fail criteria upfront
- Run every model candidate against it

**2. LLM-as-Judge**

- Use Claude Opus or GPT-4 as scorer
- Give it a strict rubric (don't ask "is this good?" — ask "did it follow format? is claim X correct?")
- Cheap, fast, scales

**3. Metrics that matter in prod**

- Hallucination rate on your domain
- Instruction-following reliability (does it always return JSON when asked?)
- Latency p50/p95
- Cost per 1k requests

**4. Vibe evals first**

- Manually review 20–30 outputs before automating anything
- You'll catch failure patterns faster than any script

### Tools worth using:

- **Promptfoo** — free, local, fast
- **Braintrust** — better for teams/prod logging

### Rule of thumb:

> Real task + real data + clear rubric = 80% of what you need. Everything else is noise.
