
# What to test in LLM systems

> Level 1 — LLM Testing Foundations

---

## Why AI testing is different

Governing AI programmes at EPAM — across agentic AI, NLP pipelines, generative AI platforms, and intelligent QA automation — made one thing immediately clear: traditional QA methods don't transfer cleanly to AI systems.

In traditional software, a function given the same input always returns the same output. You write a test. It passes or fails. The failure points to code.

LLMs break this contract in three fundamental ways.

**Non-determinism.** The same prompt can produce different outputs each run. You are testing a distribution, not a fixed function. Temperature, sampling, and model internals all introduce variance by design.

**No ground truth.** There is rarely one "correct" answer. Correctness is contextual, task-dependent, and sometimes subjective — especially in financial services, healthcare, and regulated environments where nuance carries legal weight.

**Emergent failure.** Bugs aren't in code. They're in prompt-model interactions, in training data distributions, in retrieved context quality, in tool-call chains. They often surface only at edge cases, at scale, or after a model version change.

This doesn't mean AI systems are untestable. It means you need a different philosophy — one built around distributions, evaluation science, and observability rather than pass/fail assertions alone.

---

## The 5 dimensions of LLM quality

Don't only test output correctness. Govern quality across all five dimensions, especially in regulated contexts where a failure in any one can carry programme risk.

| Dimension | What it covers | Example failure |
|---|---|---|
| **Correctness** | Is the answer factually right? | Model states incorrect regulatory figure, wrong date |
| **Faithfulness** | Does the answer stay grounded in provided context? | Model adds facts not in the retrieved document |
| **Instruction following** | Does it do what was asked? | Asked for structured JSON output, returns prose |
| **Safety & refusal** | Does it refuse harmful requests? Does it over-refuse benign ones? | Jailbreak succeeds in a regulated environment; or benign compliance query refused |
| **Consistency** | Are similar inputs producing similar quality outputs? | Rephrased question returns a contradictory answer in the same session |

In regulated financial services environments — FCA, MiFID, GDPR — faithfulness and consistency are not just quality dimensions. They are compliance dimensions.

---

## The full test taxonomy

### 1. Prompt behaviour testing

Test how the model responds to your prompts across variations — not just a single run.

- **Regression testing.** Save a prompt and its expected behaviour. Re-run automatically after any model or prompt change. This is your primary safety net across programme releases.
- **Boundary testing.** What happens at the edge of the context window? With empty inputs? With malformed JSON in the system prompt?
- **Paraphrase robustness.** Does semantically equivalent input produce semantically equivalent output? Test this especially for compliance-sensitive queries.
- **Instruction priority.** When system prompt and user message conflict, which wins? Define this and test it explicitly — in enterprise Private GPT deployments, this controls policy adherence.

### 2. Output format testing

If your system depends on structured output — JSON, markdown, tables, extraction schemas — test this as a first-class concern.

- Does the model output valid, parseable JSON consistently under varied input lengths?
- Does it respect field names, types, and required vs optional schema fields?
- Does format compliance degrade under long inputs or adversarial prompts?
- Does it fail gracefully when it genuinely cannot comply — returning an informative error, not malformed output?

In intelligent QA automation and agentic systems, format failures cascade. A malformed JSON output in step 1 corrupts every downstream step.

### 3. Factual accuracy and hallucination

- **Reference-based checking.** Compare output against known ground truth — essential in financial data workflows.
- **Self-consistency.** Ask the same question multiple ways. Flag contradictions across responses.
- **Citation checking.** If the model cites sources, verify they exist and that the claim matches the cited content.
- **RAG faithfulness.** In RAG-enabled systems, every claim in the output should be traceable to retrieved context. *(See the RAG testing checklist.)*

In the EPAM GenAI platform programme — Claude, OpenAI, Gemini deployments in regulated banking — achieving 90%+ validated response accuracy in production required making hallucination detection a structured, ongoing process, not a pre-launch check.

### 4. Safety and policy testing

- **Harmful content.** Does the model refuse requests for dangerous, illegal, or policy-violating content?
- **Prompt injection.** Can a user override system behaviour through crafted input? Especially critical when retrieved documents are in the context window.
- **PII leakage.** Does the model surface personally identifiable information from context it should not expose?
- **Jailbreak resistance.** Test known jailbreak patterns against your specific system prompt. Responsible AI controls require this to be documented, not assumed.

### 5. Latency and performance

- **P50/P95/P99 latency** per prompt type. Long-running prompts behave differently at the 95th percentile.
- **Token efficiency.** Are prompts producing unnecessarily long outputs? In enterprise deployments, token costs compound at scale.
- **Cost per query.** Monitor token usage as a programme metric, not just an engineering metric.
- **Timeout and degradation behaviour.** What happens when the model is slow, returns a 429, or is temporarily unavailable? Define and test the fallback path.

---

## What you cannot fully automate

Honesty about limits is part of programme governance. Automation covers the fast, repeatable, cheap checks. Human judgment remains irreplaceable for the rest.

| Test type | Automation | Why |
|---|---|---|
| Output format validation | High | Schema and regex checks are deterministic |
| Refusal on known inputs | High | Pass/fail against a fixed test set |
| Factual correctness | Partial | Requires a ground truth dataset — expensive to build and maintain at scale |
| Tone and style quality | Partial | LLM-as-judge helps but needs human calibration |
| Novel failure modes | Low | Requires adversarial imagination and domain expertise |
| Nuanced safety in regulated contexts | Low | Context-dependent; high-stakes; needs human review |
| User satisfaction | None | Requires real users, real feedback loops |

---

## Testing levels at a glance

```
Unit level
  └─ Single prompt → expected output shape or content
  └─ Format validation (JSON schema, regex match)
  └─ Known refusal and safety behaviour

Integration level
  └─ Prompt + retrieved context → correct information used
  └─ Tool calls triggered and parameterised correctly
  └─ Conversation history and memory handled properly

System level
  └─ Full user flow end-to-end
  └─ Latency under realistic concurrent load
  └─ Graceful degradation when upstream services fail

Evaluation level — ongoing, not one-time
  └─ BLEU / ROUGE / BERTScore on a held-out dataset
  └─ LLM-as-judge on sampled production traffic
  └─ Human review on flagged or low-scoring outputs
  └─ Stakeholder-visible quality reporting on a programme cadence
```

---

## Key principles

**Test distributions, not just instances.** One good output doesn't mean the system is reliable. Run test cases multiple times and measure variance across runs — especially after model or prompt changes.

**Build a golden dataset early.** A curated set of prompt → expected behaviour pairs is your most valuable test asset. Govern it, version-control it, and grow it deliberately. In programme terms, it is your definition of done for AI quality.

**Version prompts like code.** A prompt change is a release. Treat it that way — review, regression run, staged rollout, rollback procedure. The GenAI platforms I governed at EPAM required this discipline to maintain 90%+ response accuracy in production.

**Separate concerns.** Test retrieval separately from generation. Test generation separately from tool use. Mixed failures are the hardest to diagnose, and in a matrixed programme, they're also the hardest to own.

**Monitor in production.** No pre-launch test suite catches everything. Observability is a programme requirement, not an engineering optional. The failures you don't design tests for are the ones you need dashboards for.

---

*Next: [BLEU / ROUGE Evaluation Guide →](../02-evaluation-metrics/bleu-rouge-guide.md)*
