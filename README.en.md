# Agent1st Protocols

Full documentation is in Russian: [README.md](README.md).

This repo publishes ready-to-use **agent system prompts** and task-prompting guides for DeepSeek V4 and GLM 5.2.

| Model | Protocol | File |
|-------|----------|------|
| **DeepSeek V4 Pro** | Agent1st v36 | [`agents/deepseek/agent1st_v36-pro.md`](agents/deepseek/agent1st_v36-pro.md) |
| **DeepSeek V4 Flash** | Agent1st v36 | [`agents/deepseek/agent1st_v36-flash.md`](agents/deepseek/agent1st_v36-flash.md) |
| **GLM 5.2** | Agent1st v13 | [`agents/glm/agent1st_v13-glm.md`](agents/glm/agent1st_v13-glm.md) |

This is not a skills repository. Skills live in [opencode-skills](https://github.com/dimkurilo/opencode-skills). Agent1st protocols are system prompts plus human-facing guides for writing agent tasks.

## Benchmark results

Public [ai-forever/harness-bench-fast](https://github.com/ai-forever/harness-bench-fast) runs use mechanical verification, not LLM-as-judge. Harness: OpenCode CLI with Agent1st.

| Protocol measured | Model | Result | Issue |
|-------------------|-------|--------|-------|
| agent1st_v33 | DeepSeek V4 Flash | **311/313 (99.4%)** | [issue #11](https://github.com/ai-forever/harness-bench-fast/issues/11) |
| agent1st_v10-glm | GLM 5.2 | **312/313 (99.7%)** | [issue #12](https://github.com/ai-forever/harness-bench-fast/issues/12) |

Baselines for the same models:

| Harness / setup | Model | Result | Source |
|-----------------|-------|--------|--------|
| deepagents stock | DeepSeek V4 Flash | 266/298 (89.3%) | [upstream README](https://github.com/ai-forever/harness-bench-fast#current-results) |
| OpenCode CLI with Agent1st v33 | DeepSeek V4 Flash | **311/313 (99.4%)** | [issue #11](https://github.com/ai-forever/harness-bench-fast/issues/11) |
| openclaude | GLM 5.2 | 309/313 (98.7%) | cited in [issue #12](https://github.com/ai-forever/harness-bench-fast/issues/12) |
| OpenCode CLI with Agent1st v10-glm | GLM 5.2 | **312/313 (99.7%)** | [issue #12](https://github.com/ai-forever/harness-bench-fast/issues/12) |

Published files are `v36` and `v13`, lineage successors of the measured `v33` and `v10` (newer discipline plus optional Vision Bridge sections). The exact files in this tree have not yet been claimed as a public 313-task full-suite re-bench. Details: [docs/benchmarks.md](docs/benchmarks.md).

Vision runtime is separate: [vision-bridge-opencode](https://github.com/dimkurilo/vision-bridge-opencode). Without it, agents stay text-only.

## Install

OpenCode loads each `.md` file under `~/.config/opencode/agents/` as a separate agent. Keep distinct filenames.

```bash
mkdir -p ~/.config/opencode/agents

cp agents/deepseek/agent1st_v36-pro.md   ~/.config/opencode/agents/
cp agents/deepseek/agent1st_v36-flash.md ~/.config/opencode/agents/
cp agents/glm/agent1st_v13-glm.md        ~/.config/opencode/agents/
```

Personal subfolders such as `A/` or `M/` are optional local organization. They are not required by OpenCode or this guide.

For other runtimes, paste the file body as a system prompt and adapt the frontmatter.

## Notes

Core protocol works without vision. `v36` / `v13` include optional Vision Bridge *discipline* in the system prompt (`opencode-eyesight` overview + `describe_image`). Install the runtime from **[vision-bridge-opencode](https://github.com/dimkurilo/vision-bridge-opencode)** if you need images; ignore or strip vision sections otherwise.

Design rationale: [docs/why-it-works.md](docs/why-it-works.md). License: [MIT](LICENSE).
