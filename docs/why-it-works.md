# Почему эти инструкции работают

Кратко: на чём стоит дизайн Agent1st для DeepSeek V4 и GLM 5.2, и почему модели получают разные протоколы.

---

## 1. Разные модели, разные протоколы

| | DeepSeek V4 | GLM 5.2 |
|--|-------------|---------|
| Thinking control | Non-think / High / Max через `reasoning_effort` | on/off и `reasoning_effort`; текст system prompt thinking не переключает |
| Temperature | 0.25, в основном для non-think | 1.0, типичный default для GLM agentic |
| Context | большой, но без GLM-style 1M continuity | 1M context continuity для длинной траектории |
| Частые failure modes | commentary вместо execution, mode collapse, слабая discipline injection | fabrication, tool passivity, long-session drift |

Один generic coding-agent prompt под обе модели работает хуже, чем model-specific Agent1st. DeepSeek нужно жёстче вести по режимам thinking. GLM нужно давать структуру задачи и не пытаться управлять thinking текстом.

---

## 2. Общее ядро Agent1st

Независимо от модели протокол держит несколько инвариантов:

1. Shipped working results: цель - рабочий результат, а не красивый разбор.
2. Smallest effective change: минимальное изменение, которое закрывает задачу.
3. Confirmation gate: на non-trivial задачах сначала plan, потом explicit go, потом первый modifying call.
4. Path is clear: trivial task или явный go; иначе агент не расширяет задачу по дороге.
5. Evidence-first: file:line и command output вместо «похоже, что».
6. Positive-action discipline: запрет связывается с положительным действием, чтобы модель не залипала в negative space.
7. Stop signals: явные остановки против overthinking и бесконечного exploration.

Эти правила бьют по типовым агентным сбоям (agentic failure modes): tool spam, silent scope creep, fake confidence.

---

## 3. DeepSeek-specific

### 3.1 Thinking modes как execution lanes

| Lane | Thinking | Когда |
|------|----------|-------|
| Fast | disabled | однофайловый фикс, скрипт без архитектуры |
| Explore | high | read-only investigation |
| Standard | high | multi-file feature |
| Complex | max | debug, ambiguous, risky |

В thinking modes главным регулятором остаётся `reasoning_effort`; temperature влияет заметнее в non-think.

### 3.2 Injection block `【思维模式要求】`

DeepSeek V4 хорошо реагирует на roleplay-style monologue. Для инженерной работы это вредно: thinking начинает наполняться «я думаю», «хм» и narrative drift.

Инжекционный блок из [`guides/deepseek-injection-block.md`](../guides/deepseek-injection-block.md) задаёт инженерный стиль thinking: constraints, trap detection, tool choice, execution steps. На DeepSeek V4 Flash с injection-блоком реже уходит narrative drift, чаще - actionable steps.

Публичный вывод простой: используйте блок как optional prefix для engineering tasks. Это не magic spell и не замена нормальной постановке задачи.

### 3.3 Prompt guide

Гайд для DeepSeek сводится к трём правилам:

1. Отделить «что сделать» от «почему это нужно».
2. Перед отправкой проверить, что есть действие, место и критерий готовности.
3. Не смешивать meta-формулировку задачи с execution-сессией.

DeepSeek лучше ест компактную task spec, чем длинное эссе с контекстом вперемешку.

---

## 4. GLM 5.2-specific

### 4.1 Thinking не переключается текстом

Фразы вроде «подумай шаг за шагом» в system/user prompt не управляют GLM thinking. Управление идёт через API: `thinking.type` и `reasoning_effort`. Протокол фиксирует это явно, чтобы человек и агент не ждали эффекта от prose.

### 4.2 Anchors против failure modes

`agent1st_v13-glm.md` держит anchors против нескольких сбоев:

- fabrication: не писать «я вижу» или «точно так» без evidence;
- tool passivity: если можно вызвать tool, вызывай tool;
- long-session drift: возвращаться к mission и gate в длинной сессии;
- mode collapse: не залипать в один шаблон ответа.

### 4.3 Prompt guide

GLM выигрывает от meta-сессии: сначала заполнить Task / Context / Constraints / Done, затем отправить чистый execution prompt в новую сессию. Это описано в [`guides/kak-pisat-zaprosy-dlya-glm-5.2.md`](../guides/kak-pisat-zaprosy-dlya-glm-5.2.md).

---

## 5. Vision Bridge

Text-only модели в OpenCode могут получать visual evidence через optional Vision Bridge. Runtime (плагин + MCP) - не в этом репозитории:

**[vision-bridge-opencode](https://github.com/dimkurilo/vision-bridge-opencode)**

Два слоя:

- auto: плагин `opencode-eyesight` подменяет картинку text overview в request (слабый evidence);
- manual: tool `describe_image(path, prompt)` - точный ответ vision-модели (сильный evidence).

В Agent1st `v36` / `v13` уже вшита discipline: attribution (`per describe_image` / `per auto-overview`), запрет «I can see…», когда звать manual. Без bridge эти секции можно игнорировать; core protocol остаётся text agent.

---

## 6. Версии в этом релизе

| Artifact | Version | Notes |
|----------|---------|-------|
| DeepSeek Pro agent | v36 | optional Vision Bridge sections |
| DeepSeek Flash agent | v36 | optional Vision Bridge sections |
| GLM agent | v13 | optional Vision Bridge sections |
| DeepSeek prompt guide | current | 3 rules |
| GLM prompt guide | current | 5 rules + meta template |
| Injection block | current | DeepSeek engineering thinking |

Vision Bridge runtime: [vision-bridge-opencode](https://github.com/dimkurilo/vision-bridge-opencode).

---

## 7. Public bench

Full-suite public scores for Agent1st lineage in OpenCode CLI:

| Measured protocol | Model | Score | Issue |
|-------------------|-------|-------|-------|
| agent1st_v33 | DeepSeek V4 Flash | 311/313 (99.4%) | [issue #11](https://github.com/ai-forever/harness-bench-fast/issues/11) |
| agent1st_v10-glm | GLM 5.2 | 312/313 (99.7%) | [issue #12](https://github.com/ai-forever/harness-bench-fast/issues/12) |

Baseline comparison:

- deepagents stock on DeepSeek V4 Flash: 266/298 (89.3%) in upstream README;
- openclaude on GLM 5.2: 309/313 (98.7%) cited in issue #12.

Details and caveats: [benchmarks.md](benchmarks.md).
