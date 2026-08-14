# Week 4 AI Evals: Customer Support Agent Evaluation

This repository contains the Week 4 AI evaluation project for The Gen Academy. The project evaluates an e-commerce customer support routing agent that reads one customer ticket and predicts exactly one category:

- `order_status`
- `refund_request`
- `product_issue`
- `account_help`
- `other`

The goal is not just to report accuracy. The goal is to inspect failures, group them into useful failure categories, make one focused prompt improvement, rerun the evaluator, and measure the delta.

## Project Workflow

The project follows the Week 4 handout:

1. Inspect classifier failures.
   - Filter failed rows.
   - Add concrete annotations explaining why the classifier was wrong.

2. Create failure categories.
   - Group similar failures.
   - Name each failure pattern.
   - Label each failed row with one category.

3. Tweak the classifier prompt.
   - Pick one important failure category.
   - Make one focused prompt change.
   - Avoid rewriting the whole classifier prompt.

4. Rerun and compare.
   - Compare baseline vs improved predictions.
   - Track both wins and regressions.

5. Optional: run LLM-as-a-judge.
   - Choose one target failure category.
   - Use a binary judge to classify whether each failed row belongs to that target category.
   - Compare judge predictions against human labels.

## Repository Assets

| File | Purpose |
| --- | --- |
| `customer_support_evals.ipynb` | Main notebook. Builds the LangGraph classifier, enables LangSmith/OpenTelemetry tracing, generates predictions, evaluates metrics, and compares prompt versions. |
| `customer_support_evals_opentelemetry_executed.ipynb` | Executed notebook from the latest run. Includes generated outputs and metric summaries. |
| `results_v1.csv` | Baseline prompt prediction results. |
| `results_v2.csv` | Improved prompt prediction results. |
| `docs/opentelemetry_integration_one_pager.md` | Short explanation of where OpenTelemetry was added and how it supports the Week 4 eval workflow. |

## Latest Run Summary

The latest executed notebook produced:

| Run | Accuracy | Correct |
| --- | ---: | ---: |
| Baseline prompt | 92.00% | 92 / 100 |
| Improved prompt | 97.00% | 97 / 100 |
| Delta | +5.00% | +5 net correct |

Prediction changes between runs:

- 11 tickets changed prediction.
- 8 changed from wrong to right.
- 3 regressed from right to wrong.

The baseline prompt mainly over-routed some `order_status` and `product_issue` tickets into `refund_request` when the customer mentioned money back or refund language. The improved prompt added clearer disambiguation rules for damaged/wrong items, non-delivery, and refund-status requests.

## OpenTelemetry Integration

The notebook uses LangSmith as the AI evaluation and trace inspection UI. OpenTelemetry is added as the standard tracing layer.

In `customer_support_evals.ipynb`, OpenTelemetry was added in three places:

1. Dependencies:
   - `langsmith[otel]`
   - `opentelemetry-sdk`
   - `opentelemetry-exporter-otlp`

2. Environment setup:
   - `LANGSMITH_OTEL_ENABLED=true`
   - `OTEL_SERVICE_NAME=customer-support-evals-notebook`

3. Prediction loop:
   - Each ticket classification is wrapped in a span named `customer_support.classify_ticket`.
   - Each span records ticket-level evaluation metadata:
     - `eval.example_id`
     - `eval.run_name`
     - `eval.prompt_version`
     - `eval.true_category`
     - `eval.predicted_category`
     - `eval.correct`
     - `eval.reasoning`

This makes every CSV row traceable back to a LangSmith/OpenTelemetry run. When a row fails, you can inspect the exact trace, model reasoning, prompt version, and expected vs predicted label.

## Running The Notebook

Create a virtual environment and install dependencies:

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install langgraph "langsmith[otel]>=0.4.25" langchain-openai langchain-core pandas scikit-learn pydantic tqdm opentelemetry-sdk opentelemetry-exporter-otlp
```

Set API keys locally:

```bash
export OPENAI_API_KEY="..."
export LANGSMITH_API_KEY="..."
export LANGSMITH_TRACING=true
export LANGSMITH_PROJECT=customer-support-evals
export LANGSMITH_ENDPOINT=https://api.smith.langchain.com
export LANGSMITH_OTEL_ENABLED=true
export OTEL_SERVICE_NAME=customer-support-evals-notebook
```

Then open and run:

```bash
jupyter notebook customer_support_evals.ipynb
```

Do not commit API keys, `.env` files, or local notebook secrets.

## Source Materials

- [Customer Support Agent Evaluation Project Handout](https://docs.google.com/document/d/1zsxSbgZxsLVNVeOx3n95w555xAykzqZM9ewc9a4rTtk/edit?tab=t.0)
- [Week 4 AI Evals spreadsheet](https://docs.google.com/spreadsheets/d/1DQXHydc-zx2fPh9flEwUKXyIMERs153CBvbLzUqqZWw/edit)
- [Week 4 submission form](https://forms.gle/ziAMAGXSvzyWzqAU7)
