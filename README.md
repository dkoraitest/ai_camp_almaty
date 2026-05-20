# AI Sales Toolkit

Практические AI-инструменты для команд продаж — для участников **AI Camp Almaty 2026**, сессия «AI в продажах на практике».

Спикер: **Дмитрий Коробовцев**, AI Surfers

> **Статус:** v0.1 (work in progress). Block 1 в активной разработке к моменту начала кэмпа.

---

## Что внутри

4 практических блока — каждый = end-to-end AI-инструмент, который ты можешь запустить на своих данных и получить результат сегодня.

| # | Блок | AI-механика | Что получаешь | Статус |
|---|---|---|---|---|
| **1** | **Outbound** — обогащение + персонализированная коммуникация | Агентная разведка | Досье на компанию + 1-3 готовых opening сообщения под конкретных ЛПР | 🟡 In progress |
| **2** | **Inbound** — скоринг входящих лидов + auto-reply | Скоринг + автоматизация | Score + reasoning + готовый ответ + флаг эскалации | ⚪ Coming soon |
| **3** | **Diagnose** — почему сделки не закрываются | Диагностика корневых причин | Карта пайплайна с «дырявыми» переходами + гипотезы + проверки | ⚪ Coming soon |
| **4** | **Coach** — коучинг команды на их же диалогах | Анализ + обратная связь | Структурный фидбек по диалогу с клиентом + что упустил + что сказать иначе | ⚪ Coming soon |

---

## Quick Start

### Что нужно установить (one-time, ~15 мин)

1. **VS Code** — `code.visualstudio.com`
2. **Claude Code extension** — поиск «Claude Code» в VS Code marketplace
3. **Anthropic аккаунт** — Claude Pro или API ключ (`console.anthropic.com`)

### Запуск

```bash
git clone https://github.com/dkoraitest/ai_camp_almaty.git
cd ai_camp_almaty
code .
```

В VS Code открывается репо со встроенным Claude Code. Дальше — вызывай нужный skill из чата:

```
/enrich-and-pitch Globalink Logistics. Мы продаём: AI-инструменты для отдела продаж.
```

Skill сделает свою работу (60-120 сек) и **автоматически откроет HTML-отчёт** в браузере.

---

## Структура репозитория

```
ai-sales-toolkit/
├── .claude/skills/                    # Claude Code skills (агентные)
│   └── enrich-and-pitch/              # Block 1
│       ├── SKILL.md
│       └── templates/report.html
│
├── 01-outbound-enrichment-comms/      # Block 1 — README, examples, monday-action
├── 02-inbound-scoring/                # Block 2 (coming)
├── 03-conversion-diagnosis/           # Block 3 (coming)
├── 04-team-coaching/                  # Block 4 (coming)
│
├── reports/                           # Тут появляются твои HTML-отчёты (в .gitignore)
├── README.md
└── LICENSE
```

---

## Free стек vs опциональные платные интеграции

Все skills работают **out-of-the-box на free-стеке** (WebSearch, WebFetch, adata.kz, hh.kz, telegram-channel-parser). Не требуют API-ключей.

Для **продвинутого качества** можно подключить:
- **Apify** — глубокие LinkedIn данные ЛПР ($0.10-0.50/запрос)
- **goszakup.gov.kz token** — история госзакупок компаний (бесплатно, но нужна заявка)
- **Scrape.do** — обход блокировок ($0.10-1/запрос)

Подробности в `OPTIONAL-UPGRADES.md` каждого блока.

---

## Контакты + обратная связь

Telegram: @dkorobovtsev
Email: dkor.aitest@gmail.com

Если что-то не работает, что-то получилось не так как описано, или хочешь поделиться кейсом из своего бизнеса — пиши.

---

## License

MIT — см. [LICENSE](LICENSE).
