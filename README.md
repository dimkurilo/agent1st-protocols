# Agent1st Protocols

🇷🇺 Основной язык документации — русский · 🇬🇧 [English overview](README.en.md)

Готовые **agent system instructions** и **гайды по промптам** для:

| Модель | Протокол | Файл |
|--------|----------|------|
| **DeepSeek V4 Pro** | Agent1st v36 | [`agents/deepseek/agent1st_v36-pro.md`](agents/deepseek/agent1st_v36-pro.md) |
| **DeepSeek V4 Flash** | Agent1st v36 | [`agents/deepseek/agent1st_v36-flash.md`](agents/deepseek/agent1st_v36-flash.md) |
| **GLM 5.2** | Agent1st v13 | [`agents/glm/agent1st_v13-glm.md`](agents/glm/agent1st_v13-glm.md) |

Это **не skills** и не runtime-плагины. Это system prompts / agent protocols + практические инструкции, как ставить задачи этим моделям.

Связанный (но другой) репозиторий навыков: [opencode-skills](https://github.com/dimkurilo/opencode-skills).

---

## Зачем это

Обычный «будь полезным агентом» плохо работает на DeepSeek V4 и GLM 5.2: у моделей разные режимы thinking, разные failure modes и разная чувствительность к формулировке задачи.

Agent1st — это:

1. **System protocol** — operational rules для агента (режимы, confirmation gate, evidence, stop-signals).
2. **Prompt guides** — как человеку писать задачи, чтобы модель не тратила thinking на разбор простыни.
3. **Проверено на бенчмарке** — линейка Agent1st гонялась на [harness-bench-fast](https://github.com/ai-forever/harness-bench-fast) (см. [результаты ниже](#результаты-harness-bench-fast) и [docs/benchmarks.md](docs/benchmarks.md)).

---

## Результаты harness-bench-fast

Публичные прогоны на [ai-forever/harness-bench-fast](https://github.com/ai-forever/harness-bench-fast) — mechanical verify, без LLM-as-judge. Harness: **OpenCode CLI** + Agent1st.

| Протокол (прогон) | Модель | Result | % | Issue |
|-------------------|--------|--------|---|-------|
| **agent1st_v33** (DeepSeek) | DeepSeek V4 Flash | **311/313** | **99.4%** | [#11](https://github.com/ai-forever/harness-bench-fast/issues/11) |
| **agent1st_v10-glm** | GLM 5.2 (z.ai) | **312/313** | **99.7%** | [#12](https://github.com/ai-forever/harness-bench-fast/issues/12) |

### Сравнение с baseline того же бенчмарка

| Harness / setup | Модель | Result | % | Источник |
|-----------------|--------|--------|---|----------|
| deepagents (stock, README bench) | DeepSeek V4 Flash | 266/298 | 89.3% | [README harness-bench-fast](https://github.com/ai-forever/harness-bench-fast#current-results) |
| **opencode CLI + Agent1st v33** | DeepSeek V4 Flash | **311/313** | **99.4%** | [#11](https://github.com/ai-forever/harness-bench-fast/issues/11) |
| openclaude | GLM 5.2 | 309/313 | 98.7% | cited in [#12](https://github.com/ai-forever/harness-bench-fast/issues/12) |
| **opencode CLI + Agent1st v10-glm** | GLM 5.2 | **312/313** | **99.7%** | [#12](https://github.com/ai-forever/harness-bench-fast/issues/12) |

**Честно про версии:** полный 313-task suite снят на **предшественниках** публикуемых файлов (v33 Flash, v10 GLM). В этом репозитории лежат **v36** (DeepSeek) и **v13** (GLM) — та же линейка Agent1st, следующие итерации (vision bridge и discipline). Свежий full re-bench v36/v13 на 313 задачах пока не выложен; частичные прогоны v35 есть локально, но не как public claim.

Подробности, failures, waves: [docs/benchmarks.md](docs/benchmarks.md).

---

## Быстрый старт

### 1. Поставить system instruction

OpenCode подхватывает **каждый** `.md` в каталоге агентов как отдельного агента. Имя файла = имя агента — **не** сводите три протокола к одному `agent1st.md`.

**Публикуемый путь (как в OpenCode):**

```bash
mkdir -p ~/.config/opencode/agents

cp agents/deepseek/agent1st_v36-pro.md   ~/.config/opencode/agents/
cp agents/deepseek/agent1st_v36-flash.md ~/.config/opencode/agents/
cp agents/glm/agent1st_v13-glm.md        ~/.config/opencode/agents/
```

| Файл | Агент |
|------|--------|
| `agent1st_v36-pro.md` | DeepSeek V4 Pro |
| `agent1st_v36-flash.md` | DeepSeek V4 Flash |
| `agent1st_v13-glm.md` | GLM 5.2 |

Нужен только один — копируйте только его.

Подпапки вроде `A/`, `M/` — **личная** раскладка (разделить engineering / marketing и т.п.), не требование OpenCode и не часть этой инструкции. Если у вас уже так устроено — копируйте в свою подпапку; для публичного README достаточно `~/.config/opencode/agents/`.

Другой runtime (не OpenCode): вставьте тело протокола как system prompt; frontmatter (model, temperature, tools) оставьте или адаптируйте.

### 2. Писать задачи по гайду

| Модель | Гайд |
|--------|------|
| DeepSeek V4 | [`guides/kak-pisat-zaprosy-dlya-deepseek.md`](guides/kak-pisat-zaprosy-dlya-deepseek.md) |
| GLM 5.2 | [`guides/kak-pisat-zaprosy-dlya-glm-5.2.md`](guides/kak-pisat-zaprosy-dlya-glm-5.2.md) |

Для DeepSeek дополнительно есть injection-блок thinking mode:  
[`guides/deepseek-injection-block.md`](guides/deepseek-injection-block.md)

---

## Что внутри

```
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
│   ├── why-it-works.md              # основания: на чём строится
│   └── benchmarks.md                # harness-bench-fast results + comparison
├── README.md
├── README.en.md
├── CHANGELOG.md
└── LICENSE
```

---

## Core vs optional Vision Bridge

**Core (всегда):** режимы работы, confirmation gate, smallest effective change, evidence-first, anti-fabrication, stop-signals.

**Optional:** секции Vision Bridge (`opencode-eyesight` + `describe_image`) в v36/v13. Они нужны только если у вас стоит vision bridge для text-only моделей в OpenCode. Без bridge:

- протокол всё равно работает как text agent;
- vision-секции можно игнорировать или вырезать перед установкой.

Не путать этот репозиторий с пакетом Vision Bridge — vision bridge отдельный runtime-компонент.

---

## Почему отдельный репозиторий, а не opencode-skills

| | `agent1st-protocols` | `opencode-skills` |
|--|----------------------|-------------------|
| Содержимое | system protocols + prompt guides | skill packages (`SKILL.md` + scripts) |
| Установка | копируете agent.md / system prompt | symlink skill dirs |
| Аудитория | «хочу агента под DeepSeek/GLM» | «хочу skill в OpenCode/Grok» |

Skills ≠ agent system prompts. Смешивать их в одном root catalog ухудшает discoverability.

Позже можно поставить ссылку-кросс из `opencode-skills` README сюда (и наоборот).

---

## На чём основано (кратко)

- **DeepSeek V4:** thinking modes (Non-think / High / Max), `reasoning_effort`, IndexShare / Preserved Thinking constraints, injection block `【思维模式要求】`, discipline против mode collapse и «commentary instead of execution».
- **GLM 5.2:** thinking on/off + `reasoning_effort` (не текстом system prompt), temperature=1.0, failure modes (fabrication, tool passivity, long-session drift), 1M-context continuity.
- **Общее ядро Agent1st:** confirmation gate, path-is-clear, evidence file:line, smallest effective change, positive-action discipline.
- **Измерения:** full suite harness-bench-fast — v33 Flash 311/313, v10 GLM 312/313 ([#11](https://github.com/ai-forever/harness-bench-fast/issues/11), [#12](https://github.com/ai-forever/harness-bench-fast/issues/12)).

Подробнее: [docs/why-it-works.md](docs/why-it-works.md), [docs/benchmarks.md](docs/benchmarks.md).

---

## Имя репозитория (provisional)

Локальная папка и provisional name: **`agent1st-protocols`**.

Альтернативы, если захотите другое имя на GitHub:

| Имя | Плюс | Минус |
|-----|------|-------|
| `agent1st-protocols` | точно описывает продукт | чуть длинно |
| `agent1st` | коротко, бренд | абстрактнее |
| `deepseek-glm-agents` | SEO по моделям | не масштабируется на Qwen и др. |
| `model-agent-protocols` | нейтрально | generic |

Финальное имя — на ваш выбор; remote добавим после создания GitHub repo.

---

## License

MIT — см. [LICENSE](LICENSE).

Протоколы и гайды публикуются as-is. Модели (DeepSeek, Zhipu/GLM) — отдельные продукты соответствующих вендоров.
