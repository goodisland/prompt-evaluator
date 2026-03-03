# PROMPT-EVALUATOR

**AI Prompt Quality Evaluation Ecosystem** — Built for the Mistral Hackathon

> A complete toolkit for evaluating, scoring, and improving AI prompts — in real-time, in batch, and right inside your browser.

## Demo

[![Demo Video](https://img.youtube.com/vi/kcLIAN3F1IY/maxresdefault.jpg)](https://youtu.be/kcLIAN3F1IY)

> [Hackathon Submission](https://hackiterate.com/mistral-worldwide-hackathons?tab=submissions&submission=245ade06-09c8-4b8c-8ac8-c9f71bad0fe7)

---

## Team

**PROMPT-HACKERS** — Mistral Hackathon 2026

Huge thanks to every teammate who made this possible:

- [@nowex35](https://github.com/nowex35) — Led the frontend development, building the core UI and glassmorphism experience
- [@shu-ozawa](https://github.com/shu-ozawa) — Built `eval-score` and `auto-eval-prompt`, powering annotation and batch evaluation
- [@goodisland](https://github.com/goodisland) (me) — Led the model training pipeline, fine-tuning and deploying the custom Mistral evaluator

---

## What Is This?

PROMPT-EVALUATOR is a multi-component system that helps users write better prompts for AI models. It evaluates prompts across 6 structured criteria, provides real-time feedback, visualizes strengths and weaknesses, and learns from human-annotated data through fine-tuning.

The system spans a web app, a batch CLI tool, a browser extension, a fine-tuned model, and an analysis dashboard — all sharing the same evaluation framework.

---

## Evaluation Criteria

Every component uses the same 6-metric framework:

| Criterion | Scale | Description |
|-----------|-------|-------------|
| **Clarity** | 1–5 | Are the instructions clear and unambiguous? |
| **Role Assignment** | Yes/No | Is a role or persona given to the LLM? |
| **Prompt Structure Order** | 1–3 | Does it follow Role → Context → Task → Output? |
| **Target Audience** | 1–5 | Is the intended audience specified? |
| **Scope Limitation** | 1–5 | Is the response scope appropriately constrained? |
| **Output Format & Examples** | 1–5 | Are format and concrete examples provided? |

**Overall Score:** Normalized average across all criteria (0–100)

---

## Repositories

### [`prompt-evaluator`](https://github.com/PROMPT-HACKERS/prompt-hackers) — Web Application

Interactive FastAPI-based web app for real-time prompt evaluation.

- Real-time scoring as you type (debounced)
- Radar chart visualization of 6 criteria
- Detailed feedback with rewrite suggestions
- 3D glassmorphism UI powered by Three.js
- Multi-language support: English, Japanese, French
- Powered by Mistral AI (`ministral-3b-2512` + `mistral-large-latest`)

```bash
cd prompt-evaluator
pip install -r requirements.txt
uvicorn server:app --reload
```

**Key endpoints:**
- `POST /api/{lang}/evaluate` — Quick scoring
- `POST /api/{lang}/detailed-feedback` — In-depth analysis with suggestions

---

### [`auto-eval-prompt`](https://github.com/PROMPT-HACKERS/auto-eval-prompt) — Batch Evaluation CLI

Command-line tool for evaluating prompt datasets in bulk.

- Batch process JSON datasets via Mistral API or custom endpoints
- Configurable retry logic, rate limiting, progress tracking
- Optional Google Sheets (GAS API) result submission
- Output format compatible with `eval-score` analysis tools

```bash
cd auto-eval-prompt
uv run python auto_eval.py data/prompts.json --model ministral-3b-2512
```

**Options:**
- `--model` — Mistral model to use
- `--url` — Custom evaluation server endpoint
- `--submit` — POST results to GAS endpoint
- `--delay` — Rate limiting between API calls
- `--retries` — Retry count for failed evaluations

---

### [`training-model`](https://github.com/PROMPT-HACKERS/training-model) — Fine-tuned Model

Trains and deploys a custom prompt evaluation model using supervised fine-tuning.

- Base: `Ministral-3B-Instruct-2512` (Unsloth-optimized)
- LoRA fine-tuning (rank 16, lr 2e-4) with W&B tracking
- Training input: JSONL files in `messages` format
- Deployable as a FastAPI service on HuggingFace Spaces

**Workflow:**
1. Annotate prompts with `eval-score/annotator.html`
2. Fine-tune with `train_mistral_unsloth_from_jsonl.py`
3. Serve via `hf_space/app.py` (`POST /evaluate`)

```bash
cd training-model
python train_mistral_unsloth_from_jsonl.py
```

---

### [`eval-score`](https://github.com/PROMPT-HACKERS/eval-score) — Annotation & Analysis Dashboard

Static HTML tools for human annotation and result visualization — no server required.

| Tool | Purpose |
|------|---------|
| `annotator.html` | Create and annotate prompt datasets |
| `index.html` | Score individual prompts manually |
| `analyze.html` | Aggregated results across evaluators |
| `individual.html` | Per-evaluator performance breakdown |

Just open any file in a browser. Exports/imports JSON compatible with `auto-eval-prompt` output.

---

### [`eval_prompt_extension`](https://github.com/PROMPT-HACKERS/eval_prompt_extension) — Browser Extension

Chrome extension that overlays real-time prompt evaluation on AI chat interfaces.

**Supported platforms:**
- ChatGPT
- Claude.ai
- Google Gemini
- Mistral Chat

**Features:**
- Real-time radar chart overlay as you type
- Two modes: Local API (connect to `prompt-evaluator`) or direct Mistral API
- Draggable overlay window with glassmorphism UI
- Multi-language support

**To install:**
1. Open `chrome://extensions`
2. Enable Developer Mode
3. Load unpacked from `eval_prompt_extension/prompt-evaluator-extension/`
4. Configure API key or local server URL in the popup

---

## System Architecture

```
┌─────────────────────────────────────────┐
│           DATA ANNOTATION               │
│   eval-score/annotator.html             │
│   → Labeled prompt datasets (JSON)      │
└────────────────┬────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────┐
│            EVALUATION LAYER             │
│                                         │
│  prompt-evaluator   — Web UI            │
│  auto-eval-prompt   — Batch CLI         │
│  eval_prompt_extension — Browser ext    │
│  training-model     — Fine-tuned API    │
└────────────────┬────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────┐
│         ANALYSIS & VISUALIZATION        │
│   eval-score/analyze.html               │
│   eval-score/individual.html            │
└─────────────────────────────────────────┘
```

All components share:
- The same `config.json` evaluation criteria
- A common `eval_prompt.md` system prompt
- Compatible JSON data formats

---

## Tech Stack

| Layer | Technologies |
|-------|-------------|
| AI Models | Mistral AI API, HuggingFace Transformers, Unsloth |
| Backend | FastAPI, Uvicorn, Python 3.12+ |
| Frontend | Vanilla JS, Three.js, Shadow DOM |
| Training | LoRA (TRL), Weights & Biases |
| Extension | Chrome Manifest V3, Service Worker |
| Deployment | HuggingFace Spaces, Docker |

---

## Languages Supported

English · Japanese · French

---

## License

MIT
