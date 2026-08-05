---
name: Build and run a retrieval-grounded SeekrFlow agent
description: >-
  Upload source documents, index them in a SeekrFlow vector database, attach a FileSearch tool to
  a new agent, promote the agent, and run it on a thread — then read the cited sources back out of
  the response.
api: openapi/seekr-llm-training-openapi.json
also:
  - openapi/seekr-agents-openapi.json
base_url: https://flow.seekr.com/v1
generated: '2026-08-05'
method: generated
source: >-
  Grounded in operationIds verified in openapi/seekr-llm-training-openapi.json and
  openapi/seekr-agents-openapi.json; conventions from conventions/seekr-conventions.yml.
operations:
  - file_upload_v1_flow_files_put
  - create_vector_database_route_v1_flow_vectordb_post
  - create_vector_database_ingestion_job_v1_flow_vectordb__database_id__ingestion_post
  - get_vector_database_ingestion_job_v1_flow_vectordb__database_id__ingestion__job_id__get
  - create_tool_v1_flow_tools_post
  - create_v1_flow_agents_create_post
  - promote_v1_flow_agents__agent_id__promote_put
  - create_thread_endpoint_v1_threads_post
  - run_agent_v1_threads__thread_id__runs_post
  - get_run_endpoint_v1_threads__thread_id__runs__run_id__get
  - list_messages_endpoint_v1_threads__thread_id__messages_get
---

# Build and run a retrieval-grounded SeekrFlow agent

## Before you start

- Base URL is `https://flow.seekr.com/v1`.
- Send `Authorization: <YOUR_API_KEY>` — **no `Bearer ` prefix**. This is the single most common
  auth mistake against this API.
- Every resource belongs to a **team**. Send `x-team-id: <TEAM_ID>` on every request, or the
  resource lands in your personal workspace and later calls scoped to the team will 403/404.
- `Content-Type: application/json` on POST/PUT bodies; file uploads are `multipart/form-data`.
- There is **no idempotency key**. A retried create makes a second object. Record the returned id
  before retrying anything.

## Steps

1. **Upload the source documents.** `file_upload_v1_flow_files_put`
   (`PUT /v1/flow/files`, multipart). Repeat per file, or use
   `bulk_file_upload_v1_flow_bulk_files_put` for a batch. Keep every returned file `id`.

2. **Create the vector database.** `create_vector_database_route_v1_flow_vectordb_post`
   (`POST /v1/flow/vectordb`). Keep the returned `database_id`.

3. **Ingest the files into it.**
   `create_vector_database_ingestion_job_v1_flow_vectordb__database_id__ingestion_post`
   (`POST /v1/flow/vectordb/{database_id}/ingestion`) with the `file_ids` from step 1.

4. **Poll until ingestion finishes.**
   `get_vector_database_ingestion_job_v1_flow_vectordb__database_id__ingestion__job_id__get`.
   This is a create-then-poll job — there is no webhook and no callback. Back off between polls;
   no rate limit is published, so be conservative.

5. **Create the FileSearch tool** bound to the vector database.
   `create_tool_v1_flow_tools_post` (`POST /v1/flow/tools`). Keep the returned tool `id`.

6. **Create the agent.** `create_v1_flow_agents_create_post` (`POST /v1/flow/agents/create`) with
   `name`, `instructions`, `model_id`, and `tool_ids: [<tool id from step 5>]`. Pick `model_id`
   from `ml_models_v1_flow_models_get`.

7. **Promote the agent** so it can be run.
   `promote_v1_flow_agents__agent_id__promote_put` (`PUT /v1/flow/agents/{agent_id}/promote`).
   An unpromoted agent is a draft.

8. **Create a thread.** `create_thread_endpoint_v1_threads_post` (`POST /v1/threads`).

9. **Run the agent.** `run_agent_v1_threads__thread_id__runs_post`
   (`POST /v1/threads/{thread_id}/runs`) with the `agent_id`. For token-by-token output use
   `run_agent_stream_v1_threads__thread_id__runs_stream_post` instead (SSE); to re-join an
   in-flight run use `attach_to_run_v1_threads__thread_id__runs__run_id__attach_get`.

10. **Poll the run** with `get_run_endpoint_v1_threads__thread_id__runs__run_id__get`, then read
    the answer with `list_messages_endpoint_v1_threads__thread_id__messages_get`.

11. **Show the receipts.** Message parts carry file references, and
    `get_context_attribution_from_run_v1_explainability_context_attributor_from_run_post`
    (explainability API) maps the answer back to the retrieved chunks. See
    `skills/seekr-explain-a-response.md`.

## Errors to expect

- **401** — missing/misformatted `Authorization`, or an inactive key. Check for a stray `Bearer `.
- **403** — valid key, wrong team. Set `x-team-id` to the team that owns the resource.
- **422** — validation failure. The body is `{"detail": [{"loc": [...], "msg": "...", "type": "..."}]}`;
  `loc` names the offending field. This is the most frequent failure on this API — it is declared
  on 172 of 212 operations.
- **500 / 503** — retry with backoff; check `https://status.seekr.com/`.

See `errors/seekr-problem-types.yml` for the full status table and both error envelopes.
