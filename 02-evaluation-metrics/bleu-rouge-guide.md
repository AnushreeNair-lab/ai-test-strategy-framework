
# BLEU / ROUGE evaluation guide

> Level 2 — Evaluation Metrics for LLM Systems

---

## Overview

BLEU and ROUGE are **reference-based evaluation metrics** — they measure how similar a model's output is to one or more human-written reference answers. Originally developed for machine translation (BLEU) and text summarisation (ROUGE), they are now widely used as fast, automated quality signals in LLM evaluation pipelines.

They are not measures of truth. They measure **lexical overlap**. Understanding that distinction determines whether these metrics serve you or mislead you.

In the EPAM GenAI programme — governing Private GPT deployments across FCA-regulated banking environments — achieving and sustaining 90%+ validated response accuracy required a layered evaluation approach. BLEU and ROUGE were part of that stack, but only one part.

---

## BLEU — Bilingual Evaluation Understudy

### What it measures

N-gram precision: what fraction of n-grams in the model's output also appear in the reference answer.

### How it works

```
Candidate:  "The fund reported a 4.2% return in Q3"
Reference:  "The fund delivered a 4.2% return in the third quarter"

Unigram matches: The, fund, a, 4.2%, return, in → most match
Bigram  matches: "4.2% return" → matches; "The fund" → matches
4-gram  matches: fewer — penalises paraphrase even when meaning is preserved
```

BLEU-4 combines unigram through 4-gram precision with a **brevity penalty** that discourages short outputs that cherry-pick easy n-gram matches.

```
BLEU = BP × exp( Σ wₙ × log pₙ )

  BP  = brevity penalty (1.0 if candidate ≥ reference length, else < 1.0)
  pₙ  = n-gram precision at level n
  wₙ  = weight per level, typically 1/N each
```

### Score interpretation

These benchmarks were established for machine translation. In your domain — financial services, healthcare, regulated workflows — recalibrate thresholds against human review before trusting them as programme gates.

| BLEU score | Interpretation (MT benchmark) |
|---|---|
| < 10 | Almost unusable |
| 10–19 | Hard to get the gist |
| 20–29 | Understandable with effort |
| 30–40 | Good quality |
| 40–50 | High quality |
| 50+ | Near-human quality |

### When to use BLEU

- Constrained extraction tasks where exact wording matters (regulatory field values, structured data outputs)
- Code generation where syntax precision is required
- Version regression — comparing model A vs model B on the same fixed prompt set
- Template-driven responses where paraphrase is not acceptable

### When not to use BLEU

- Open-ended generation with many valid answers
- Summarisation where paraphrasing is expected and acceptable
- Conversational AI responses
- Any task where meaning matters more than wording

---

## ROUGE — Recall-Oriented Understudy for Gisting Evaluation

ROUGE flips the lens. Instead of precision (how much of the output is in the reference), it measures **recall** — how much of the reference is covered by the output.

### ROUGE variants

**ROUGE-N** measures n-gram recall:

```
ROUGE-N = matched n-grams in output / total n-grams in reference

ROUGE-1: unigram recall  (vocabulary coverage)
ROUGE-2: bigram recall   (phrase-level coverage)
```

**ROUGE-L** finds the Longest Common Subsequence — words appearing in both texts in order but not necessarily contiguous. Better at capturing sentence-level structure without requiring exact phrase matches. Preferred for document summarisation tasks.

**ROUGE-S** allows skip-bigrams (bigrams with arbitrary gaps between words). Rarely the primary metric in practice.

### Score interpretation

| ROUGE-1 | Interpretation |
|---|---|
| 0.0 – 0.2 | Poor coverage of key content |
| 0.2 – 0.4 | Moderate — key points partially present |
| 0.4 – 0.6 | Good summary quality |
| 0.6 + | Strong coverage |

### When to use ROUGE

- Summarisation quality — ROUGE-L in particular
- Key information coverage (critical for regulatory document summarisation)
- Document Q&A where reference answers exist
- Regression testing when you have golden outputs from prior model versions

---

## BLEU vs ROUGE — quick reference

| | BLEU | ROUGE |
|---|---|---|
| Orientation | Precision | Recall |
| Question asked | How much of the output is correct? | How much of the reference is covered? |
| Best for | Constrained generation, extraction, code | Summarisation, key coverage tasks |
| Penalises | Verbose outputs (via brevity penalty) | Missing key content |
| Common variants used | BLEU-1, BLEU-4 | ROUGE-1, ROUGE-2, ROUGE-L |

