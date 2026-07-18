# Почему эти инструкции работают

Краткая публичная выжимка оснований. Полный research trail (transition docs, A/B, session logs) живёт в приватной research-папке; сюда — только то, что объясняет дизайн для пользователя репозитория.

---

## 1. Разные модели — разные протоколы

| | DeepSeek V4 | GLM 5.2 |
|--|-------------|---------|
| Thinking control | Non-think / High / Max + `reasoning_effort` | on/off + `reasoning_effort` (текст system prompt **не** переключает thinking) |
| Temperature | 0.25 (имеет смысл в non-think) | 1.0 (типичный default для GLM agentic) |
| Context | большой, но без «1M continuity» как у GLM | 1M-context state continuity — long-horizon discipline |
| Частые failure modes | commentary вместо execution; mode collapse; слабая дисциплина injection | fabrication; tool passivity; long-session drift |

Один generic «coding agent» prompt под обе модели даёт хуже результат, чем model-specific Agent1st.

---

## 2. Ядро Agent1st (общее)

Независимо от модели:

1. **Shipped working results** — цель = работающий результат, не красивый разбор.
2. **Smallest effective change** — минимальное изменение, которое закрывает задачу.
3. **Confirmation gate** — на non-trivial задачах plan → user go → first modifying call.
4. **Path is clear** — только trivial **или** явный go; иначе не «улучшаю по дороге».
5. **Evidence-first** — file:line / command output, не «похоже, что».
6. **Positive-action discipline** — запреты формулируются как «делай X вместо Y», чтобы модель не залипала в negative space.
7. **Stop-signals** — явные остановки против overthinking и бесконечного exploration.

Эти правила бьют по общим agentic failure modes: tool spam, silent scope creep, fake confidence.

---

## 3. DeepSeek-specific

### 3.1 Thinking modes как execution lanes

| Lane | Thinking | Когда |
|------|----------|--------|
| Fast | disabled | однофайловый фикс, скрипт без архитектуры |
| Explore | high | read-only investigation |
| Standard | high | multi-file feature |
| Complex | max | debug, ambiguous, risky |

Temperature в thinking modes почти не крутится; `reasoning_effort` — основной knob.

### 3.2 Injection block `【思维模式要求】`

DeepSeek V4 обучался на roleplay-style monologue. Инжекционный блок (см. `guides/deepseek-injection-block.md`) задаёт **инженерный** стиль thinking вместо «я думаю / хм / интересно».

Наблюдение из internal stats (Hermes + DeepSeek V4 Flash): с injection-блоком качество agent-run выше, чем без — меньше narrative drift, больше actionable steps. Публично: используйте блок как optional prefix для engineering tasks, не как magic spell.

### 3.3 Prompt guide (3 правила)

1. Отдели **что** от **почему**.
2. Перед отправкой: что сделать / где / что = done.
3. Не мешай meta-формулировку с execution-сессией.

Гайд короткий специально: DeepSeek лучше ест компактные task specs, чем эссе.

---

## 4. GLM 5.2-specific

### 4.1 Thinking не текстом

Фразы «подумай шаг за шагом» в system/user **не** управляют GLM thinking. Управление — API: `thinking.type` + `reasoning_effort`. Протокол это явно фиксирует, чтобы агент (и человек) не ждали эффекта от prose.

### 4.2 Failure-mode anchors

v13 держит behavioral anchors против:

- **fabrication** — нет «я вижу / точно так» без evidence;
- **tool passivity** — если можно вызвать tool, вызывай;
- **long-session drift** — возвращайся к mission/gate на длинной сессии;
- **mode collapse** — не залипать в один шаблон ответа.

### 4.3 Prompt guide (5 правил + meta template)

GLM выигрывает от meta-сессии: сначала заполнить шаблон Task/Scope/…, потом чистый execution prompt. Это отражено в `guides/kak-pisat-zaprosy-dlya-glm-5.2.md`.

---

## 5. Vision Bridge (optional, v36 / v13)

Text-only модели + OpenCode:

- **auto:** `opencode-eyesight` подменяет image overview текстом в request;
- **manual:** `describe_image` для точного вопроса (STRONG evidence).

Протокол требует attribution («per describe_image…») и запрещает «I can see…».  
Без bridge — игнорируйте § Vision; core protocol не ломается.

---

## 6. Чего здесь нет (намеренно)

| Не публикуем | Почему |
|--------------|--------|
| Полный AGENT-PROTOCOL-CHANGELOG (v4→v36) | research noise для end-user |
| Internal session logs / vetting reports | не product surface |
| Все промежуточные agent versions | support surface = latest stable only |
| Vision Bridge runtime | отдельный продукт/репозиторий |

Публичный пакет = **latest stable agents + human guides + why**.

---

## 7. Версии в этом релизе

| Artifact | Version | Notes |
|----------|---------|--------|
| DeepSeek Pro agent | v36 | + optional vision bridge vs v35 |
| DeepSeek Flash agent | v36 | + optional vision bridge vs v35 |
| GLM agent | v13 | + optional vision bridge vs v12 |
| DeepSeek prompt guide | current | 3 rules |
| GLM prompt guide | current | 5 rules + meta |
| Injection block | current | DeepSeek engineering thinking |

Если нужна история эволюции — это research, не install path.

---

## 8. Public bench (harness-bench-fast)

Full-suite public scores (Agent1st lineage, OpenCode CLI):

| Measured protocol | Model | Score | Issue |
|-------------------|-------|-------|-------|
| v33 DeepSeek | DeepSeek V4 Flash | 311/313 (99.4%) | [#11](https://github.com/ai-forever/harness-bench-fast/issues/11) |
| v10 GLM | GLM 5.2 | 312/313 (99.7%) | [#12](https://github.com/ai-forever/harness-bench-fast/issues/12) |

vs stock deepagents DeepSeek V4 Flash 266/298 (89.3%) in upstream README; vs openclaude GLM 5.2 309/313 (98.7%).

Details and caveats (task-set 298 vs 313, v36/v13 not re-scored on full suite): [benchmarks.md](benchmarks.md).
