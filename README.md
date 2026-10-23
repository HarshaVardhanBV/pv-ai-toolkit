# Pharmacovigilance AI Toolkit

Three focused AI tools for drug-safety (pharmacovigilance) teams — each takes one well-defined job in the work of monitoring medicines for harmful effects, and does it end-to-end, evaluated, and deployed.

**▶ Live app: https://harsha-pharma-portfolio.streamlit.app**

Not a single "solve pharmacovigilance" platform — that space is too broad for any one tool. These are three sharp point solutions that share a common engineering method (typed outputs, real evaluation, task-based model routing, and a demo mode that works even without a live connection).

![App screenshot](docs/screenshot.png)

---

## The three tools

| Tool | What it does | Stack |
|---|---|---|
| **AE Triage** | Turns a free-text patient safety report into a structured, seriousness-graded record (ICH E2A criteria). | FastAPI · Pydantic · Claude (Haiku) |
| **Regulatory Q&A** | Answers plain questions about reporting rules, quoted from official ICH/FDA guidance — and honestly declines when the answer isn't in the documents. | RAG · ChromaDB · Claude |
| **Safety Signals** | Takes a medicine and computes real disproportionality statistics (PRR / ROR / IC) on public FDA FAERS data to flag side effects reported more than expected. | openFDA · tool-use agent · Claude (Opus) |

Each page has a **live / cached** toggle, so the app always renders a result — even with no API key or a cold backend.

---

## Evaluation (measured, not asserted)

| System | Metric | Result |
|---|---|---|
| Regulatory Q&A | RAG-triad, LLM-as-judge over an 8-question gold set | **4.79 / 5** answer quality · 4.57 retrieval |
| Safety Signals | Assessment vs. known-correct label (9 drugs) | **89% (8/9)**, rationale groundedness **5.0 / 5** |
| Safety Signals | Disproportionality detector (precision / recall / F1) | 1.00 / 0.67 / 0.80 |
| AE Triage | Output contract | Typed & Pydantic-validated against ICH E2A |

The two detector misses are shown, not hidden: a brand-name lookup (Lipitor → no generic match) and a genuine signal-in-noise case (aspirin).

_The gold sets are deliberately small for now (8 Q&A questions, 9 drugs) — enough to be discriminating, not yet enough to be conclusive. Expanding to 30+ cases with reaction-level labels is in progress._

---

## Architecture

The tools stand alone but share typed contracts, so they can be composed into an end-to-end case workflow when needed:

![Architecture](pv-copilot-architecture.svg)

---

## Run it locally

`02-reg-rag` is a **git submodule** (it lives in its own repo), so clone with `--recurse-submodules` or that folder will be empty:

```bash
git clone --recurse-submodules https://github.com/HarshaVardhanBV/pv-ai-toolkit.git
cd pv-ai-toolkit/portfolio-hub

python3 -m venv .venv
source .venv/bin/activate          # Windows: .venv\Scripts\activate
pip install -r requirements.txt
python3 -m streamlit run Home.py
```

_Already cloned without it? Run `git submodule update --init` from the repo root._

The app opens at `localhost:8501`. Switch any page to **Cached (demo)** to run it with no API key. For live mode, add your `ANTHROPIC_API_KEY` to `.streamlit/secrets.toml`.

---

## Repository layout

```
01-ae-triage-api/     AE Triage — FastAPI service (routing + concurrent batch)
02-reg-rag/           Regulatory Q&A — RAG pipeline + API
03-evals/             Evaluation harness (LLM-as-judge, RAG triad)
04-safety-agent/      Safety Signals — tool-use agent + disproportionality + eval
portfolio-hub/        Streamlit app tying the three tools together
pv-copilot-architecture.svg   Architecture diagram
```

---

---

_Uses only public data (openFDA FAERS, ICH/FDA guidance). A demonstration, not a validated system; not for clinical or regulatory decisions._

Built by **Harsha Vardhan B V** · [LinkedIn](https://www.linkedin.com/in/harsha-vardhan-b-v)

_Stack: Python · FastAPI · Streamlit · ChromaDB · openFDA · Anthropic API; deployed on Streamlit Community Cloud and Railway._
