# Benchmarks — harness-bench-fast

Публичные evidence, что Agent1st — не «просто красивая инструкция», а прогонялся на независимом agent coding bench.

**Бенчмарк:** [ai-forever/harness-bench-fast](https://github.com/ai-forever/harness-bench-fast)  
**Landing:** https://ai-forever.github.io/harness-bench-fast/  
**Verify:** mechanical (exact content / regex / pytest / …), **не** LLM-as-judge.

---

## Headline results (full suite, public issues)

| Agent protocol | Model | Harness | Result | % | Public issue |
|----------------|-------|---------|--------|---|--------------|
| agent1st_v33 (DeepSeek) | DeepSeek V4 Flash | opencode CLI | 311/313 | 99.4% | [issue #11](https://github.com/ai-forever/harness-bench-fast/issues/11) |
| agent1st_v10-glm | GLM 5.2 (z.ai coding plan) | opencode CLI | 312/313 | 99.7% | [issue #12](https://github.com/ai-forever/harness-bench-fast/issues/12) |

### Setup (общий)

- OpenCode CLI: каждый task = fresh `opencode run` subprocess  
- LSP off, MCP off, `--dangerously-skip-permissions`  
- concurrency 1, per-task timeout 900s  
- Evidence JSON приложены к issues

### Relation to published files in this repo

| Published here | Full-suite measured on | Note |
|----------------|------------------------|------|
| `agent1st_v36-flash.md` / `v36-pro.md` | **v33** Flash (311/313) | v36 = successor in same lineage (+ vision bridge optional) |
| `agent1st_v13-glm.md` | **v10-glm** (312/313) | v13 = successor (+ vision bridge optional) |

We do **not** claim that v36/v13 were re-run on all 313 tasks in a published issue. The numbers prove the **protocol family** under the same models, not a fresh marketing score for the exact file bytes in this tree.

---

## Comparison with harness-bench-fast distribution baselines

Upstream README «Current Results» uses **298-task** set (`task-set v0.7.0`). Our public issues used a **313-task** set (extra waves: diagnostic / memory / agentic / VCS). Scores are not bit-identical apples-to-apples across task-set sizes — use them as **same-model, different harness/protocol** signals.

### DeepSeek V4 Flash

| Setup | Tasks | Result | % | Source |
|-------|------:|--------|---|--------|
| deepagents (stock) | 298 | 266/298 | 89.3% | [README Current Results](https://github.com/ai-forever/harness-bench-fast#current-results) |
| opencode CLI + Agent1st v33 | 313 | **311/313** | **99.4%** | [issue #11](https://github.com/ai-forever/harness-bench-fast/issues/11) |

Same model family; large jump when harness + Agent1st protocol replace stock deepagents defaults.

### GLM 5.2

| Setup | Tasks | Result | % | Source |
|-------|------:|--------|---|--------|
| openclaude | 313 | 309/313 | 98.7% | cited in [issue #12](https://github.com/ai-forever/harness-bench-fast/issues/12) |
| opencode CLI + Agent1st v10-glm | 313 | **312/313** | **99.7%** | [issue #12](https://github.com/ai-forever/harness-bench-fast/issues/12) |

Same model; +3 tasks vs openclaude row on the 313-set.

---

## Wave breakdown (from public issues)

### DeepSeek V4 Flash + agent1st_v33 — 311/313

| Wave | Tasks | Passed | % |
|------|------:|-------:|---|
| core (1–30) | 30 | 30 | 100% |
| extra (31–60) | 30 | 29 | 96.7% |
| more (61–100) | 40 | 40 | 100% |
| hard (101–150) | 50 | 50 | 100% |
| extreme (151–205) | 55 | 54 | 98.2% |
| diagnostic (206–221) | 16 | 16 | 100% |
| memory (222–253) | 32 | 32 | 100% |
| agentic (254–298) | 45 | 45 | 100% |
| VCS (299–313) | 15 | 15 | 100% |
| **Total** | **313** | **311** | **99.4%** |

Failures: `task_41_count_todos`, `task_179_csv_zscore_outliers` (counting / float vs int edge cases).

### GLM 5.2 + agent1st_v10-glm — 312/313

| Wave | Tasks | Passed | % |
|------|------:|-------:|---|
| core … extreme | 205 | 205 | 100% |
| diagnostic (206–221) | 16 | 15 | 93.8% |
| memory + agentic + VCS | 92 | 92 | 100% |
| **Total** | **313** | **312** | **99.7%** |

Failure: `task_210_tar_manifest_with_hashes` (hash format edge case).

---

## What this does / does not prove

**Does prove**

- Agent1st protocols were run end-to-end on a public mechanical bench  
- Results and JSON are inspectable in GitHub issues  
- Same model can score very differently under stock harness vs protocol+opencode  

**Does not prove**

- That published v36/v13 file bytes = exact 311/312 scores (measured on v33 / v10)  
- Superiority over every harness on every task-set revision  
- That temperature / provider quirks never matter  

---

## Reproduce (sketch)

```bash
# clone harness-bench-fast, install per upstream README
# install agent file under ~/.config/opencode/agents/

OPENCODE_CONFIG=opencode-bench.json \
uv run python -m harness_bench run-cli \
  --cli-command 'opencode run -m <model> --agent <agent-name> --dangerously-skip-permissions' \
  --concurrency 1 --timeout 900 \
  --json-output results.json
```

Exact model ids and agent names from the issues:

- DeepSeek: `opencode-go/deepseek-v4-flash` + agent1st_v33  
- GLM: z.ai coding plan GLM 5.2 + agent1st_v10-glm  

For the files in this repo, swap agent name to `agent1st_v36-flash` / `agent1st_v13-glm` after copy into `~/.config/opencode/agents/`.
