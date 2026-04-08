# RAG testing checklist

> Level 2 — Retrieval-Augmented Generation Quality Gates

---

## How a RAG pipeline fails

Governing Private GPT deployments in FCA-regulated banking environments — with Claude, OpenAI, and Gemini — made one thing immediately visible: RAG systems fail in ways that are hard to see from the output alone.

A RAG system has three distinct stages. Each has its own failure modes. Each can fail silently while producing a response that looks confident.

```
User query
    ↓
[ 1. RETRIEVAL ]      — Find relevant chunks from the knowledge base
    ↓
[ 2. AUGMENTATION ]   — Assemble a prompt from query + retrieved context
    ↓
[ 3. GENERATION ]     — LLM produces a response grounded in that context
    ↓
Final answer
```

**The cardinal rule:** Test each stage independently before testing end-to-end. A correct final answer can mask broken retrieval — the model used its parametric knowledge instead of retrieved context. In regulated environments where the source of an answer matters as much as the answer itself, that distinction is not academic. It is a compliance risk.

---

## Stage 1: Retrieval quality

### Relevance

- [ ] Top-K retrieved chunks are topically relevant to the query
- [ ] Irrelevant documents are not appearing in top results
- [ ] Domain-specific vocabulary queries retrieve the right chunks — not just topically adjacent ones (critical in capital markets and healthcare where terminology is precise)
- [ ] Ambiguous queries: ranking handles ambiguity sensibly

### Recall

- [ ] All key documents needed to answer the query are retrieved
- [ ] Queries where the answer spans multiple chunks: are all necessary chunks present?
- [ ] No category of relevant documents systematically excluded — check for embedding blind spots, especially for documents ingested in different formats (PDF vs HTML vs structured data)

### Ranking quality

- [ ] Most relevant chunk ranks first or within top 3
- [ ] Ranking is stable: same query run twice returns same order
- [ ] If a reranker is in use, verify it improves over base retrieval on your domain-specific test set — don't assume

### Edge cases

- [ ] Query with no relevant document in the knowledge base → retrieval returns low-confidence results, not hallucinated context
- [ ] Very short query (1–2 words) → retrieval still returns useful chunks
- [ ] Very long query (multi-sentence) → handled without silent truncation of the query itself
- [ ] Out-of-domain query → clearly not retrieved, not returned with high confidence

### Retrieval metrics to track

| Metric | What it measures | Target |
|---|---|---|
| Precision@K | Fraction of top-K results that are relevant | ≥ 0.80 |
| Recall@K | Fraction of relevant docs found in top-K | ≥ 0.90 |
| MRR | Mean Reciprocal Rank — how high is the first relevant result? | ≥ 0.80 |
| NDCG | Normalized ranking quality | ≥ 0.75 |

---

## Stage 2: Context augmentation

### Context assembly

- [ ] Retrieved chunks are inserted into the prompt without silent truncation
- [ ] Chunk ordering in the prompt is consistent — highest-ranked first, or time-ordered — and the policy is documented
- [ ] Context window limits are respected; overflows are handled explicitly, not silently dropped
- [ ] Source metadata (document title, date, regulatory reference number if applicable) is preserved if the system uses it for citations

### Prompt construction

- [ ] System prompt explicitly instructs the model to answer from provided context only — not from parametric knowledge
- [ ] Prompt format is consistent across all query types
- [ ] Retrieved document content cannot inject instructions into the prompt — prompt injection via RAG is a real attack vector in enterprise deployments
- [ ] Prompt structure survives edge cases: empty context, single chunk, maximum chunk count

In the regulated financial services deployments I governed, the system prompt instruction to answer only from context was a responsible AI control — not just a quality preference. It was documented, tested, and audited.

---

## Stage 3: Generation quality

### Faithfulness (grounding)

- [ ] Every factual claim in the output is traceable to retrieved context
- [ ] Model does not introduce facts from parametric knowledge not present in context
- [ ] Test with documents containing unusual, jurisdiction-specific, or deliberately non-standard values — model should reproduce them faithfully, not correct them from training memory
- [ ] When context is genuinely insufficient, model says "I don't have enough information" rather than fabricating — especially critical for compliance queries

