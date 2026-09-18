---
name: yue2-music
description: Generate, cover, transcribe, and edit songs with YuE2 and SheetSage2/MERT2 through the yue2-runner MCP service. Use for YuE2 full/melody/off generation, audio-to-ABC covers, style or lyric changes, score editing, agentic reharmonization, melody preservation, singable lyric adaptation, reproducible listening comparisons, task inspection, and result retrieval; also for YuE2 生成、翻唱、改编、改谱、换词和智能体编辑.
---

# YuE2 Music through Runner

Turn a musical request into a reproducible song and an audible comparison. Use released model interfaces. Retain an original song and its plan before making changes.

All model work goes through the `yue2-runner` MCP server. Do not load YuE2 or SheetSage2 locally merely because the upstream helper scripts are bundled with this skill.

## Choose the workflow

| Request | Workflow |
| --- | --- |
| Generate with editable melody and harmony | YuE2 `cot="full"` → ABC → song |
| Generate with a melody plan and free accompaniment | YuE2 `cot="melody"` → chord-free ABC → song |
| Generate without symbolic planning | YuE2 `cot="off"` → song; no editable ABC |
| Cover a recording | SheetSage2 → inspect/correct ABC → strip chords → YuE2 `melody` |
| Cover an ABC melody | Inspect uploaded ABC → strip chords → YuE2 `melody` |
| Change harmony, instruments, tempo, structure, or lyrics | Copy full plan → edit ABC/text → regenerate |
| Agentic editing | Export plan/baseline → bounded editing agent → check invariants → render → compare |
| Analyze musical features | Use MERT2 only when continuous features are needed |

```text
audio → SheetSage2 [loads MERT-v2-FullSong itself] → ABC
style + lyrics → YuE2 full/melody planning        → ABC
                                                  edit/validate
style + lyrics + ABC → YuE2 semantic generation → synthesis → latents → VAE → song
style + lyrics      → YuE2 off generation      → synthesis → latents → VAE → song
```

Do not feed public MERT feature tensors to YuE2 as codec tokens. YuE2 exposes no audio-reference, phoneme-alignment, or local-inpainting argument.

## Use the Runner contract

The server exposes exactly four tools:

- `runner_upload` signs a short-lived, single-use upload URL; it does not transfer bytes.
- `runner_submit` submits one durable task.
- `runner_result` reads task state without requeueing it.
- `runner_result_source` retrieves a named artifact inline or as a link.

The deployment is local, tokenless, and single-tenant. Always use `local` for `payload.code_input.userId` and create a meaningful `workflow_id`.

- Listing tools is the safe connectivity check. Never submit a GPU task merely to test connectivity.
- Submit only when the user asked for model or score work.
- Create one idempotency key before the first submission and reuse it for retries of the same intent. A timeout is not permission to create a new key.
- Poll an accepted task with `runner_result`; never resubmit to check progress.
- Do not invent task IDs, asset IDs, artifact names, paths, or completion state.
- After `runner_upload`, PUT the exact local file bytes to the returned URL before referencing the returned asset. Do not add an authorization header. If the single-use URL fails or expires, request a new one.

Read [runner-contract.md](references/runner-contract.md) for exact request shapes and operation constraints.

## Understand the model setup

Read [models-and-setup.md](references/models-and-setup.md). The Runner host, not the client, owns the YuE2 and SheetSage2 environments and public model snapshots. YuE2 and SheetSage2 use separate environments because their dependency pins differ. Model weights and third-party dependencies retain their applicable licenses.

Use the supported baseline: one request at a time, BF16-capable NVIDIA GPU with 24 GB VRAM, and default YuE2 settings. Do not silently shorten a requested song or lower inference settings to hide an OOM. Report resource-driven changes instead.

Use `YuE2-Vae` for listening and `YuE2-Vae-legacy` only when reproducing the supplied benchmark protocol. Keep decoded files separate. Do not infer their roles from the word “legacy.”

The bundled `scripts/` are the official direct-runtime helpers retained for reproducibility. Do not invoke them in the normal MCP workflow. Use them only when the user explicitly selects direct execution in an environment where the official models and dependencies are installed.

## Generate and retain the plan

Start with [assets/prompt.json](assets/prompt.json) as an original shape example. Replace its content with the user's request. Put genre, instruments, vocal character, language, and intended tempo in `style`; put section tags and actual words in `lyrics`. Keep implementation notes out of lyrics.

Submit `yue2_task` through `runner_submit`:

- use `operation: generate` for one requested mode;
- use `operation: plan` when the user wants the symbolic plan without rendering a fresh song;
- use `operation: all_modes` only when the user explicitly wants all supported modes;
- set `parameter.reset` and `parameter.user_multi_task` to `false`.

