---
description: Generate a song with the local YuE2 Runner
---

Generate a song from the user's request: $ARGUMENTS

Use the `yue2-music` skill and `yue2-runner` MCP tools. Gather missing style or lyrics, then submit exactly one `yue2_task` generation with `userId: local`, a meaningful workflow id, and a stable idempotency key. Preserve the request id, seed, and idempotency key across retries. Poll the returned task id with `runner_result`; do not resubmit to check progress. When ready, retrieve artifact names from the result and return usable audio links with `runner_result_source` in link mode.
