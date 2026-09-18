# Runner contract

## Shared submission envelope

```json
{
  "parameter": {
    "task_name": "yue2_task",
    "reset": false,
    "user_multi_task": false
  },
  "payload": {
    "code_input": {
      "userId": "local",
      "workflow_id": "workflow-name"
    },
    "operation": "generate"
  },
  "idempotency_key": "one-stable-key-per-intent"
}
```

Only `yue2_task` and `sheetsage2_task` are available in this deployment. Do not add `_service`, interpreter paths, model paths, output paths, shell commands, or unrelated fields.

## YuE2 operations

### Generate or plan

`generate`, `plan`, and `all_modes` require `payload.request`:

```json
{
  "id": "song-id",
  "style": "music style, voice, instrumentation, tempo",
  "lyrics": "[Verse]\n...\n[Chorus]\n...",
  "cot": "full",
  "seed": 831001
}
```

- `id` must start with an alphanumeric character and then contain only alphanumerics, `_`, `.`, or `-`.
- `cot` is `full`, `melody`, or `off`.
- `seed` is an integer from `0` through `2^63 - 1`.
- Optional `cfg_scale` must be finite and between `0` and `20`.
- Optional inline `abc` and `abc_source` are mutually exclusive.
- `all_modes` accepts text request input only.

### Decode

`decode` requires `source_task_id` and does not accept `request`, `abc_source`, or score-check fields.

### Score check

`score_check` requires:

```json
{
  "operation": "score_check",
  "check": {
    "action": "inspect",
    "source": {
      "task_id": "existing-task-id",
      "result_source_name": "name-from-artifacts"
    },
    "voices": "both",
    "allow_tempo_change": false
  }
}
```

Actions are `inspect`, `strip_chords`, or `compare`. `compare` also requires `after`. A resource reference contains either `asset_id`, or both `task_id` and `result_source_name`.

## SheetSage2 transcription

```json
{
  "parameter": {
    "task_name": "sheetsage2_task",
    "reset": false,
    "user_multi_task": false
  },
  "payload": {
    "code_input": {
      "userId": "local",
      "workflow_id": "transcription-name"
    },
    "operation": "transcribe",
    "audio_asset_id": "asset-id-returned-by-upload",
    "transcription": {
      "task": "full"
    }
  }
}
```

Transcription modes are `full`, `melody-full`, and `melody-vocal`. `max_seconds` is optional and must be positive.

## Task results

An accepted submission returns a real `task_id`. Query it with `runner_result`. A completed task contains a result with `outcome`, `delivery_status`, and `artifacts`. Only retrieve an artifact after it appears in this returned list.
