# Architecture — PV AI Toolkit

How the three tools are built and how they fit together. The rendered diagram is `pv-copilot-architecture.svg`; this note is the written companion.

## What this is
Three independent point solutions, each solving one bounded task in drug-safety work, deployed behind one Streamlit hub. They are presented and used separately; they also share typed input/output contracts, so they *could* be composed into an end-to-end case flow (see "Optional composition" below).

## The three tools (built and deployed)
1. **AE Triage** — free-text adverse-event report → structured, seriousness-graded record (ICH E2A). FastAPI + Pydantic; routed to Haiku; batch endpoint runs concurrently.
2. **Regulatory Q&A** — question → answer grounded in ICH/FDA guidance with sources, or an honest refusal when the answer isn't in the documents. RAG over ChromaDB.
3. **Safety Signals** — medicine → a tool-use agent pulls openFDA FAERS and computes disproportionality statistics (**PRR, ROR, and IC** — a BCPNN shrinkage measure) with 95% intervals and signal flags.

## Cross-cutting (built)
- **Evaluation** — each system has a golden-set harness: LLM-as-judge for the RAG triad; deterministic label match plus detector precision/recall for the agent. Gold sets are currently small (8 Q&A, 9 drugs) and being expanded.
- **Demo mode** — every hub page has a live/cached toggle, so it renders with no API key or a cold backend.
- **Model routing** — Haiku on the high-volume extraction path, Opus for the signal judgement.

## Shared contracts
Each tool returns typed, validated output (Pydantic): AE Triage emits `{seriousness, suspected_drug, event, summary, confidence}`; Safety Signals emits a validated verdict plus a per-reaction statistics table; Regulatory Q&A emits `{answer, sources}`.

## Optional composition (design sketch — NOT built)
Because the outputs are typed, the tools *could* be chained into a single case lifecycle — intake → triage → (if serious) FAERS signal check → ICH-criteria check → draft narrative → human review — with a per-case audit log recording each step. **This orchestrator and the audit log are a design sketch, not implemented.** The diagram shows this as the optional path. What ships today is the three tools, used independently.

## On EBGM
The agent computes **IC**, not EBGM. True EBGM needs an empirical-Bayes prior fit across the whole FAERS database and can't be computed from per-query API calls; IC is the honest, computable stand-in that serves the same purpose. Adding real EBGM from an ingested FAERS extract is planned, not done.

## Stack
FastAPI · Pydantic · Streamlit · ChromaDB · openFDA · Anthropic API · Railway · Streamlit Community Cloud.

---
_Uses only public data. A demonstration, not a validated system; not for clinical or regulatory use._