### Answer relevance

- [ ] Answer directly addresses the question asked
- [ ] No off-topic content introduced
- [ ] Answer is appropriately scoped — not too broad, not missing required detail
- [ ] Multi-part questions: all parts addressed; none quietly dropped

### Attribution and citations

- [ ] If citing sources, citations are accurate — right document, right claim
- [ ] Section or page references (if used) are correct
- [ ] No phantom citations — no references to documents not in the retrieved set
- [ ] In regulated environments: citation integrity is a compliance matter, not just a quality preference

### Negative cases

- [ ] Query outside the knowledge base → model correctly declines, does not hallucinate
- [ ] Conflicting information in retrieved chunks → model surfaces the conflict rather than silently picking one version
- [ ] Outdated document retrieved → model uses it as provided, or flags staleness if date metadata is available
- [ ] Adversarial user input attempting to extract out-of-scope information → system prompt controls hold

### Generation metrics to track

| Metric | What it measures | How to measure |
|---|---|---|
| Faithfulness | Claims traceable to retrieved context | LLM-as-judge, NLI model |
| Answer relevance | Answer addresses the question asked | LLM-as-judge, ROUGE-L vs reference |
| Context utilisation | Fraction of relevant retrieved context actually used | Token overlap analysis |
| Hallucination rate | Percentage of outputs with unsupported claims | Human review on sampled outputs |

---

## End-to-end test scenarios

Every RAG system needs coverage across these scenarios before any production release.

| Scenario | What to verify |
|---|---|
| Direct lookup | Query phrase matches exactly in knowledge base → retrieved and answered correctly |
| Semantic match | Query uses different vocabulary than the source document → embedding retrieval works |
| Multi-hop | Answer requires combining two retrieved chunks → both present; synthesis is correct |
| Negative space | Topic not in knowledge base → model declines gracefully, does not fabricate |
| Conflicting documents | Two docs give different answers → model surfaces the conflict explicitly |
| Long document | Relevant information at the end of a long chunk → not lost to positional truncation |
| Recency conflict | Same information, two documents with different dates → more recent version prioritised |
| Adversarial context | Retrieved document contains a prompt injection attempt → system prompt controls hold; injection ignored |

---

## RAG evaluation framework (RAGAS-inspired)

Four metrics that give a complete picture of RAG pipeline quality:

```
Context precision
  = relevant chunks in top-K / total chunks retrieved
  → Are we retrieving clean signal, or noise?

Context recall
  = relevant chunks retrieved / total relevant chunks in knowledge base
  → Are we missing important context?

Faithfulness
  = claims supported by context / total claims in output
  → Is the answer grounded?

Answer relevance
  = how directly the answer addresses the question asked
  → Is the output actually useful?
```

In programme terms, these four metrics map to four stakeholder concerns: retrieval quality (engineering), coverage completeness (product), accuracy risk (compliance), and user value (business). Reporting all four gives different stakeholders the signal they need.

---

## Pre-launch RAG quality gate

Before releasing any RAG feature to production, all of the following must be documented and verified:

- [ ] Context precision ≥ 0.80
- [ ] Context recall ≥ 0.85
- [ ] Faithfulness ≥ 0.90 on the test set
- [ ] Hallucination rate < 5% on a sampled human review
- [ ] Zero prompt injection vulnerabilities identified in retrieved content path
- [ ] "No answer" behaviour verified on a set of out-of-scope queries
- [ ] Latency P95 within programme SLA
- [ ] Regression test suite passing — prompt changes don't degrade retrieval quality
- [ ] Citation integrity verified for any system surfacing source references
- [ ] Responsible AI controls (system prompt adherence, scope containment) documented and tested

---

*Previous: [BLEU / ROUGE Evaluation Guide →](../02-evaluation-metrics/bleu-rouge-guide.md)*  
*Next: [Agentic AI Validation Approach →](../04-agentic-ai/agentic-validation-approach.md)*
