# Configuration & data ingestion testing

> Level 3 — Quality Gates for AI System Infrastructure

---

## Why this layer matters

Most AI quality conversations focus on model outputs. Two upstream problems silently undermine everything downstream — and they are the hardest to diagnose because by the time they surface, they look like model problems.

**Bad configuration** — wrong model version, wrong temperature, wrong system prompt, wrong responsible AI controls — produces wrong behaviour with no error thrown. Nothing breaks visibly. You get subtly worse answers, unexplained accuracy drops, or policy violations with no clear cause.

**Bad data ingestion** — corrupted chunks, missed documents, encoding errors, chunking that splits mid-sentence — means the model can never answer correctly regardless of quality, because the information it needs is not retrievable.

Governing Private GPT deployments in FCA-regulated banking environments, and managing data ingestion across NLP pipelines and RAG systems at EPAM, made clear that data and configuration quality are programme governance concerns — not just engineering concerns. They belong in RAID logs, in release checklists, and in stakeholder reporting.

---

## Part 1: Configuration testing

### What "configuration" includes

```
Model settings
  ├─ Model name and version (must be pinned — never "latest" in production)
  ├─ Temperature
  ├─ Max output tokens
  ├─ Top-p, frequency penalty
  └─ Request timeout

Prompt configuration
  ├─ System prompt version (version-controlled, not hardcoded)
  ├─ Prompt templates per use case
  ├─ Context window allocation (system prompt + retrieved context + output)
  └─ Output format and structure instructions

Infrastructure config
  ├─ Vector database connection
  ├─ Embedding model name and version (pinned)
  ├─ Retrieval K — number of chunks returned
  ├─ Reranker settings (if used)
  └─ Auth and API keys (test presence and validity, never log values)

Responsible AI controls
  ├─ System prompt scope constraints
  ├─ Refusal and escalation logic
  └─ Audit logging configuration
```

### Schema validation

- [ ] Config file parses without error on startup
- [ ] All required fields present — fail fast on missing keys, do not silently use defaults
- [ ] Field types are correct (temperature is a float in range 0–2, not a string)
- [ ] Enum fields contain valid values (model name is in the approved list for the environment)
- [ ] No conflicting settings (max_tokens + system prompt tokens > context_window → raise an explicit error)

### Model version testing

- [ ] Specified model version is reachable in the target API environment
- [ ] Model version matches expected capability tier — do not silently fall back to a weaker or different model
- [ ] Test: change model version in config → system behaviour changes as expected; no silent cache hit
- [ ] Confirm that "latest" is not used in production config — always pin explicitly and document the pinned version in the release log

In regulated environments, model version is an auditable parameter. "We were running the model that was current at the time" is not an acceptable answer during an FCA review.

### Temperature and sampling

- [ ] At temperature 0.0: outputs are deterministic (same prompt → same output, verified across 5 runs)
- [ ] At temperature > 0: variance is appropriate to the task and within acceptable bounds for the use case
- [ ] High temperature on structured output tasks → format failure rate is measured and within defined thresholds
- [ ] Verify that temperature is actually being passed to the API and is not being silently overridden by a middleware layer

### System prompt version control

- [ ] System prompts stored in version control — not hardcoded in application code
- [ ] Prompt version logged alongside every API call in production
- [ ] Rollback procedure documented and tested end-to-end
- [ ] After any prompt change: regression suite runs before deployment reaches production
- [ ] Responsible AI control clauses in system prompts are flagged as change-controlled — they require additional review, not just engineering approval

### Context window budget

- [ ] System prompt tokens + max retrieved context tokens + max output tokens ≤ model context window
- [ ] Tested at maximum context load: output is not truncated
- [ ] Overflow handled explicitly: input is truncated with a logged warning — not the output

---

## Part 2: Data ingestion testing

### The ingestion pipeline

```
Raw source (PDF, HTML, database export, API feed)
    ↓
[ Extraction ]    — Pull text from the source format
    ↓
[ Cleaning ]      — Remove noise, normalise whitespace, strip boilerplate
    ↓
[ Chunking ]      — Split into retrieval-sized segments with overlap
    ↓
[ Embedding ]     — Convert chunks to dense vectors
    ↓
[ Indexing ]      — Store vectors and metadata in the vector database
```

Failures at any stage propagate silently downstream. A document that fails extraction produces no chunks. A chunk that embeds incorrectly produces a vector that never retrieves usefully. Neither throws an error visible without explicit monitoring.

In the NLP healthcare engagement I governed at EPAM — where defect reduction of ~10% was a programme target — systematic data quality monitoring at each pipeline stage was what made it measurable and sustainable.

### Extraction quality

- [ ] All source files successfully extracted — count verification: documents submitted ≥ documents extracted
- [ ] No silent failures — missing documents produce an explicit error logged to monitoring, not an empty result
- [ ] Encoding handled correctly: UTF-8, special characters, non-Latin scripts where applicable
- [ ] PDF extraction: tables, multi-column layouts, and headers are correctly linearised — not garbled into a stream of fragments
- [ ] HTML extraction: navigation, footer, cookie banners, and ads stripped; main content preserved
- [ ] Scanned PDFs: flagged for OCR processing, not silently ingested as blank documents

