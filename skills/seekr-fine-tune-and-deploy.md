---
name: Turn documents into a fine-tuned, deployed SeekrFlow model
description: >-
  Run the full AI-ready data pipeline — upload and ingest source files, submit a data job,
  generate instruction-tuning pairs, launch a fine-tune, promote the resulting model, and deploy
  it for inference.
api: openapi/seekr-llm-training-openapi.json
base_url: https://flow.seekr.com/v1
generated: '2026-08-05'
method: generated
source: >-
  Grounded in operationIds verified in openapi/seekr-llm-training-openapi.json; conventions from
  conventions/seekr-conventions.yml; deprecation from lifecycle/seekr-lifecycle.yml.
operations:
  - file_upload_v1_flow_files_put
  - ingest_files_v1_flow_ingestion_post
  - submit_data_job_v1_flow_data_jobs_post
  - add_files_to_data_job_v1_flow_data_jobs__data_job_id__add_files_post
  - start_data_job_alignment_v1_flow_data_jobs__data_job_id__start_post
  - get_data_job_v1_flow_data_jobs__data_job_id__get
  - alignment_outputs_v1_flow_alignment__alignment_job_id__outputs_get
  - generate_price_estimate_for_finetuning_v1_flow_pricing_fine_tuning_post
  - fine_tune_v1_flow_fine_tune_post
  - get_fine_tune_v1_flow_fine_tunes__fine_tune_id__get
  - fine_tune_workflow_phase_v1_flow_fine_tunes__fine_tune_id__workflow_phase_get
  - promote_model_v1_flow_fine_tunes__fine_tune_id__promote_model_get
  - deploy_v1_flow_deployments_post
  - promote_v1_flow_deployments__deployment_id__promote_put
---

# Turn documents into a fine-tuned, deployed SeekrFlow model

## Before you start

- `https://flow.seekr.com/v1`, `Authorization: <YOUR_API_KEY>` with **no `Bearer ` prefix**,
  `x-team-id: <TEAM_ID>` on every call.
- Every stage is a **job**: create it, then poll it. Nothing calls you back.
- **Context-grounded fine-tuning was deprecated on 2026-06-23 and removed on 2026-07-17.** Choose
  another method (instruction fine-tuning, LoRA, DPO preference tuning, GRPO reinforcement tuning,
  or vision-language tuning) for new work.
- No idempotency key exists — capture every returned job id before you retry.

## Steps

1. **Upload the source documents.** `file_upload_v1_flow_files_put` (PDF, DOCX, Markdown, PPT).
   Keep the file ids.

2. **Ingest them.** `ingest_files_v1_flow_ingestion_post` (`POST /v1/flow/ingestion`) converts raw
   documents into reviewable Markdown. Poll `ingestion_get_v1_flow_ingestion__ingestion_job_id__get`.

3. **Submit a data job.** `submit_data_job_v1_flow_data_jobs_post` (`POST /v1/flow/data-jobs`).
   A data job is the single tracked id that bundles ingestion, Markdown review, system-prompt
   generation and alignment configuration.

4. **Attach files to the job** if you did not pass them at creation:
   `add_files_to_data_job_v1_flow_data_jobs__data_job_id__add_files_post`.
   (`remove_files_from_data_job_...` is the inverse.)

5. **Start alignment** — generate the instruction/QA pairs:
   `start_data_job_alignment_v1_flow_data_jobs__data_job_id__start_post`.

6. **Poll progress.** `get_data_job_v1_flow_data_jobs__data_job_id__get`, or the alignment job's
   own `alignment_workflow_phase_v1_flow_alignment__alignment_job_id__workflow_phase_get`.
   Retrieve the generated dataset metadata with
   `alignment_outputs_v1_flow_alignment__alignment_job_id__outputs_get`.

7. **Price it before you spend.**
   `generate_price_estimate_for_finetuning_v1_flow_pricing_fine_tuning_post`
   (`POST /v1/flow/pricing/fine-tuning`) returns a cost estimate for the job you are about to run.

8. **Launch the fine-tune.** `fine_tune_v1_flow_fine_tune_post` (`POST /v1/flow/fine-tune`).

9. **Track it.** `get_fine_tune_v1_flow_fine_tunes__fine_tune_id__get` and
   `fine_tune_workflow_phase_v1_flow_fine_tunes__fine_tune_id__workflow_phase_get`.
   Abort with `cancel_fine_tune_v1_flow_fine_tunes__fine_tune_id__cancel_put`;
   pull weights with `download_fine_tune_v1_flow_fine_tunes__fine_tune_id__download_get`.

10. **Promote the model.** `promote_model_v1_flow_fine_tunes__fine_tune_id__promote_model_get`
    makes the fine-tuned model selectable. (`demote_model_...` reverses it.)

11. **Deploy it.** `deploy_v1_flow_deployments_post` (`POST /v1/flow/deployments`) with the
    `model_id`, then `promote_v1_flow_deployments__deployment_id__promote_put` to make the
    endpoint live. Watch `get_deployment_metrics_v1_flow_metrics_deployments_get` and
    `get_deployment_events_by_id_v1_flow_events_deployments__deployment_id__get`.

12. **Call it.** The deployed model is addressable from
    `route_chat_completion_v1_inference_chat_completions_post` on the serving API — or from any
    OpenAI SDK pointed at `https://flow.seekr.com/v1/inference`.

## Errors to expect

- **422** with a `detail[]` array is the dominant failure — read `detail[].loc` for the field.
- **409** appears on a small number of operations; treat it as a state conflict and re-read the
  job before retrying.
- **403** almost always means the wrong `x-team-id`.

See `errors/seekr-problem-types.yml` and `conventions/seekr-conventions.yml`.