---

## Going beyond: BERTScore

BLEU and ROUGE fail when the model paraphrases correctly — different words, same meaning. In financial services and healthcare contexts, where precise professional language varies naturally across documents, this is a constant problem.

**BERTScore** solves this by comparing semantic embeddings rather than exact token matches.

```
Candidate: "The portfolio delivered gains of 4.2% in the third quarter"
Reference: "The fund reported a 4.2% return in Q3"

BLEU / ROUGE:  Lower  (different tokens)
BERTScore:     Higher  (semantically equivalent)
```

Use BERTScore when paraphrasing is acceptable, when domain vocabulary naturally varies, or when meaning matters more than surface wording. It is computationally heavier than BLEU/ROUGE but significantly more aligned with human quality judgement.

---

## LLM-as-judge

For tasks where no automatic metric fully captures quality — nuanced compliance questions, multi-part responses, tone and appropriateness in client-facing AI — use a stronger LLM to evaluate outputs against a defined rubric.

```
System prompt:
  "You are an expert evaluator for financial services AI systems.
   Score the following response on a 1–5 scale for each criterion:
   correctness, faithfulness to context, and regulatory appropriateness.
   Return JSON only: { correctness: int, faithfulness: int, compliance: int, reasoning: str }"

User prompt:
  "Reference answer: {reference}
   Model output: {candidate}"
```

**Best practices — especially in programme-scale evaluation:**

- Use a stronger model than the one being evaluated
- Define your rubric precisely in the system prompt — vague criteria produce noisy, ungovernable scores
- Run each evaluation 3× and average — LLM judges have variance too
- Calibrate scores against a sample of human-reviewed outputs before using them as programme gates
- Watch for length bias — LLMs tend to favour longer, more confident-sounding answers regardless of accuracy

The GenAI platform programme at EPAM used LLM-as-judge as part of the production quality monitoring cadence — not as a pre-launch check alone, but as an ongoing evaluation signal reviewed on a sprint-by-sprint basis.

---

## Building your evaluation pipeline

```
1. Collect your golden dataset
   └─ 50–500 prompt → reference answer pairs
   └─ Cover edge cases and domain-specific vocabulary, not just clean examples
   └─ Treat this dataset as a programme asset — version-control it, review it quarterly

2. Run the candidate model
   └─ Generate outputs for all prompts
   └─ Log model name, version, temperature, timestamp — all of these affect output

3. Score automatically
   └─ ROUGE-L for summarisation and coverage tasks
   └─ BLEU-4 for constrained or extraction tasks
   └─ BERTScore for semantic tasks where paraphrase is acceptable
   └─ LLM-as-judge for nuanced quality dimensions including compliance appropriateness

4. Sample for human review
   └─ Review bottom 10% by score — find systematic failures before they reach stakeholders
   └─ Review random 5% for calibration — verify metrics are tracking real quality
   └─ Document failure patterns; feed them back into the golden dataset

5. Track over time and report
   └─ Score regressions signal prompt or model degradation
   └─ Set threshold alerts for production monitoring
   └─ Include quality metrics in programme reporting — treat them as delivery KPIs
```

---

## Common pitfalls

| Pitfall | Fix |
|---|---|
| Treating BLEU as ground truth | It is one signal. Always complement with human evaluation, especially in regulated contexts. |
| Single reference answer per prompt | Collect 3–5 references per prompt where possible — ROUGE and BLEU both improve significantly |
| Wrong metric for the task | ROUGE for summarisation, BLEU for constrained tasks — don't swap them |
| Not version-controlling the evaluation set | The dataset is a programme asset. Govern it accordingly. |
| One-time pre-launch evaluation only | Evaluation is ongoing. Model updates, prompt changes, and data drift all require re-evaluation. |
| Metric optimisation without human alignment | If you optimise for BLEU, you get BLEU-optimised output. Calibrate to human quality, then monitor the metric. |

---

*Previous: [What to Test in LLM Systems →](../01-llm-testing-foundations/what-to-test-in-llm-systems.md)*  
*Next: [RAG Testing Checklist →](../03-rag-testing/rag-testing-checklist.md)*
