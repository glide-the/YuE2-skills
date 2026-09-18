# YuE2 Skills

Claude Code plugin for the local YuE2 Runner MCP service. It bundles the MCP connection, a reusable YuE2/SheetSage2 skill, official YuE2 music assets and adapted domain references, and commands for generation, transcription, task inspection, and result retrieval. The plugin is a pure client: every operation runs as a Runner Task/Processor on the server.

## Install

In Claude Code:

```text
/plugin marketplace add glide-the/YuE2-skills
/plugin install yue2@yue2-skills
```

Restart Claude Code if requested, then run `/mcp` and confirm that `yue2-runner` is connected.

## Prerequisites

Start the Runner on AutoDL, then keep an SSH tunnel open from the client machine:

```bash
ssh -N -o ExitOnForwardFailure=yes \
  -L 11000:127.0.0.1:10000 \
  -p <AutoDL-SSH-port> root@<AutoDL-SSH-host>
```

The plugin connects to `http://127.0.0.1:11000/mcp`. No application token is required. Merely connecting or listing tools does not load YuE2 or SheetSage2 model weights.

## Commands

| Command | Purpose |
| --- | --- |
| `/yue2:generate-song` | Generate a song from style and lyrics |
| `/yue2:transcribe-audio` | Upload and transcribe audio to ABC notation |
| `/yue2:task-status` | Read task state and retrieve completed artifacts |
| `/yue2:help` | Show capabilities and connection requirements |

## Server task mapping

| Runner Task | Server-side capability |
| --- | --- |
| `yue2_task` | Generate, plan, compare modes, or decode |
| `sheetsage2_task` | Transcribe uploaded audio |
| `music_score_task` | Inspect, strip chords, or compare ABC |
| `music_listen_task` | Build a downloadable listening bundle |

The plugin intentionally contains no `scripts/` directory. Claude uploads inputs, submits one of the tasks above, polls the returned task ID, and retrieves declared artifacts. Executable helpers live only in the Runner deployment.

## Repository layout

- `claude-code/` — installable `yue2` Claude Code plugin
- `claude-code/.claude-plugin/plugin.json` — plugin metadata and bundled MCP endpoint
- `claude-code/skills/yue2-music/` — automatically discovered skill, Runner contract, and vendored official resources
- `claude-code/commands/` — namespaced slash commands
- `.claude-plugin/marketplace.json` — marketplace catalog

## Validate

```bash
claude plugin validate ./claude-code
```

## Upstream resources

The skill includes official `assets/` and adapted `references/` from the [YuE2 music skill](https://github.com/multimodal-art-projection/YuE/tree/main/skills/yue2-music). They remain under the upstream Apache-2.0 license. See `claude-code/skills/yue2-music/UPSTREAM.md` for the pinned revision and adaptation boundary.
