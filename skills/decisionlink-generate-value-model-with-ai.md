---
name: decisionlink-generate-value-model-with-ai
description: Use the Xfactor.io Value Chat service to generate a value proposition or value model with Growth AI — start the job, poll its status, and manage the chat session and prompts behind it.
api: Xfactor.io Value Chat API
base_url: https://api.xfactor.io
generated: '2026-08-13'
method: generated
source: openapi/_original/decisionlink-value-chat-openapi.json
operations:
  - get_chat_id_v1_value_chat_chat_id_post
  - get_chat_questions_v1_value_chat_chat_questions_get
  - upload_chat_file_v1_value_chat_chat__chat_id__upload_post
  - start_generate_vp_v1_value_chat_generate_vp_post
  - get_generate_vp_status_stream_generate_vp__job_id__status_get
  - start_generate_value_model_v1_value_chat_generate_value_model_post
  - get_generate_value_model_status_stream_generate_value_model__job_id__status_get
  - get_chat_history_v1_value_chat_chat__chat_id__history_get
  - chat_status_v1_value_chat_chat__chat_id__status_get
  - get_value_prompts_v1_value_chat_prompts_get
  - submit_chat_feedback_v1_value_chat_chat_feedback_post
---

# Generate a value model with Growth AI

The Value Chat service is Xfactor.io's conversational generation surface. Generation is **not**
synchronous: you start a job, then poll a `/stream/.../status` endpoint until it finishes.

> **Access.** `Authorization: Bearer <token>` on every call — an Auth0 JWT from
> `https://xf-prd.us.auth0.com/` or a customer-issued API key. See
> `authentication/decisionlink-authentication.yml`.

## Steps

1. **Open a chat session.**
   `POST /v1/value-chat/chat/id` (`get_chat_id_v1_value_chat_chat_id_post`) returns the `chat_id`
   every subsequent call in the session hangs off. Optionally read the starter question set with
   `GET /v1/value-chat/chat/questions` (`get_chat_questions_v1_value_chat_chat_questions_get`).

2. **Attach source material (optional).**
   `POST /v1/value-chat/chat/{chat_id}/upload`
   (`upload_chat_file_v1_value_chat_chat__chat_id__upload_post`) — the only `multipart/form-data`
   operation across all four Xfactor.io services. Everything else is `application/json`.

3. **Start the generation job.**
   - A value proposition: `POST /v1/value-chat/generate-vp`
     (`start_generate_vp_v1_value_chat_generate_vp_post`) with a `GenerateVPRequest`.
   - A value model: `POST /v1/value-chat/generate-value-model`
     (`start_generate_value_model_v1_value_chat_generate_value_model_post`) with a
     `GenerateValueModelRequest`.

   Both answer `202 Accepted` with a job identifier. Nothing is generated yet.

4. **Poll for completion.**
   `GET /stream/generate-vp/{job_id}/status`
   (`get_generate_vp_status_stream_generate_vp__job_id__status_get`) or
   `GET /stream/generate-value-model/{job_id}/status`
   (`get_generate_value_model_status_stream_generate_value_model__job_id__status_get`).
   Poll with backoff. **No polling interval, timeout, or terminal-state list is published** — treat
   any non-terminal response as "keep waiting" and impose your own ceiling.

5. **Read the conversation and close the loop.**
   `GET /v1/value-chat/chat/{chat_id}/history`
   (`get_chat_history_v1_value_chat_chat__chat_id__history_get`),
   `GET /v1/value-chat/chat/{chat_id}/status`
   (`chat_status_v1_value_chat_chat__chat_id__status_get`), and
   `POST /v1/value-chat/chat/feedback`
   (`submit_chat_feedback_v1_value_chat_chat_feedback_post`).

6. **Reuse prompts.** `GET /v1/value-chat/prompts`
   (`get_value_prompts_v1_value_chat_prompts_get`) lists the saved value prompts; create, update and
   delete are available on the same collection.

## Cautions

- **A realtime transport exists but is undocumented.** The service root answers Socket.IO/Engine.IO
  handshakes (`GET https://api.xfactor.io/value-chat/` → `400` naming the Engine.IO protocol), so
  there is very likely a push channel for generation progress. Xfactor.io publishes no AsyncAPI and
  no channel documentation, so polling the status endpoint is the only contract you can rely on.
- **Starting a job is not idempotent.** Two `POST /v1/value-chat/generate-vp` calls start two jobs
  and consume generation twice. There is no idempotency key.
- **Path prefix matters.** This service answers on `/api/v1/value-chat/...` for its OpenAPI document
  and Swagger UI, while the operations themselves are declared under `/v1/value-chat/...`. See
  `conventions/decisionlink-conventions.yml`.
