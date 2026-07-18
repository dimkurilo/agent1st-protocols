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
3. ** empirically grounded** — версии эволюционировали через transition docs, A/B и log analysis (см. [docs/why-it-works.md](docs/why-it-works.md)).

---

## Быстрый старт

### 1. Поставить system instruction

OpenCode подхватывает **каждый** `.md` в `~/.config/opencode/agents/` (у нас — в подпапке `A/`) как отдельного агента. Имя файла = имя агента: **не** сводите все протоколы к одному `agent1st.md`, иначе они перезапишут друг друга.

```bash
# можно поставить все три сразу — у каждого своё имя
mkdir -p ~/.config/opencode/agents/A

cp agents/deepseek/agent1st_v36-pro.md   ~/.config/opencode/agents/A/
cp agents/deepseek/agent1st_v36-flash.md ~/.config/opencode/agents/A/
cp agents/glm/agent1st_v13-glm.md        ~/.config/opencode/agents/A/
```

После копирования в списке агентов будут три разных профиля:

| Файл | Агент |
|------|--------|
| `agent1st_v36-pro.md` | DeepSeek V4 Pro |
| `agent1st_v36-flash.md` | DeepSeek V4 Flash |
| `agent1st_v13-glm.md` | GLM 5.2 |

Нужен только один — копируйте только его.  
Другой runtime (не OpenCode): вставьте тело протокола как system prompt; frontmatter (model, temperature, tools) оставьте или адаптируйте под свой CLI.

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
│   └── why-it-works.md              # основания: на чём строится
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

Подробнее: [docs/why-it-works.md](docs/why-it-works.md).

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
