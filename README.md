# Agent1st Protocols

Основной язык документации - русский. Короткий английский обзор: [README.en.md](README.en.md).

Готовые системные инструкции (system prompts) и гайды по постановке задач для агентных моделей:

| Модель | Протокол | Файл |
|--------|----------|------|
| **DeepSeek V4 Pro** | Agent1st v36 | [`agents/deepseek/agent1st_v36-pro.md`](agents/deepseek/agent1st_v36-pro.md) |
| **DeepSeek V4 Flash** | Agent1st v36 | [`agents/deepseek/agent1st_v36-flash.md`](agents/deepseek/agent1st_v36-flash.md) |
| **GLM 5.2** | Agent1st v13 | [`agents/glm/agent1st_v13-glm.md`](agents/glm/agent1st_v13-glm.md) |

Это не skills и не runtime-плагины. В репозитории лежат тела системных инструкций для агентов и практические гайды, как ставить задачи DeepSeek V4 и GLM 5.2. Skills живут отдельно: [opencode-skills](https://github.com/dimkurilo/opencode-skills).

---

## Результаты harness-bench-fast

[ai-forever/harness-bench-fast](https://github.com/ai-forever/harness-bench-fast) - агентный coding benchmark с механической проверкой (mechanical verify). Проверка идёт через точное содержимое, регулярные выражения, `pytest` и похожие проверяемые условия, без LLM-as-judge.

Публичные прогоны Agent1st в OpenCode CLI:

| Протокол в прогоне | Модель | Result | % | Issue |
|--------------------|--------|--------|---|-------|
| **agent1st_v33** | DeepSeek V4 Flash | **311/313** | **99.4%** | [issue #11](https://github.com/ai-forever/harness-bench-fast/issues/11) |
| **agent1st_v10-glm** | GLM 5.2 | **312/313** | **99.7%** | [issue #12](https://github.com/ai-forever/harness-bench-fast/issues/12) |

Сравнение с baseline того же бенчмарка:

| Harness / setup | Модель | Result | % | Источник |
|-----------------|--------|--------|---|----------|
| deepagents stock | DeepSeek V4 Flash | 266/298 | 89.3% | [README harness-bench-fast](https://github.com/ai-forever/harness-bench-fast#current-results) |
| **OpenCode CLI with Agent1st v33** | DeepSeek V4 Flash | **311/313** | **99.4%** | [issue #11](https://github.com/ai-forever/harness-bench-fast/issues/11) |
| openclaude | GLM 5.2 | 309/313 | 98.7% | cited in [issue #12](https://github.com/ai-forever/harness-bench-fast/issues/12) |
| **OpenCode CLI with Agent1st v10-glm** | GLM 5.2 | **312/313** | **99.7%** | [issue #12](https://github.com/ai-forever/harness-bench-fast/issues/12) |

Честная оговорка по версиям: полный suite на 313 задач снят на `agent1st_v33` для DeepSeek V4 Flash и `agent1st_v10-glm` для GLM 5.2. В этом репозитории лежат `v36` и `v13` - продолжатели той же линейки. Они добавляют, среди прочего, optional Vision Bridge и дисциплину новых версий, но публичного full re-bench именно этих файлов на 313 задачах пока нет.

Подробности по setup, breakdown и failures: [docs/benchmarks.md](docs/benchmarks.md).

---

## Зачем это

Обычный «будь полезным агентом» даёт просадку на DeepSeek V4 и GLM 5.2. У моделей разные режимы мышления (thinking modes), разные типовые сбои (failure modes) и разная чувствительность к формулировке задачи.

Agent1st состоит из трёх частей:

1. Системный протокол (system protocol): режимы работы, confirmation gate, evidence, stop signals.
2. Гайды по задачам (prompt guides): как человеку писать запросы, чтобы модель не тратила мышление на разбор простыни.
3. Проверка на harness-bench-fast: публичные mechanical-verify прогоны с issue-ссылками.

---

## Быстрый старт

### 1. Поставить системную инструкцию

OpenCode подхватывает каждый `.md` в `~/.config/opencode/agents/` как отдельного агента. Имя файла становится именем агента, поэтому файлы должны называться по-разному. Не сводите три протокола к одному `agent1st.md`.

```bash
mkdir -p ~/.config/opencode/agents

cp agents/deepseek/agent1st_v36-pro.md   ~/.config/opencode/agents/
cp agents/deepseek/agent1st_v36-flash.md ~/.config/opencode/agents/
cp agents/glm/agent1st_v13-glm.md        ~/.config/opencode/agents/
```

| Файл | Агент |
|------|-------|
| `agent1st_v36-pro.md` | DeepSeek V4 Pro |
| `agent1st_v36-flash.md` | DeepSeek V4 Flash |
| `agent1st_v13-glm.md` | GLM 5.2 |

Нужен один агент - копируйте только его.

Подпапки вроде `A/` и `M/` - личная организация, например чтобы разделить engineering и marketing агентов. Это не требование OpenCode и не часть публичной инструкции. Для обычной установки достаточно `~/.config/opencode/agents/`.

Другой runtime: вставьте тело файла как системную инструкцию (system prompt). Frontmatter с model, temperature и tools оставьте или адаптируйте под свой CLI.

### 2. Писать задачи по гайду

| Модель | Гайд |
|--------|------|
| DeepSeek V4 | [`guides/kak-pisat-zaprosy-dlya-deepseek.md`](guides/kak-pisat-zaprosy-dlya-deepseek.md) |
| GLM 5.2 | [`guides/kak-pisat-zaprosy-dlya-glm-5.2.md`](guides/kak-pisat-zaprosy-dlya-glm-5.2.md) |

Для DeepSeek есть отдельный injection-блок thinking mode: [`guides/deepseek-injection-block.md`](guides/deepseek-injection-block.md).

---

## Что внутри

```text
agent1st-protocols/
├── agents/
│   ├── deepseek/
│   │   ├── agent1st_v36-pro.md      # DeepSeek V4 Pro
│   │   └── agent1st_v36-flash.md    # DeepSeek V4 Flash
│   └── glm/
│       └── agent1st_v13-glm.md      # GLM 5.2
├── guides/
│   ├── kak-pisat-zaprosy-dlya-deepseek.md
│   ├── kak-pisat-zaprosy-dlya-glm-5.2.md
│   └── deepseek-injection-block.md
├── docs/
│   ├── why-it-works.md
│   └── benchmarks.md
├── README.md
├── README.en.md
├── CHANGELOG.md
└── LICENSE
```

---

## Core и optional Vision Bridge

Core protocol работает всегда: режимы работы, confirmation gate, smallest effective change, evidence-first, anti-fabrication и stop signals.

В `v36` и `v13` также есть optional Vision Bridge sections для OpenCode: `opencode-eyesight` и `describe_image`. Они нужны, если text-only модель должна получать описание изображения. Без Vision Bridge протокол продолжает работать как text agent; vision-секции можно игнорировать или вырезать перед установкой.

Vision Bridge - отдельный runtime-компонент. Этот репозиторий публикует только agent protocols и гайды.

---

## Почему отдельный репозиторий

| | `agent1st-protocols` | `opencode-skills` |
|--|----------------------|-------------------|
| Содержимое | system protocols и prompt guides | skill packages (`SKILL.md` и scripts) |
| Установка | копирование agent `.md` или вставка system prompt | symlink skill dirs |
| Аудитория | разработчики, которым нужен агент под DeepSeek или GLM | пользователи OpenCode/Grok skills |

Skills и agent system prompts - разные поверхности. Если смешать их в одном каталоге, хуже становится и установка, и аудит.

---

## На чём основано

- DeepSeek V4: thinking modes, `reasoning_effort`, IndexShare / Preserved Thinking constraints, injection block `【思维模式要求】`, защита от mode collapse и commentary вместо execution.
- GLM 5.2: thinking on/off через API, `reasoning_effort`, `temperature: 1.0`, защита от fabrication, tool passivity, long-session drift, поддержка 1M context.
- Общее ядро Agent1st: confirmation gate, path-is-clear, evidence file:line, smallest effective change, positive-action discipline.
- Измерения: full suite harness-bench-fast, v33 Flash 311/313 и v10 GLM 312/313, [issue #11](https://github.com/ai-forever/harness-bench-fast/issues/11) и [issue #12](https://github.com/ai-forever/harness-bench-fast/issues/12).

Подробнее: [docs/why-it-works.md](docs/why-it-works.md), [docs/benchmarks.md](docs/benchmarks.md).

---

## Имя репозитория

Локальная папка и provisional name: `agent1st-protocols`.

| Имя | Плюс | Минус |
|-----|------|-------|
| `agent1st-protocols` | точно описывает продукт | чуть длинно |
| `agent1st` | коротко, бренд | абстрактнее |
| `deepseek-glm-agents` | SEO по моделям | не масштабируется на Qwen и другие модели |
| `model-agent-protocols` | нейтрально | generic |

Финальное имя - на ваш выбор; remote можно добавить после создания GitHub repo.

---

## License

MIT - см. [LICENSE](LICENSE).

Протоколы и гайды публикуются as-is. Модели DeepSeek и Zhipu/GLM - отдельные продукты соответствующих вендоров.
