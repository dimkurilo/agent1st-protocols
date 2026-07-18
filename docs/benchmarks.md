# Benchmarks: harness-bench-fast

Эта страница фиксирует публичные результаты Agent1st на [ai-forever/harness-bench-fast](https://github.com/ai-forever/harness-bench-fast). Бенч проверяет агентные coding tasks механически: exact content, regex, `pytest` и другие проверяемые условия. LLM-as-judge не используется.

Landing page бенчмарка: <https://ai-forever.github.io/harness-bench-fast/>.

---

## Headline results

Публичные full-suite прогоны:

| Agent protocol | Model | Harness | Result | % | Public issue |
|----------------|-------|---------|--------|---|--------------|
| agent1st_v33 | DeepSeek V4 Flash | OpenCode CLI | 311/313 | 99.4% | [issue #11](https://github.com/ai-forever/harness-bench-fast/issues/11) |
| agent1st_v10-glm | GLM 5.2 | OpenCode CLI | 312/313 | 99.7% | [issue #12](https://github.com/ai-forever/harness-bench-fast/issues/12) |

Общий setup:

- OpenCode CLI
- каждый task запускается как fresh `opencode run` subprocess
- LSP off, MCP off, `--dangerously-skip-permissions`
- concurrency 1
- per-task timeout 900s
- evidence JSON приложены к issues

---

## Связь с файлами в этом репозитории

| Published here | Full-suite measured on | Note |
|----------------|------------------------|------|
| `agent1st_v36-flash.md` / `agent1st_v36-pro.md` | `agent1st_v33` Flash, 311/313 | `v36` - successor той же DeepSeek lineage; optional Vision Bridge sections (runtime: [vision-bridge-opencode](https://github.com/dimkurilo/vision-bridge-opencode)) |
| `agent1st_v13-glm.md` | `agent1st_v10-glm`, 312/313 | `v13` - successor той же GLM lineage; optional Vision Bridge sections (runtime: [vision-bridge-opencode](https://github.com/dimkurilo/vision-bridge-opencode)) |

Мы не заявляем, что exact file bytes `v36` и `v13` уже прогнаны на всех 313 задачах в публичном issue. Эти числа доказывают результат protocol family на тех же моделях, а не новый score для текущих файлов.

---

## Сравнение с baseline

Upstream README показывает `deepagents` на 298-task set (`task-set v0.7.0`). Публичные issues Agent1st использовали 313-task set с дополнительными waves: diagnostic, memory, agentic, VCS. Поэтому сравнение не является byte-for-byte apples-to-apples. Это same-model сигнал по разным harness/protocol setups.

### DeepSeek V4 Flash

| Setup | Tasks | Result | % | Source |
|-------|------:|--------|---|--------|
| deepagents stock | 298 | 266/298 | 89.3% | [README Current Results](https://github.com/ai-forever/harness-bench-fast#current-results) |
| OpenCode CLI with Agent1st v33 | 313 | **311/313** | **99.4%** | [issue #11](https://github.com/ai-forever/harness-bench-fast/issues/11) |

Same model family. Разница между 266/298 и 311/313 показывает, насколько setup и системная инструкция влияют на outcome.

### GLM 5.2

| Setup | Tasks | Result | % | Source |
|-------|------:|--------|---|--------|
| openclaude | 313 | 309/313 | 98.7% | cited in [issue #12](https://github.com/ai-forever/harness-bench-fast/issues/12) |
| OpenCode CLI with Agent1st v10-glm | 313 | **312/313** | **99.7%** | [issue #12](https://github.com/ai-forever/harness-bench-fast/issues/12) |

На том же 313-task set Agent1st закрыл на 3 задачи больше, чем openclaude row из issue #12.

---

## Wave breakdown

### DeepSeek V4 Flash with agent1st_v33: 311/313

| Wave | Tasks | Passed | % |
|------|------:|-------:|---|
| core (1-30) | 30 | 30 | 100% |
| extra (31-60) | 30 | 29 | 96.7% |
| more (61-100) | 40 | 40 | 100% |
| hard (101-150) | 50 | 50 | 100% |
| extreme (151-205) | 55 | 54 | 98.2% |
| diagnostic (206-221) | 16 | 16 | 100% |
| memory (222-253) | 32 | 32 | 100% |
| agentic (254-298) | 45 | 45 | 100% |
| VCS (299-313) | 15 | 15 | 100% |
| **Total** | **313** | **311** | **99.4%** |

Failures: `task_41_count_todos`, `task_179_csv_zscore_outliers`.

### GLM 5.2 with agent1st_v10-glm: 312/313

| Wave | Tasks | Passed | % |
|------|------:|-------:|---|
| core through extreme | 205 | 205 | 100% |
| diagnostic (206-221) | 16 | 15 | 93.8% |
| memory, agentic, VCS | 92 | 92 | 100% |
| **Total** | **313** | **312** | **99.7%** |

Failure: `task_210_tar_manifest_with_hashes`.

---

## What this proves

It proves:

- Agent1st protocols were run end-to-end on a public mechanical bench.
- Results and JSON evidence are inspectable in GitHub issues.
- The same model can score very differently under different harness/protocol setups.

It does not prove:

- exact `v36` / `v13` file bytes have the exact 311/313 or 312/313 scores;
- superiority over every harness and every task-set revision;
- immunity to temperature, provider and CLI differences.

---

## Reproduce sketch

```bash
# clone harness-bench-fast and install it per upstream README
# install the agent file under ~/.config/opencode/agents/

OPENCODE_CONFIG=opencode-bench.json \
uv run python -m harness_bench run-cli \
  --cli-command 'opencode run -m <model> --agent <agent-name> --dangerously-skip-permissions' \
  --concurrency 1 --timeout 900 \
  --json-output results.json
```

Agent names used in public issues:

- DeepSeek: `opencode-go/deepseek-v4-flash`, agent `agent1st_v33`
- GLM: z.ai coding plan GLM 5.2, agent `agent1st_v10-glm`

For current files in this repo, copy them to `~/.config/opencode/agents/` and use `agent1st_v36-flash` or `agent1st_v13-glm` as the agent name.
