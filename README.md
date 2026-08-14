# Week 4 Project: Customer Support AI Evals

This repo contains the completed Week 4 AI Evals project for The Gen Academy. The project evaluates a customer support routing assistant that reads a customer ticket and predicts one support category:

- `order_status`
- `refund_request`
- `product_issue`
- `account_help`
- `other`

The notebook is already executed, so you can open it and review the full workflow, outputs, metrics, and trace instrumentation without rerunning anything first.

## Start Here

Open:

```text
week4_customer_support_evals.ipynb
```

That notebook walks through the full Week 4 workflow:

1. Generate a labeled support-ticket dataset.
2. Run a baseline classifier prompt.
3. Evaluate predictions against the known labels.
4. Inspect mistakes and group failure patterns.
5. Improve the prompt based on one failure pattern.
6. Rerun the eval and compare the results.
7. Add OpenTelemetry tracing so individual ticket runs can be inspected in LangSmith.

## Results At A Glance

| Run | Accuracy | Correct |
| --- | ---: | ---: |
| Baseline prompt | 92.00% | 92 / 100 |
| Improved prompt | 97.00% | 97 / 100 |
| Change | +5.00% | +5 net correct |

The improved prompt reduced confusion between refund-related language, order-status issues, and product problems. It also exposed a few regressions, which is useful for Week 4 because the assignment is about understanding both improvements and tradeoffs.

## Files In This Repo

| File | What It Is |
| --- | --- |
| `week4_customer_support_evals.ipynb` | Final executed notebook with the complete project workflow and outputs. |
| `results_v1.csv` | Baseline prompt predictions and evaluation results. |
| `results_v2.csv` | Improved prompt predictions and evaluation results. |
| `docs/opentelemetry_integration_one_pager.md` | Short explanation of how OpenTelemetry was added to the project. |

## How OpenTelemetry Fits

LangSmith is used as the AI evaluation and trace-inspection workspace. OpenTelemetry is used as the standard tracing layer that sends structured spans into LangSmith.

In this project, each ticket classification is wrapped in an OpenTelemetry span. The span records useful evaluation context, including:

- ticket id
- prompt version
- expected category
- predicted category
- whether the model was correct
- model reasoning

This makes the eval easier to explain. Instead of only saying "accuracy improved," you can point to the exact ticket traces that show where the baseline failed, what the model reasoned, and how the improved prompt changed the outcome.

If you see this warning while rerunning the notebook:

```text
Failed to export span batch code: 404, reason: Not Found
```

the classifier is still running, but OpenTelemetry is trying to send traces to the wrong endpoint. The notebook now sets the LangSmith OTLP endpoint explicitly to avoid that issue.

## Rerunning The Notebook

You only need to rerun the notebook if you want to regenerate results or create fresh traces in your own LangSmith project.

Install the dependencies:

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install langgraph "langsmith[otel]>=0.4.25" langchain-openai langchain-core pandas scikit-learn pydantic tqdm opentelemetry-sdk opentelemetry-exporter-otlp
```

Set your own API keys locally:

```bash
export OPENAI_API_KEY="your-openai-key"
export LANGSMITH_API_KEY="your-langsmith-key"
export LANGSMITH_TRACING=true
export LANGSMITH_PROJECT=customer-support-evals
export LANGSMITH_ENDPOINT=https://api.smith.langchain.com
export LANGSMITH_OTEL_ENABLED=true
```

Then run:

```bash
jupyter notebook week4_customer_support_evals.ipynb
```

Do not commit API keys, `.env` files, or local notebook secrets.

## Source Materials

- [Customer Support Agent Evaluation Project Handout](https://docs.google.com/document/d/1zsxSbgZxsLVNVeOx3n95w555xAykzqZM9ewc9a4rTtk/edit?tab=t.0)
- [Week 4 AI Evals spreadsheet](https://docs.google.com/spreadsheets/d/1DQXHydc-zx2fPh9flEwUKXyIMERs153CBvbLzUqqZWw/edit)
- [Week 4 submission form](https://forms.gle/ziAMAGXSvzyWzqAU7)
