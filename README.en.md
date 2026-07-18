# Agent1st Protocols

🇷🇺 Full docs are in Russian: [README.md](README.md)

Ready-to-use **agent system instructions** and **task-prompting guides** for:

| Model | Protocol | File |
|-------|----------|------|
| **DeepSeek V4 Pro** | Agent1st v36 | [`agents/deepseek/agent1st_v36-pro.md`](agents/deepseek/agent1st_v36-pro.md) |
| **DeepSeek V4 Flash** | Agent1st v36 | [`agents/deepseek/agent1st_v36-flash.md`](agents/deepseek/agent1st_v36-flash.md) |
| **GLM 5.2** | Agent1st v13 | [`agents/glm/agent1st_v13-glm.md`](agents/glm/agent1st_v13-glm.md) |

This is **not** a skills repo. Skills live in [opencode-skills](https://github.com/dimkurilo/opencode-skills). This repo is **system prompts / agent protocols** plus human-facing prompt guides.

## Quick start

OpenCode loads **each** `.md` under `~/.config/opencode/agents/` (here: `A/`) as a **separate** agent. Keep the original filenames — do **not** rename all three to one `agent1st.md` (they would overwrite each other).

```bash
mkdir -p ~/.config/opencode/agents/A

cp agents/deepseek/agent1st_v36-pro.md   ~/.config/opencode/agents/A/
cp agents/deepseek/agent1st_v36-flash.md ~/.config/opencode/agents/A/
cp agents/glm/agent1st_v13-glm.md        ~/.config/opencode/agents/A/
```

| File | Agent |
|------|--------|
| `agent1st_v36-pro.md` | DeepSeek V4 Pro |
| `agent1st_v36-flash.md` | DeepSeek V4 Flash |
| `agent1st_v13-glm.md` | GLM 5.2 |

Install only the one(s) you need. Frontmatter is OpenCode-oriented; adapt for other CLIs.

Then write tasks with the matching guide under `guides/`.

## Core vs Vision Bridge

Core protocol works without vision. v36/v13 include optional Vision Bridge sections (`opencode-eyesight` + `describe_image`) for OpenCode text-only models — ignore or strip them if you do not use that bridge.

## Why it works

Short design rationale: [docs/why-it-works.md](docs/why-it-works.md) (Russian).

## License

MIT — see [LICENSE](LICENSE).
