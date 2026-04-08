# AI Test Strategy Framework

> The structured documentation of a 3-level AI testing curriculum — designed and delivered at EPAM Systems to transition 15+ manual QA professionals into AI engineers, reducing vendor dependency by ~30%.

![Generative AI](https://img.shields.io/badge/Generative%20AI-4B5563?style=flat-square)
![RAG](https://img.shields.io/badge/RAG-1D6A96?style=flat-square)
![LLM Testing](https://img.shields.io/badge/LLM%20Testing-0F6E56?style=flat-square)
![Program Management](https://img.shields.io/badge/Program%20Management-7C3AED?style=flat-square)

---

## Origin

This framework didn't start as documentation. It started as a delivery problem.

At EPAM, I was governing 6–8 concurrent AI-enabled workstreams — agentic AI, NLP pipelines, generative AI platforms, intelligent QA automation — across regulated financial services, healthcare, and retail clients. The teams executing these workstreams were experienced QA professionals. The problem: traditional QA methods don't transfer cleanly to AI systems.

Outputs are probabilistic. There's no ground truth. Failure modes are subtle — hallucination, retrieval drift, tool misuse, reasoning loops. And there's no off-the-shelf playbook for testing them at programme scale.

So I built one.

The **3-Level AI Testing Curriculum** progressed practitioners from LLM evaluation basics through RAG-specific testing to full agentic system validation — with a parallel track for senior engineers and architects on AI business and risk evaluation. This repository is that curriculum, written up as a framework anyone can use.

---

## What's inside

```
Level 1 — LLM Testing Foundations
  ├─ Why AI testing is fundamentally different
  ├─ The 5 dimensions of LLM quality
  ├─ Full test taxonomy: prompt behaviour, output format,
  │   hallucination, safety, latency
  └─ What you can and cannot automate

Level 2 — Evaluation Metrics & RAG
  ├─ BLEU / ROUGE — mechanics, score interpretation, pitfalls
  ├─ BERTScore and LLM-as-judge patterns
  └─ RAG-specific testing: retrieval quality, context
      augmentation, generation faithfulness, pre-launch gates

Level 3 — Agentic AI & Production Systems
  ├─ 6 agentic failure modes with real examples
  ├─ Tool-call validation and chain-of-thought auditing
  ├─ Loop detection, scope containment, multi-step reasoning
  └─ Configuration and data ingestion quality gates
```

---

## Repository structure

```
ai-test-strategy-framework/
├── README.md
├── 01-llm-testing-foundations/
│   └── what-to-test-in-llm-systems.md
├── 02-evaluation-metrics/
│   └── bleu-rouge-guide.md
├── 03-rag-testing/
│   └── rag-testing-checklist.md
├── 04-agentic-ai/
│   └── agentic-validation-approach.md
└── 05-config-and-data/
    └── config-data-ingestion-testing.md
```

---

## Who this is for

| Role | Start here |
|---|---|
| QA / SDET transitioning into AI | Level 1 — LLM Testing Foundations |
| ML / AI engineer building RAG | Level 2 — RAG Testing Checklist |
| Engineer or PM governing agentic systems | Level 3 — Agentic Validation |
| Programme manager setting release gates | RAG pre-launch gate + Config quality gate |
| Team lead building an AI testing capability | All 3 levels as a structured curriculum |

---

## Context: where this was applied

- **Agentic AI programmes** — NLP pipelines, autonomous agent frameworks, and AI feedback systems across Capital Markets, Healthcare, and Retail at EPAM
- **Generative AI platforms** — Private GPT deployments (Claude, OpenAI, Gemini) in FCA-regulated banking environments
- **Google engagement** — AI initiative governance across intelligent QA automation and NLP systems
- **Internal capability uplift** — structured delivery transitioning 15+ QA professionals to AI engineers, with a separate track for senior architects on AI risk evaluation

---

## Docs

- [What to Test in LLM Systems →](./01-llm-testing-foundations/what-to-test-in-llm-systems.md)
- [BLEU / ROUGE Evaluation Guide →](./02-evaluation-metrics/bleu-rouge-guide.md)
- [RAG Testing Checklist →](./03-rag-testing/rag-testing-checklist.md)
- [Agentic AI Validation Approach →](./04-agentic-ai/agentic-validation-approach.md)
- [Config & Data Ingestion Testing →](./05-config-and-data/config-data-ingestion-testing.md)

---

## About the author

**Anushree Nair** — Program Manager, AI & Digital Transformation, 15 years.

Governed AI programmes at EPAM (Google engagement, GenAI platforms, NLP pipelines) and Schroders PLC (£526B AUM, FCA/MiFID regulatory platform). M.Tech in AI/ML, BITS Pilani. Currently authoring a research paper on AI-driven financial risk forecasting.

[LinkedIn](https://linkedin.com/in/anushreenair) · [Email](mailto:s.anushreenair@gmail.com)

---

*Topics: `generative-ai` · `llm-testing` · `rag` · `agentic-ai` · `evaluation-metrics` · `bleu` · `rouge` · `quality-assurance` · `program-management` · `ai-testing` ·  `financial-services`*
