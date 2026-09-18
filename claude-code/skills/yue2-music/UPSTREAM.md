# Upstream YuE2 music resources

The following resources originate from [`multimodal-art-projection/YuE`](https://github.com/multimodal-art-projection/YuE/tree/main/skills/yue2-music):

- `assets/`
- `references/abc-editing.md`
- `references/editing-workflows.md`
- `references/generation-and-covers.md`
- `references/listening-and-evaluation.md`
- `references/models-and-setup.md`
- `LICENSE`

Pinned upstream revision: `bd90e4ccae671d869b3ecaca6d7e893927d29442`.

The references are adapted so every executable workflow uses the tokenless Runner MCP deployment. The upstream `scripts/` directory is intentionally not distributed in this client plugin; corresponding implementations are installed only on the Runner host and exposed through typed Tasks/Processors. Upstream resources remain licensed under Apache License 2.0; model weights and third-party dependencies retain their own licenses.
