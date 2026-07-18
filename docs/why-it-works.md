# Почему эти инструкции работают

Это короткая публичная выжимка. Полный research trail, transition docs, A/B и session logs остаются в приватной research-папке. Здесь только то, что помогает понять дизайн published agents.

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

Инжекционный блок из [`guides/deepseek-injection-block.md`](../guides/deepseek-injection-block.md) задаёт инженерный стиль thinking: constraints, trap detection, tool choice, execution steps. В internal stats по Hermes и DeepSeek V4 Flash с ним было меньше narrative drift и больше actionable steps.

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

Text-only модели в OpenCode могут получать visual evidence через optional Vision Bridge:

- auto: `opencode-eyesight` добавляет image overview в request;
- manual: `describe_image` отвечает на точный вопрос по изображению.

Протокол требует attribution, например `per describe_image`, и запрещает писать `I can see` без источника. Без bridge секции Vision можно игнорировать; core protocol остаётся рабочим text-agent протоколом.

---

## 6. Чего здесь нет

| Не публикуем | Почему |
|--------------|--------|
| Полный `AGENT-PROTOCOL-CHANGELOG` с v4 до v36 | research noise для end-user |
| Internal session logs и vetting reports | это не product surface |
| Все промежуточные agent versions | support surface должен указывать на latest stable |
| Vision Bridge runtime | это отдельный продукт и отдельный репозиторий |

Публичный пакет - это latest stable agents, human guides и короткое объяснение дизайна.

---

## 7. Версии в этом релизе

| Artifact | Version | Notes |
|----------|---------|-------|
| DeepSeek Pro agent | v36 | optional Vision Bridge относительно v35 |
| DeepSeek Flash agent | v36 | optional Vision Bridge относительно v35 |
| GLM agent | v13 | optional Vision Bridge относительно v12 |
| DeepSeek prompt guide | current | 3 rules |
| GLM prompt guide | current | 5 rules and meta |
| Injection block | current | DeepSeek engineering thinking |

История эволюции относится к research, а не к install path.

---

## 8. Public bench

Full-suite public scores for Agent1st lineage in OpenCode CLI:

| Measured protocol | Model | Score | Issue |
|-------------------|-------|-------|-------|
| agent1st_v33 | DeepSeek V4 Flash | 311/313 (99.4%) | [issue #11](https://github.com/ai-forever/harness-bench-fast/issues/11) |
| agent1st_v10-glm | GLM 5.2 | 312/313 (99.7%) | [issue #12](https://github.com/ai-forever/harness-bench-fast/issues/12) |

Baseline comparison:

- deepagents stock on DeepSeek V4 Flash: 266/298 (89.3%) in upstream README;
- openclaude on GLM 5.2: 309/313 (98.7%) cited in issue #12.

Details and caveats: [benchmarks.md](benchmarks.md).