Use `cot: full` unless the user requests melody-only planning or explicitly disables symbolic planning. Choose a stable request `id` and integer seed, and preserve the request, seed, and idempotency key across retries.

Poll the returned task ID. Inspect the completed result, truncation/failure state, and exact artifact list. Retrieve `score.abc`, request metadata, tokens/latents, and audio only when those artifacts are actually listed. Preserve all requested modes and failures. A successful process or playable file does not establish musical quality.

Read [generation-and-covers.md](references/generation-and-covers.md) for Python semantics, exact plan continuation, CFG, sampling, and cached decoding. With Runner, continue or modify a plan by referencing a returned task artifact or uploading a new ABC asset; do not pass client filesystem paths to the server. An idempotent retry verifies the same intent—it does not continue an interrupted generation under a new identity.

## Cover a recording

1. Call `runner_upload` for the source recording and PUT its bytes to the single-use URL. Preserve the original recording.
2. Submit `sheetsage2_task` with `operation: transcribe`, the returned `audio_asset_id`, and the requested mode. Select `melody-vocal` for vocal melody only, `melody-full` for the full lead including instrumental passages, or `full` when harmony must be retained.
3. Poll with `runner_result`, retrieve the raw transcription artifact, and inspect warnings. Correct missed notes, meter, or key before attributing errors to YuE2.
4. For a melody-conditioned cover, submit a `yue2_task` `score_check` with `action: strip_chords`, referencing the returned score artifact. Select a retained voice explicitly when dropping a part; removing chords alone should preserve melodic voices and rests.
5. Submit a new `yue2_task` generation with `cot: melody`, target style and suitable lyrics, and `abc_source` referencing the checked chord-free score artifact.

This supplies a symbolic melody condition; it does not preserve the source singer's identity or waveform. `cot: melody` does not remove chord symbols automatically. To retain original harmony as well, use a full transcription and `cot: full`; call this score-conditioned regeneration with melody and harmony.

## Edit or delegate an edit

Read [editing-workflows.md](references/editing-workflows.md) and [abc-editing.md](references/abc-editing.md) before changing a score.

1. Render and retain a baseline from the full plan. Never overwrite its task artifacts.
2. Define invariants: exact pitches; pitch plus rhythm; contour only; or bounded melodic adaptation. Specify voices/passages, lyrics, instruments, tempo, meter, and structure.
3. Retrieve the source ABC from its real artifact name. If delegation is available, give a score-editing agent the raw ABC, prompt, lyrics, requested change, and [edit brief](assets/edit-brief.md). Request a new ABC, revised style/lyrics as needed, and an edit manifest. Give a separate reviewer the before/after artifacts and constraints. Without delegation, perform these stages yourself. Keep model generation sequential per GPU.
4. Check musical events, not character strings: ties, accidentals, and compressed rests matter. Upload the edited ABC as a new immutable asset. Submit `score_check` with `inspect` and, when comparing, `compare` with both source and edited references. Set `allow_tempo_change` only for an intentional tempo change. Exact comparison should fail for intentional rhythm changes; audit permitted differences instead of relabeling the result “melody preserved.”
5. Regenerate in a new task with the edited ABC asset in `abc_source`. Keep the edited score explicitly connected. Omitting both inline ABC and `abc_source` generates a fresh plan and discards the edit.
6. Changing style, lyrics, or ABC requires a new render. Old acoustic latents can be decoded again but cannot implement a musical or lyric edit.
7. Compare full songs and short passages around the edit. Revise when the requested effect fails; retain each attempt and its actual prompt.

For lyric translation, adapt syllables, stress, vowels, and breath points. Keep a syllable/phoneme-to-note sidecar. Do not invent a `phonemes` field or mistake the sidecar for hard acoustic alignment. Use ASR/PER and listening as separate evidence.

## Deliver an audible result

Read [listening-and-evaluation.md](references/listening-and-evaluation.md). Return playable audio, full prompt/lyrics, before/after ABC, invariant checks, and requested evaluations. Keep model/decoder identity and failures visible.

Use `runner_result_source` with each exact returned `result_source_name`. Prefer `mode: auto` for small text or ABC artifacts and `mode: link` for audio or other large files. If the user requests a local comparison page after downloading artifacts, the bundled official `scripts/listen.py` can build it without publishing or uploading anything.

Distinguish symbolic checks, ASR, listening, and quality scores. Deliver custom edit manifests and before/after comparison reports alongside audio. Do not claim exact note realization, instrument removal, singer identity preservation, or sample-accurate preservation from an ABC check or SongBench score alone.

The upstream helpers, assets, and domain references retain their Apache-2.0 license in [LICENSE](LICENSE). Provenance and the pinned revision are recorded in [UPSTREAM.md](UPSTREAM.md).
