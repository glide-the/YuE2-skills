---
description: Show the YuE2 Runner capabilities and connection requirements
---

Explain concisely that the YuE2 plugin can generate and plan songs, transcribe audio into ABC notation with SheetSage2, inspect/strip/compare scores, build listening comparison bundles, monitor durable tasks, and retrieve output artifacts. These operations run as server-side Runner tasks; the plugin never executes the vendored scripts or loads models on the client.

The bundled MCP endpoint is `http://127.0.0.1:11000/mcp`. It requires the AutoDL Runner to be running and an SSH tunnel from local port `11000` to the Runner's `127.0.0.1:10000`. It does not use an application token. Listing MCP tools is a safe connection test and must not start a model.

Mention the commands `/yue2:generate-song`, `/yue2:transcribe-audio`, and `/yue2:task-status`. Do not submit a task while showing help.
