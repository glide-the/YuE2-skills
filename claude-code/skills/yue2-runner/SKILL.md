---
name: yue2-runner
description: Generate or plan songs with YuE2, transcribe audio to ABC notation with SheetSage2, inspect music task status, and retrieve Runner artifacts through the yue2-runner MCP server. Use when the user requests YuE music generation, cover preparation, audio transcription, score inspection, or results from an existing YuE task.
---

# YuE2 Runner

Use the four `yue2-runner` MCP tools as the only service interface:

- `runner_upload` signs a short-lived, single-use upload URL. It does not transfer the file.
- `runner_submit` submits one durable task.
- `runner_result` reads task state without requeueing it.
- `runner_result_source` retrieves a named artifact inline or as a link.

The service is local, tokenless, and single-tenant. Always use `local` for `payload.code_input.userId`. Create a meaningful `workflow_id` for the user's workflow.

## Safety and task identity

- Do not submit a GPU task merely to test connectivity. Listing tools is the safe connectivity check.
- Submit only when the user asked to generate, plan, decode, inspect, or transcribe.
- Generate one idempotency key before the first submission and reuse it for retries of the same intent. Never generate a new key merely because a request timed out.
- Poll an accepted task with `runner_result`; never resubmit it to check progress.
- Do not invent task IDs, asset IDs, artifact names, local paths, or completion state.
- Treat a successful `runner_upload` call as URL issuance only. Upload the bytes before submitting a task that references the returned asset.
- An upload URL is single-use. If upload fails or expires, request a new one.

## Generate a song

Collect the intended style and lyrics. Ask only for material information that is missing. Submit:

- `parameter.task_name`: `yue2_task`
- `parameter.reset`: `false`
- `parameter.user_multi_task`: `false`
- `payload.operation`: normally `generate`; use `plan` or `all_modes` only when the user requests that behavior
- `payload.request`: the song request

Use `cot: full` unless the user requests melody-only planning or explicitly disables planning. Choose a stable request `id` and an integer seed; preserve both when retrying.

## Transcribe audio

1. Confirm the local audio path and operating system.
2. Call `runner_upload` with that path and OS.
3. PUT the exact local file bytes to the returned `upload_url`, without an authorization header. Use a suitable audio content type when known.
4. Read the returned `asset_id` from the upload response.
5. Submit `sheetsage2_task` with `operation: transcribe`, the real `audio_asset_id`, and the requested transcription mode.

Default transcription mode is `full`. Use `melody-full` or `melody-vocal` only when that matches the user's request.

## Wait and retrieve results

Poll with a reasonable interval and give compact progress updates. Stop polling on completion, failure, cancellation, or when the user asks to stop.

When `delivery_status` is `ready`, read the actual `artifacts` list. Use each artifact's returned `result_source_name`; do not guess names. Prefer `mode: auto` for small text/ABC results and `mode: link` for audio or other large files. Report the task outcome and usable artifact locations.

For exact request shapes and operation constraints, read [references/runner-contract.md](references/runner-contract.md).
