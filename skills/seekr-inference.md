---
name: Run inference on SeekrFlow (chat, embeddings, rerank, transcription)
description: >-
  Call SeekrFlow's serving surface directly — OpenAI-compatible chat completions and embeddings,
  plus rerank, score, batch and audio transcription — including from an unmodified OpenAI SDK.
api: openapi/seekr-serving-openapi.json
base_url: https://flow.seekr.com
generated: '2026-08-05'
method: generated
source: >-
  Grounded in operationIds verified in openapi/seekr-serving-openapi.json; compatibility matrix
  from https://docs.seekr.com/flow/sdk/getting-started.
operations:
  - show_models_v1_models_get
  - route_chat_completion_v1_inference_chat_completions_post
  - route_completion_v1_inference_completions_post
  - route_embeddings_v1_inference_embeddings_post
  - route_v1_rerank_v1_inference_rerank_post
  - route_v1_score_v1_inference_score_post
  - route_v1_audio_transcriptions_v1_inference_audio_transcriptions_post
  - route_batches_v1_batches_post
  - route_list_batches_v1_batches_get
  - route_get_batch_v1_batches__batch_id__get
  - route_cancel_batch_v1_batches__batch_id__delete
  - health_v1_inference_health_get
---

# Run inference on SeekrFlow

## Before you start

- Serving lives under `https://flow.seekr.com/v1/inference` (some operational routes —
  `/health`, `/version`, `/metrics`, `/engines`, `/rerank`, `/score` — are unversioned at the
  host root).
- `Authorization: <YOUR_API_KEY>`, **no `Bearer ` prefix**. `x-team-id` scopes to a team.
- This is the one surface with an **OpenAI-compatible** shape.

## Steps

1. **List what you can call.** `show_models_v1_models_get` (`GET /v1/models`) returns the models
   available to your team, including any fine-tuned model you promoted.

2. **Chat.** `route_chat_completion_v1_inference_chat_completions_post`
   (`POST /v1/inference/chat/completions`). `route_completion_v1_inference_completions_post` is
   the legacy text-completion form.

3. **Embed.** `route_embeddings_v1_inference_embeddings_post`
   (`POST /v1/inference/embeddings`).

4. **Rerank / score.** `route_v1_rerank_v1_inference_rerank_post` and
   `route_v1_score_v1_inference_score_post` — use these to order retrieval candidates before
   feeding them to a model.

5. **Transcribe audio.** `route_v1_audio_transcriptions_v1_inference_audio_transcriptions_post`.

6. **Batch large jobs.** `route_batches_v1_batches_post` to submit,
   `route_list_batches_v1_batches_get` / `route_get_batch_v1_batches__batch_id__get` to track,
   `route_cancel_batch_v1_batches__batch_id__delete` to stop. Batch inputs are uploaded with
   `route_files_v1_files_post` and read back with
   `route_get_file_content_v1_files__file_id__content_get`.

7. **Check health** with `health_v1_inference_health_get` before blaming your payload.

## Using the OpenAI SDK instead

Point an unmodified OpenAI client at `base_url="https://flow.seekr.com/v1/inference"` with your
Seekr key.

- **Supported:** `model`, `messages`, `stream`, `temperature`, `logprobs`, `top_logprobs`,
  `max_tokens`, `stop`, `top_p`, `frequency_penalty`, `presence_penalty`, `tools`.
- **Not supported:** `tool_choice`, `parallel_tool_calls`, `n`, `logit_bias`,
  `max_completion_tokens`. Sending them is a compatibility trap — do not assume parity.

LangChain users can use `ChatSeekrFlow` from `langchain-seekrflow` (tool calling, structured
output, JSON mode, streaming, token usage; no async, no image/audio/video input).

## Notes

- Dedicated serving instances can be parked and woken: `route_sleep_sleep_post`,
  `route_wake_up_wake_up_post`, `route_is_sleeping_is_sleeping_get`. Budget for a cold start after
  a sleep.
- Estimate spend first with `generate_price_estimate_for_serverless_inference_...` or
  `generate_price_estimate_for_dedicated_inference_...` on the platform API.
- **No published rate limits and no `X-RateLimit-*` headers** — implement your own client-side
  concurrency ceiling and exponential backoff on 429/503.
- The serving spec is the only one of the four with **no operation tags at all**; see
  `overlays/seekr-serving-overlay.yaml` for the capability tags API Evangelist added.
