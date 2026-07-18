# Как писать запросы для GLM 5.2

Пять правил + метапромпт-шаблон. GLM 5.2 — не DeepSeek, не Qwen, не GPT. Она думает иначе, и промпты под неё нужно писать иначе.

> Актуально для протокола `agent1st_v10-glm.md` (533 строки). Guide обновлён 2026-06-18.

---

## Правило 0: Забудь о «thinking» в промпте

**GLM 5.2 игнорирует prompt-директивы про thinking.** Не пиши «think step by step», «давай подумаем», «рассуждай». Thinking управляется **исключительно** через API-параметр `thinking.type`, и модель не может его переключить из промпта.

Что ещё GLM 5.2 игнорирует:
- Китайские скобки `【】` — директивы в них не работают (это для DeepSeek)
- «Включи глубокое мышление» — параметр API, не текст
- Команды `/think`, `/no_think` — не распознаются
- Персонажные инструкции («ты великий программист, который...») — менее эффективны, чем конкретный контекст

**Что работает вместо этого:** напиши чёткую цель и контекст. GLM 5.2 сама решит, сколько думать.

---

## Правило 1: Используй 4-элементную структуру (Goal → Context → Constraints → Done)

GLM 5.2 обучена на structured long-horizon задачах. Четырёх элементов достаточно, чтобы она спланировала выполнение.

```
### Goal
[1-2 предложения: что нужно сделать]

### Context
- Файлы: [пути к файлам, строкам]
- Текущее состояние: [как сейчас, если баг — что не так]
- Почему: [зачем это делается]

### Constraints
- Не трогать: [границы — что нельзя менять]
- Стиль: [язык, кодстайл, архитектурные правила]
- Версии: [зависимости, API версии]

### Done
- Проверка: [команда или сценарий — как убедиться]
- Формат результата: [что вернуть — файл, summary, diff]
```

**Почему это работает на GLM 5.2:**

| Элемент | Как GLM 5.2 это использует | reasoning_effort |
|---------|---------------------------|------------------|
| **Goal** | Активирует long-horizon планирование. Чёткая цель → модель строит траекторию | `max` |
| **Context** | DSA-механизм ищет ключевые токены. Пути файлов, имена функций, числа — то, на что модель обращает внимание | `high` |
| **Constraints** | GLM 5.2 точнее соблюдает границы, чем предыдущие версии | `high` |
| **Done** | Чёткий критерий останова — модель знает, когда остановиться, и не «передумывает» | `max` или `high` |

`reasoning_effort` — параметр API, **не часть промпта**. Подсказка, какой уровень выставлять.

---

## Правило 2: Выделяй ключевые токены

GLM 5.2 использует **DeepSeek Sparse Attention (DSA)** — механизм, который выбирает top-k релевантных токенов на каждом слое. Если важные токены не выделены, DSA может их пропустить.

**Это значит:**
- Пути файлов → обрамляй как `code` или пиши с `/` в начале
- Имена функций, классов, переменных → пиши точно, без перефразирования
- Числа, версии, даты → пиши явно: `v2.3`, `2026-06-17`
- Команды → выделяй в блоки кода

**IndexCache (arXiv:2603.12201):** смежные слои GLM-5 имеют 70-100% совпадения в выбранных top-k токенах. Первые несколько предложений промпта критически важны — они определяют, на какие токены будет смотреть вся сеть.

**Плохо:** `посмотри файл с протоколом агента и проверь секцию про температуру`

**Хорошо:**
```
### Context
- Файл: /Users/dimk/.config/opencode/agents/A/agent1st_v10-glm.md
- Секция: §2.3 Sampling & output — temperature behavior
- Сейчас: temperature: 1.0 (creative default)
```

---

## Правило 3: Опасайся инструкций, которые модель не может выполнить

GLM 5.2 — модель, она управляет только своими ответами и вызовами инструментов. Она **не может**:

