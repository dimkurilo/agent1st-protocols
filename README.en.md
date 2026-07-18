# Agent1st Protocols

🇷🇺 Full docs are in Russian: [README.md](README.md)

Ready-to-use **agent system instructions** and **task-prompting guides** for:

| Model | Protocol | File |
|-------|----------|------|
| **DeepSeek V4 Pro** | Agent1st v36 | [`agents/deepseek/agent1st_v36-pro.md`](agents/deepseek/agent1st_v36-pro.md) |
| **DeepSeek V4 Flash** | Agent1st v36 | [`agents/deepseek/agent1st_v36-flash.md`](agents/deepseek/agent1st_v36-flash.md) |
| **GLM 5.2** | Agent1st v13 | [`agents/glm/agent1st_v13-glm.md`](agents/glm/agent1st_v13-glm.md) |

This is **not** a skills repo. Skills live in [opencode-skills](https://github.com/dimkurilo/opencode-skills). This repo is **system prompts / agent protocols** plus human-facing prompt guides.

## Benchmark results (harness-bench-fast)

Public runs on [ai-forever/harness-bench-fast](https://github.com/ai-forever/harness-bench-fast) (mechanical verify, OpenCode CLI + Agent1st):

| Protocol measured | Model | Result | Issue |
|-------------------|-------|--------|-------|
| agent1st_v33 | DeepSeek V4 Flash | **311/313 (99.4%)** | [#11](https://github.com/ai-forever/harness-bench-fast/issues/11) |
| agent1st_v10-glm | GLM 5.2 | **312/313 (99.7%)** | [#12](https://github.com/ai-forever/harness-bench-fast/issues/12) |

Baselines (same models, different harness/protocol): deepagents DeepSeek V4 Flash **266/298 (89.3%)** in [upstream README](https://github.com/ai-forever/harness-bench-fast#current-results); openclaude GLM 5.2 **309/313 (98.7%)** cited in #12.

Published files here are **v36 / v13** (successors of the measured v33 / v10). Full re-bench of exact v36/v13 file bytes on 313 tasks is not claimed yet — see [docs/benchmarks.md](docs/benchmarks.md).

## Quick start

OpenCode loads **each** `.md` in `~/.config/opencode/agents/` as a **separate** agent. Keep distinct filenames — do **not** rename all three to one `agent1st.md`.

```bash
mkdir -p ~/.config/opencode/agents

cp agents/deepseek/agent1st_v36-pro.md   ~/.config/opencode/agents/
cp agents/deepseek/agent1st_v36-flash.md ~/.config/opencode/agents/
cp agents/glm/agent1st_v13-glm.md        ~/.config/opencode/agents/
```

| File | Agent |
|------|--------|
| `agent1st_v36-pro.md` | DeepSeek V4 Pro |
| `agent1st_v36-flash.md` | DeepSeek V4 Flash |
| `agent1st_v13-glm.md` | GLM 5.2 |

Personal subfolders (`A/`, `M/`, …) are optional organization — not required by OpenCode or this guide.

Install only the one(s) you need. Frontmatter is OpenCode-oriented; adapt for other CLIs. Write tasks with the matching guide under `guides/`.

## Core vs Vision Bridge

Core protocol works without vision. v36/v13 include optional Vision Bridge sections (`opencode-eyesight` + `describe_image`) for OpenCode text-only models — ignore or strip them if you do not use that bridge.

## Why it works

Short design rationale: [docs/why-it-works.md](docs/why-it-works.md) (Russian).

## License

MIT — see [LICENSE](LICENSE).