### Text cleaning

- [ ] Excessive whitespace normalised consistently
- [ ] Headers, footers, and page numbers removed or preserved consistently — define the policy and test against it
- [ ] Duplicate content deduplicated — the same paragraph appearing across multiple source documents is caught
- [ ] Encoding artefacts corrected (e.g., `â€™` in place of `'`, `Ã©` in place of `é`)
- [ ] Tables preserved in a readable sequential form — not destroyed into disconnected number sequences

### Chunking strategy

- [ ] Chunk size is appropriate for the retrieval use case — typically 256–1024 tokens; validated against retrieval quality, not assumed
- [ ] Chunks do not split sentences mid-stream — sentence boundary preservation is verified on a sample
- [ ] Overlap between chunks is configured (typically 10–20%) to avoid information loss at chunk boundaries
- [ ] Chunk metadata preserved: source document identifier, page number, section heading, ingestion date
- [ ] Manual spot-check: retrieve a chunk → verify it contains complete, contextually coherent sentences

### Embedding quality

- [ ] Embedding model version pinned — not "latest"
- [ ] All chunks successfully embedded: chunks submitted = vectors produced (count verification)
- [ ] Embedding dimensions match the vector database configuration — mismatch causes silent ingestion failure in some implementations
- [ ] Determinism check: embed the same text twice → vectors are identical (or within floating-point tolerance)
- [ ] Semantic coherence spot-check: semantically similar texts → cosine similarity > 0.8
- [ ] Semantic separation spot-check: semantically unrelated texts → cosine similarity < 0.3

### Index integrity

- [ ] Total vectors in index = total chunks ingested — exact count verification, not approximate
- [ ] No duplicate vectors — same chunk not ingested twice from concurrent or repeated pipeline runs
- [ ] Index is queryable immediately after ingestion — no undocumented warm-up period in production
- [ ] Metadata is stored alongside vectors and fully retrievable at query time
- [ ] Deletion: removing a source document removes all associated chunks from the index within the defined SLA

### Incremental updates

- [ ] New documents added without requiring full re-index
- [ ] Updated documents replace old versions — not duplicated alongside them
- [ ] Deleted source documents are removed from the index within the programme-defined SLA
- [ ] Concurrent ingestion: multiple documents processing simultaneously handled without corruption or race conditions
- [ ] Post-update: retrieval quality verified on a sample before the update is treated as complete

---

## Data quality metrics

These metrics belong in programme reporting, not just engineering dashboards.

| Metric | Definition | Minimum bar |
|---|---|---|
| Extraction success rate | Documents successfully extracted / total attempted | ≥ 99% |
| Chunk completion rate | Chunks ingested / expected chunks | ≥ 99.5% |
| Embedding success rate | Vectors created / chunks submitted | 100% |
| Index-source parity | Vectors in index / chunks in pipeline | 100% |
| Metadata completeness | Chunks with full metadata / total chunks | ≥ 95% |

---

## Configuration change management

A configuration change is a release. It carries the same risk as a code change and requires the same governance.

```
Change proposed (who initiates? who owns?)
    ↓
Review (engineering + programme manager; responsible AI controls → additional approver)
    ↓
Staging deployment
    ↓
Automated regression suite
    ↓
Human evaluation on a sample (required for any system prompt change)
    ↓
Production deployment
    ↓
Monitor: error rate, latency, quality metrics for minimum 24 hours
    ↓
Rollback triggered if degradation threshold exceeded
```

**Minimum governance standards:**

- [ ] Config changes require a documented review — no direct production edits
- [ ] Rollback procedure tested at least once per quarter, not just documented
- [ ] Config diff captured in deployment logs for every production change
- [ ] Alert on config load failure at startup — system does not start with partial or fallback configuration

---

## Pre-launch data and configuration quality gate

Before any AI feature releases to production, document and verify:

- [ ] 100% of source documents attempted for ingestion
- [ ] Extraction success rate ≥ 99%
- [ ] Zero encoding artefacts in a sampled 5% of chunks (manual review with sign-off)
- [ ] Chunking produces complete, coherent sentences on spot-check across document types
- [ ] Vector count matches chunk count exactly
- [ ] Metadata retrievable for all indexed chunks
- [ ] Config schema validation passing in CI on every build
- [ ] System prompt version logged and pinned in production config
- [ ] Responsible AI control clauses validated and documented
- [ ] Rollback to prior config tested end-to-end, not just documented

---

## The core insight

A model is only as good as the data it can access and the configuration it runs under.

A hallucination problem is sometimes a chunking problem. A quality regression is sometimes a prompt version mismatch from a deployment that didn't update all environments. A latency spike is sometimes a context window overflow silently truncating retrieval mid-response.

In programme governance terms: if you're seeing AI quality issues in production and you haven't audited the ingestion pipeline and the configuration state, you haven't finished your root cause analysis.

---

*Previous: [Agentic AI Validation Approach →](../04-agentic-ai/agentic-validation-approach.md)*  
*Back to: [Framework README →](../README.md)*