| Инструкция | Почему не работает |
|-----------|-------------------|
| «Сожми траекторию, удали лишние шаги» | Модель не управляет историей сообщений — это делает харнес |
| «Примени keep-recent-k» | Модель не контролирует контекстное окно |
| «Discard tool-call history» | API не даёт модели писать в историю |
| «Переключи thinking.type на disabled» | Параметр API, модель его не контролирует |
| «Удали из контекста всё кроме последних 5 шагов» | См. выше |

**Что писать вместо этого:**
- «Составь краткий саммари выполненного — я передам его харнесу для `/compact`»
- «Сигнализируй харнесу, что контекст перегружен — выведи `## STITCH COMPRESSION`»
- «Напиши CONFIRMED_FACTS для handoff-секции»

---

## Правило 4: Учитывай особенности GLM 5.2 (v10-glm протокол)

### Температура и сэмплинг
`temperature: 1.0` по умолчанию — не меняй. Для детерминированной код-генерации можно `do_sample: false` (T игнорируется, модель берёт top-1 токен).

### Preserved Thinking (Coding Plan)
На `model: zai-coding-plan/glm-5.2` Preserved Thinking включён по умолчанию: `reasoning_content` из предыдущих витков приходит в контекст. **Не повторяй выводы — они уже в контексте.** Cache hit rate выше — длинные промпты дешевле.

### reasoning_effort
7 номинальных значений, реально различимых — 2:

| Значение | Реальность |
|----------|------------|
| `none`, `minimal`, `low`, `medium` | Маппятся на `high` (не отключают thinking) |
| `high` | Enhanced reasoning (серверный дефолт) |
| `max`, `xhigh` | Глубокое reasoning |

Для простых задач — `high`. Для сложных (debugging, архитектура, multi-file refactor) — `max`.

### 1M контекст — реально рабочий
GLM 5.2 обучена для long-horizon задач. 1M контекста — не маркетинг. State continuity > reset: GLM 5.2 trained для long-trajectory — держать работу в одной траектории обычно лучше чем фрагментировать.

### self-correction — встроенное поведение
GLM 5.2 обучена самостоятельно исправлять ошибки. После неудачного вызова: перечитай source → найди root cause → исправь минимальным изменением → перепроверь. Не проси «пожалуйста, проверь себя» — это обученное поведение.

### Stop-signals (v10)
GLM 5.2 под `max` может overthink и вытеснить output. Exit thinking когда: знаешь какой tool вызвать / повторяешься без новых данных / next action полностью сформулирован. Tool call всегда предпочтительнее ещё одного предложения reasoning.

### Positive-action discipline (v10, W2 guard)
GLM 5.2 под `max` может stall на негативных constraints («не делай X» без положительного действия). Pair every «don't» с конкретным «do»: «Don't narrate» → «Keep output to results + evidence».

---

## Правило 5: Метапромпт — template для создания промптов

**Workflow (как использовать template):**
1. Опиши задачу свободным текстом (что хочешь сделать)
2. Вставь метапромпт-template ниже
3. Попроси агента: «погрузись в контекст + составь готовый промпт по template»
4. Получи заполненный промпт → используй в новой сессии

**Метапромпт-template:**

```markdown
## GLM 5.2 Prompt Template

### Goal
[1-2 предложения: что нужно сделать]

### Context
- Файлы: [пути к файлам, строкам]
- Текущее состояние: [как сейчас, если баг — что не так]
- Почему: [зачем это делается]

### Constraints
- Не трогать: [границы — что нельзя менять]
- Стиль: [язык, кодстайл, архитектурные правила]
- Версии: [зависимости, API версии]

### Done
- Проверка: [команда или сценарий — как убедиться]
- Формат результата: [что вернуть — файл, summary, diff]
```

**Как агент заполняет template (контекст-loading правила):**
- Context → указать файлы с полными путями (DSA ищет по точным токенам)
- Если ≥3 deliverables → отметить Workflow Mode (Plan→Execute→Audit)
- Для non-trivial задач → path-is-clear gate: агент представляет plan (1-3 sentences), ждёт подтверждения
- Body changes к agent protocols → mini-test required (harness-bench, 8 tasks × 2 passes)
- Evidence-based Done: «команда + ожидаемый результат», не «probably works»

