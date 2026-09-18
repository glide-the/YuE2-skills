---
description: Transcribe a local audio file to ABC notation with SheetSage2
---

Transcribe the user's audio file: $ARGUMENTS

Use the `yue2-runner` skill. Confirm the exact local path, call `runner_upload`, and then PUT the file bytes to its single-use URL. Do not treat URL issuance as a completed upload. Submit one `sheetsage2_task` using the returned asset id, `userId: local`, and a stable idempotency key. Poll only with `runner_result`. Retrieve the actual score artifact named in the completed result, preferring inline mode for a small ABC file.
