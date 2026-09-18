---
description: Check a YuE2 Runner task and retrieve its completed artifacts
---

Inspect this YuE2 Runner task: $ARGUMENTS

Require a real task id, then call `runner_result` once. Do not submit or requeue any task. Report the current stage and outcome. If the task is complete and delivery is ready, list the returned artifacts and retrieve only those the user requests, using their exact `result_source_name` values.