---

## Пример заполнения (реальный кейс — v32 deep audit)

```
### Goal
Провести deep audit agent1st_v32-deepseek.md — проверить grounding/empirical/consistency
каждого load-bearing claim. Найти engineering inferences представленные как facts.

### Context
- Файлы:
  - /Users/dimk/.config/opencode/agents/A/agent1st_v32-deepseek.md — аудируемый агент (920 строк)
  - /Users/dimk/Documents/trae_projects/work/projects/deepseek/evolution/deepseek/V25-V29-TRANSITION.md — design principles history
  - /Users/dimk/Documents/trae_projects/work/projects/deepseek/research/analysis/deepseek_v4.md — paper extract
- Текущее состояние: v32 = bugfix-only release поверх v31. Проверен 4 subagents, grace-anchor-lint PASS.
  НО: source-trace CSA figures показал Pro-only number (top-k=1024) применён universally (Flash=512).
- Почему: проверить есть ли ДРУГИЕ engineering inferences в протоколе представленные как facts.
  Cascade breaker уже proven non-functional (0/1300 triggers).

### Constraints
- Не трогать: сам протокол (read-only audit)
- Стиль: VS-CoT Pattern C — 5-7 гипотез с вероятностями × impact
- Не запускать bench без подтверждения

### Done
- Проверка: каждый claim классифицирован (paper fact / engineering inference / single-session / heuristic)
- Формат результата: распределение гипотез с p×I ranking + MEMORY.md update
```

---

## Быстрая памятка (шпаргалка)

```
Промпт для GLM 5.2 = Goal + Context + Constraints + Done

НЕ писать:
  × «think step by step»
  × 【китайские скобки】
  × «включи глубокое мышление»
  × «удали из контекста»
  × роль персонажа
  × «не делай X» без положительного «делай Y» (W2 stall)

Писать:
  ✓ чёткую цель (1-2 предложения)
  ✓ пути файлов как code (DSA ищет по точным токенам)
  ✓ точные версии/числа (v10, 2026-06-18, 1024)
  ✓ критерий готовности (команда + ожидаемый результат)
  ✓ что НЕ трогать (Constraints)
  ✓ первые предложения — самые важные (IndexCache)

API (не промпт):
  reasoning_effort = max | high (не low/medium — всё равно high)
  do_sample = false для детерминированного кода
  thinking.type = enabled (дефолт)

GLM 5.2 делает сама (не проси):
  × self-correction после ошибок
  × interleaved thinking между tool calls
  × DSA — выбор важных токенов
  × IndexCache — ускорение long context
  × state continuity в long sessions (1M context)

Метапромпт (создание промптов):
  опиши задачу → вставь template → «погрузись в контекст + заполни»
  → получи готовый промпт для новой сессии
```

---

## Что ценного в v10-glm (обновлено 2026-06-18)

| Фича v10 | Где | Зачем |
|----------|-----|-------|
| Path-is-clear gate | §1 | Non-trivial → подтверди plan (1-3 sentences), жди "go" |
| Stop-signals | §2.4 | Exit thinking когда action clear — anti-overthinking |
| Positive-action discipline | §4.6 | Pair every "don't" с "do" — anti-stall (W2 guard) |
| Self-Harness meta-loop | §4.4 | After 3+ failures: mine patterns → concrete rules |
| Workflow Mode | §3 | ≥3 deliverables → Plan→Execute→Audit |
| Subagent selection | §7.1 | Semantic/local → GLM-native (auditor-glm, explore-glm); web/mechanical → DeepSeek (general, verifier, review) |
| STITCH trajectory | §5 | Signal harness для /compact — модель не удаляет историю, харнес делает |
| Preserved Thinking | §2.1 | reasoning_content preserved — reuse, не re-derive |

---

*Guide обновлён под agent1st_v10-glm.md (533 строки). Предыдущая версия ссылалась на v8 (402 строки) — устарела. Метапромпт-секция добавлена 2026-06-18 по workflow пользователя.*
