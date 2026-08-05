---
name: Explain a SeekrFlow response — trace it to sources, spans and training data
description: >-
  Take an agent run and produce the audit trail behind it: which retrieved chunks influenced the
  answer, which spans the run executed, and which fine-tuning examples shaped the model.
api: openapi/seekr-explainability-openapi.json
also:
  - openapi/seekr-agents-openapi.json
  - openapi/seekr-llm-training-openapi.json
base_url: https://flow.seekr.com/v1
generated: '2026-08-05'
method: generated
source: >-
  Grounded in operationIds verified in openapi/seekr-explainability-openapi.json,
  openapi/seekr-agents-openapi.json and openapi/seekr-llm-training-openapi.json.
operations:
  - get_context_attribution_from_run_v1_explainability_context_attributor_from_run_post
  - get_context_attribution_v1_explainability_context_attributor_post
  - query_spans_v1_observability_spans_post
  - retrieve_span_v1_observability_spans__span_id__get
  - create_index_v1_explainability_create_index_post
  - populate_index_v1_explainability_populate_index_post
  - populate_index_job_status_v1_explainability_populate_index_job_status_get
  - get_influential_finetuning_data_v1_explainability_influential_finetuning_data_get
  - get_influential_training_data_route_v1_flow_explain_models__model_id__influential_finetuning_data_get
  - get_vector_database_chunk_v1_flow_vectordb__database_id__chunk__chunk_id__get
---

# Explain a SeekrFlow response

Explainability is the product here, not a debugging afterthought: Seekr's pitch is receipts for
*why* a model answered, not just logs of *what* it did. There are three independent traces, and
they answer different questions.

## Before you start

- `https://flow.seekr.com/v1`, `Authorization: <YOUR_API_KEY>` with **no `Bearer ` prefix**,
  `x-team-id` on every call.
- Check the explainability service is up first: `check_status_v1_explainability_status_get`.

## A. Which retrieved sources influenced this answer?

1. Run the agent and keep the `thread_id` and `run_id`
   (see `skills/seekr-rag-agent.md`).
2. `get_context_attribution_from_run_v1_explainability_context_attributor_from_run_post`
   (`POST /v1/explainability/context-attributor-from-run`) with `thread_id` + `run_id`.
   Use `get_context_attribution_v1_explainability_context_attributor_post` instead when you want
   to attribute an arbitrary response/context pair rather than a stored run.
3. Resolve each attributed chunk back to its exact position in the source document with
   `get_vector_database_chunk_v1_flow_vectordb__database_id__chunk__chunk_id__get`.

## B. What did the run actually do?

1. `query_spans_v1_observability_spans_post` (`POST /v1/observability/spans`) filtered by
   `agent_id`, `thread_id`, `run_id` or `trace_id`.
2. `retrieve_span_v1_observability_spans__span_id__get` for one span. Spans are a tree —
   `parent_span_id` chains them under a single `trace_id`, giving every model call and tool
   invocation in order.

## C. Which training examples shaped this model?

1. **Build the index once per model.** `create_index_v1_explainability_create_index_post`, then
   `populate_index_v1_explainability_populate_index_post` with the training `file_ids`.
2. **Poll** `populate_index_job_status_v1_explainability_populate_index_job_status_get`.
3. **Query influence.** `get_influential_finetuning_data_v1_explainability_influential_finetuning_data_get`
   on the explainability API, or
   `get_influential_training_data_route_v1_flow_explain_models__model_id__influential_finetuning_data_get`
   on the platform API. Both return the training examples ranked by influence on the response.
4. `delete_index_v1_explainability_delete_index_post` tears the index down.

## Notes

- The explainability API declares an `HTTPBearer` scheme in addition to the `APIKeyHeader` used
  everywhere else — it is the only one of the four SeekrFlow specs that does. Send the API key in
  `Authorization` as normal unless a 401 says otherwise.
- Most explainability operations are tagged `Internal` by Seekr. Treat them as a surface that can
  change without a version bump, and prefer the SDK's explainability helpers where you can.
