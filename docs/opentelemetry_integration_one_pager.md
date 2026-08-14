# OpenTelemetry Integration One-Pager

## Purpose

The Week 4 customer support notebook already builds a LangGraph classifier, traces runs in LangSmith, computes accuracy, exports CSVs, and compares a baseline prompt against an improved prompt. OpenTelemetry is added as the standard tracing layer underneath that workflow. It does not replace LangSmith. It makes each evaluation run structured, portable, and easier to explain.

## Plain-English Explanation

LangSmith is the AI evaluation workspace: it stores traces, shows LLM calls, helps inspect failures, and supports baseline-vs-improved comparison.

OpenTelemetry is the instrumentation standard: it defines how code emits traces and metadata.

In this project, OpenTelemetry gives every ticket classification a trace with useful evaluation context. LangSmith receives those traces so we can debug and report them.

## Where It Fits In The Notebook

The best integration point is the `run_predictions(agent, tickets)` loop. That function calls `agent.invoke(...)` once per support ticket and records:

- `true_category`
- `predicted_category`
- `reasoning`
- `correct`

The OpenTelemetry version wraps each ticket prediction in a span named `customer_support.classify_ticket`, then attaches evaluation metadata to that span.

## Metadata Captured On Each Span

- `eval.example_id`: ticket id
- `eval.run_name`: baseline or improved
- `eval.prompt_version`: v1 or v2
- `eval.true_category`: ground-truth label
- `eval.predicted_category`: model output
- `eval.correct`: whether prediction matched the label
- `eval.reasoning`: model explanation
- `langsmith.span.kind`: chain

## What This Enables

When a ticket is misclassified, the CSV row can be connected to an exact trace. The trace shows the ticket, expected label, predicted label, prompt version, model reasoning, and child LLM call. This makes failure analysis more concrete than accuracy alone.

For the Week 4 submission, this supports:

- baseline metrics
- improved metrics
- measured delta
- trace-backed examples for failure clusters
- evidence that a prompt change helped or caused regressions

## Loom Talk Track

"I am using OpenTelemetry as the tracing standard and LangSmith as the evaluation UI. The dataset gives us known correct labels. For each ticket, the classifier run becomes one OpenTelemetry span with the ticket id, expected label, predicted label, prompt version, and correctness. LangSmith receives those spans, so when accuracy or recall drops, I can click into the exact failed trace, inspect the model reasoning, cluster the failure, update the prompt, and rerun the same dataset. The final result is not just a better score; it is trace-backed evidence of what improved and what still fails."
