---
name: enrich-and-pitch
description: |
  B2B outbound enrichment для Казахстана: компания (название или БИН) →
  идентификация ЛПР → personal context per ЛПР → 1-3 personalized opening messages.
  Работает на free-стеке (WebSearch, WebFetch, adata.kz, hh.kz, telegram-channel-parser),
  опционально подхватывает paid интеграции (Apify, goszakup token, Scrape.do).
  Triggers: enrich and pitch, обогащение и сообщения, контекст-360, контекст 360,
  outbound research, найти ЛПР и написать, personalized outreach, исходящий контакт KZ,
  персонализированное сообщение, изучить компанию казахстан.
---

# Enrich-and-Pitch — B2B Outbound Research & Generation (Казахстан)

End-to-end пайплайн для персонализированного outbound: от названия компании до готовых opening сообщений под конкретных ЛПР с публично подтверждаемым контекстом.

**Архитектура отчёта — две вкладки в одном HTML:**
- **Аналитика** — всё что мы знаем (профиль, hiring, market, LinkedIn, social — что собрано на текущем шаге)
- **Коммуникация** — как мы пишем (эволюция hook + message через шаги, latest сверху, previous приглушённые)

**ЯЗЫК — простой русский, без жаргона:**
- ✅ «Заход», «С чего зайти», «Что для него важно», «Где совпадаем» · ❌ «Hook», «Persona-fit», «outreach», «buyer»
- ✅ «Главный», «Второстепенный» · ❌ «Primary», «Secondary», «gatekeeper»
- ✅ «Текущая / Предыдущая версия», «База (бесплатно)», «+ Рынок», «+ LinkedIn», «+ Соцсети» · ❌ «Current / Previous», «Free baseline», «+ Market», «+ Social»
- ✅ «Свежие события», «Активность по найму», «Кто принимает решение» · ❌ «Recent events», «Hiring intelligence», «ЛПР»
- ✅ «Бесплатные источники», «Платные источники», «через Apify» · ❌ «free стек», «paid», «paid stack»
- ✅ «AI-агенты», «инструменты для отдела продаж», «оценка лидов» · ❌ «agentic AI», «sales tools», «lead scoring»
- Прямые цитаты людей оставляем на оригинальном языке (если их LinkedIn на английском) + переводим в скобках для контекста

Цель: продавец-неангличанин читает отчёт без напряга. Жаргон скрывает выводы.

**КЛЮЧЕВОЙ ПРИНЦИП Analytics tab — выводы, не данные:**
Сейл за 10 секунд должен понять что ВАЖНО, а не получить data dump. Каждая секция отвечает на вопрос «что я с этим делаю».

| Что | Как делать ✅ | Как НЕ делать ❌ |
|---|---|---|
| Каждая секция | 1 заголовок + 2-4 строки выводов | Fact-grid с 8-10 rows |
| Длинные данные | <details> collapsed по умолчанию | Всё всегда видно |
| Профиль компании | 3 строки (сектор + recent + клиенты) | Полный grid юр.данных на виду |
| Hiring | 3-5 буллетов выводов + collapsed список вакансий | Раскладка по 13 позициям |
| Market | 3 буллета напряжений + collapsed конкуренты | Sector definition + 5+ конкурентов + 7 dynamics |
| LinkedIn | 1 ключевая цитата + 3-4 факта + collapsed full | 8 fact-rows headline/about/edu/awards/skills |
| Полные досье | Внутри collapsed `<details>` | Всегда видимый раздел |
| Источники | Внутри collapsed `<details>` | Длинный список на виду |

Принцип: **«что брать в outreach»** > «что мы нашли».

**4-шаговая ladder эволюции hook/message:**
| Шаг | Что добавляем | Free/Paid | Hook становится |
|---|---|---|---|
| **1** | Company + ЛПР + Hiring (free) | Free | v1 |
| **2** | Market context (free, через Claude) | Free | v2 |
| **3** | LinkedIn deep dive (Apify) | Paid | v3 |
| **4** | Social signals FB/IG/TikTok (Apify) | Paid | v4 |

Каждый шаг → regenerate hook + message → новая версия в Communication tab. Latest становится current, предыдущие — previous (dimmed).

## Когда использовать

Пользователь просит:
- Исследовать компанию и подготовить outbound коммуникацию
- Сделать «контекст-360» по компании / ЛПР
- Найти decision-makers в компании и написать первое сообщение
- Подготовиться к холодному звонку / письму в KZ B2B

## Входные данные

**Обязательно:**
1. **Компания** — название (например, «Globalink Logistics») или БИН (например, 110842011929)
2. **Наш оффер** — 1-2 предложения о том, что продаём (например: «AI-инструменты для отдела продаж: лидген, скоринг входящих, аналитика звонков»)

**Опционально:**
3. **Target роль** — Buyer / Champion / User / Decision Maker если известно
4. **Сигналы для использования** — recent hiring, funding, expansion, события

Если каких-то обязательных входов не хватает — спроси у пользователя ДО старта.

## Workflow — 4 стейджа

### Stage 1: Company Discovery

**Цель:** факт-чекнутый профиль компании.

Параллельно выполни:

1. **WebSearch** — найти БИН и идентификаторы:
   - `"<company>" Казахстан БИН`
   - `site:adata.kz "<company>"` → получи URL вида `pk.adata.kz/counterparty/main/company/{БИН}/basic-info`
   - Если на вход дали БИН — пропусти этот шаг

2. **WebFetch adata.kz** на `https://pk.adata.kz/counterparty/main/company/{БИН}/basic-info`:
   - Полное юридическое название
   - БИН (верификация)
   - Статус (активна / ликвидирована / приостановлена)
   - Директор (имя)
   - Дата регистрации
   - Адрес
   - ОКЭД (вид деятельности)
   - Размер (по числу сотрудников)
   - Регистрирующий орган

3. **WebFetch официального сайта** (если SSL валидный):
   - `<domain>/about` или `<domain>/o-kompanii`
   - `<domain>/team` или `<domain>/komanda` или `<domain>/leadership`
   - `<domain>/news` или `<domain>/press`
   - Если WebFetch падает на SSL → fallback: `WebSearch site:<domain> "<keyword>"`

4. **WebSearch hh.kz** для employer_id:
   - `site:hh.kz "<company>" вакансии` → найди employer URL вида `hh.kz/employer/{id}`

5. **WebFetch ПОЛНЫЙ список вакансий** (не employer-страницу):
   - URL pattern: `https://almaty.hh.kz/search/vacancy?employer_id={id}&items_on_page=50`
   - Если редирект — следуй на almaty.hh.kz или astana.hh.kz
   - Извлеки список ВСЕХ открытых вакансий: title, локация, диапазон зарплаты, требуемый опыт

6. **Hiring intelligence — КОМПАКТНО. 3 блока, никакой воды:**

   **A. Stats — 1 строка:**
   `N вакансий · Астана X / Алматы Y · ЗП мин–макс ₸ · ссылка на источник`

   **B. Кого нанимают (3-5 буллетов):** группировка функций + ⭐ для критичных позиций
   - Группируй похожие роли в одну строку («Customer Success — 4 роли»)
   - Senior leadership / новые функции → отдельный буллет с ⭐
   - НЕ перечисляй все 13 вакансий по одной — только агрегированные блоки

   **C. Где нет людей (2-3 буллета): значимые ОТСУТСТВИЯ**
   - «0 AI/ML вакансий — внутренняя команда укомплектована или outsource»
   - «0 системных Sales — продажи ещё не масштабируются»
   - «0 Marketing/Brand — bottom-of-funnel не приоритет»
   - Каждое отсутствие → одно интерпретирующее предложение

   **D. Что это значит — 1-2 предложения, добавляет интерпретацию (не дублирует bullets):**
   - Какой моментум: formation новой функции / scaling / replacement?
   - Как связан с recent events?
   - **Пример GOOD:** «Скейлят CS на существующей базе + формируют новую международную партнёрскую функцию после интеграции с Контур. Sales infrastructure ещё не построена — окно для встраивания тулсов ДО legacy.»
   - **Пример BAD:** «Активно нанимают, открыты 13 вакансий» (без интерпретации)
   - **Пример BAD:** длинное эссе на абзац (слишком много)

   Правила гигиены:
   - Весь блок Hiring Intelligence помещается на 1/3 экрана без скролла
   - Stats — 1 строка
   - Кого нанимают — 3-5 буллетов
   - Где нет людей — 2-3 буллета
   - Что это значит — 1-2 короткие фразы

7. **WebSearch news**:
   - `"<company>" 2026 OR 2025 новости`
   - `"<company>" интервью OR пресс-релиз`
   - `"<company>" "назначен" OR "возглавил" OR "новый руководитель"`

8. **(Опционально, paid)** Если `$GOSZAKUP_TOKEN` доступен:
   - REST: `GET https://ows.goszakup.gov.kz/v3/subject/biin/{BIN}` + Bearer header → детали + сотрудники
   - REST: `GET https://ows.goszakup.gov.kz/v3/contract?supplier_biin={BIN}` → выигранные госконтракты
   - REST: `GET https://ows.goszakup.gov.kz/v3/rnu/{BIN}` → проверка по реестру недобросовестных

**Output Stage 1:** структурированный профиль компании + hiring intelligence (1 главный инсайт + категоризация ролей + ⭐ звёздные позиции). Каждый факт с источником.

---

### Stage 2: ЛПР Identification

**Цель:** 1-3 конкретных человека, принимающих решения по нашему офферу.

1. **Reasoning step (без вызова инструментов).** На основе:
   - Профиля компании из Stage 1
   - Описания нашего оффера
   
   Определи **роли** ЛПР, релевантных нашему предложению:
   - Для AI sales tools: Commercial Director / Head of Sales / Chief Sales Officer / CMO / Founder
   - Для ERP/IT: CIO / CFO / COO / CTO / Founder
   - Для логистики: Operations Director / Supply Chain Lead / CEO
   - Подумай: кто Buyer (бюджет), Champion (продвинет внутри), User (будет использовать)

2. **Найди имена для каждой роли:**
   - Директор уже известен из adata.kz (Stage 1)
   - С официального сайта /team — другие топы (если есть)
   - **WebSearch** `"<company>" CEO OR директор OR президент`
   - **WebSearch** `"<company>" "<specific role>"` (например, "коммерческий директор")
   - **WebSearch** `site:linkedin.com/in/ "<company>"` → LinkedIn snippets с именами и должностями
   - **WebSearch** `"<company>" "назначен" OR "возглавил"` → recent appointments

3. **(Опционально, paid)** Если `$APIFY_API_KEY` доступен:
   - Вызов Apify actor `LinkedIn Company People Search` → полный список сотрудников с должностями

**Output Stage 2:** 1-3 человека с именем, ролью, обоснованием «почему этот человек ЛПР для нашего оффера».

**Failure modes:**
- Если найден 0 ЛПР → стоп. Зарепортить: «недостаточно публичной информации о руководстве. Рекомендую: outreach на общий info@ или сначала установить контакт через LinkedIn по company name».
- Если найдено >3 → выбери топ-3 наиболее релевантных нашему офферу.

---

### Stage 3: Personal Deep Dive

**Цель:** context-360 на каждого выявленного ЛПР.

Для каждого ЛПР параллельно:

1. **WebSearch** `"<name>" "<company>"` → подтверждение роли + public mentions
2. **WebSearch** `"<name>" интервью OR подкаст OR выступление` → public statements
3. **WebSearch** `"<name>" site:forbes.kz OR site:kursiv.kz OR site:kapital.kz OR site:inbusiness.kz` → KZ business медиа
4. **WebSearch** `"<name>" site:linkedin.com/in/` → LinkedIn snippets из Google
5. **WebSearch** `"<name>" telegram канал OR t.me/` → проверь наличие TG-канала
   - **Если канал найден** → вызови **telegram-channel-parser** на канал → последние 20 постов
6. **WebSearch** `"<name>" награды OR премии` → recognition signals
7. **(Опционально, paid)** Если `$APIFY_API_KEY`: full LinkedIn profile + last 20 posts через Apify

Для каждого ЛПР собери:
- **Background**: история, predecessor roles, education если public
- **Recent activity** (последние 3-6 мес): посты, выступления, интервью, события
- **Themes**: о чём консистентно говорит (например, «scaling Eurasian corridor», «digital transformation», «AI in operations»)
- **Specific recent events**: конкретные интервью / награды / mentions

**Output Stage 3:** context dossier на каждого ЛПР с указанием источников.

---

### Stage 4: Synthesis (только заходы, без сообщений)

**ВАЖНО:** сообщения не генерируем. Только заходы. Продавец сам напишет сообщение на основе захода — у него есть контекст канала (LinkedIn DM vs email vs WhatsApp), срочности, тона. Наша работа — дать ему *insight*, не готовый текст.

#### Stage 4a — Person-First Action Summary (обязательное)

**Цель:** для каждого PRIMARY ЛПР собрать «карточку действий для сейла» — что нужно знать за 10 секунд.

**Что включает person-card:**

1. **В приоритете сейчас** (2-4 буллета) — что у этого ЛПР *на тарелке прямо сейчас*:
   - Recent moves (последние 3-6 мес) с датами / именами / источниками
   - Что лично говорит / постит / выступает
   - На что он публично потратил время, деньги, внимание
   - **НЕ обобщения**: «занимается AI» = плохо. «Запустил D8N.ai в 2025, инвестировал $100K в MOST Fund в Q4» = хорошо.

2. **С чего зайти** (1 событие + опорная фраза) — конкретный opening:
   - ОДНО событие/факт (свежее, верифицируемое)
   - Опорная фраза которая привязывает событие к диалогу
   - Источник для этого события (чтобы сейл мог проверить)

3. **Стык с нашим оффером** (1-2 фразы) — где их фокус пересекает наш продукт:
   - **Обязательно**. Без этого карточка не имеет ценности для сейла.
   - Не «у нас классный продукт». А «они делают X — наш Y усиливает X»

**Приоритизация ЛПР для карточек:**

Per-tier hierarchy для ВЫБОРА кого включать в person-cards:

| Tier | Кто это | Включать в Главное? |
|---|---|---|
| Primary | Buyer / Champion / Decision Maker по нашему офферу | ✅ Полная карточка |
| Secondary | Gatekeeper / co-decision (юрист / финансист) | ❌ Только в «Идентифицированные ЛПР» list. Без person-card. |

**Правила гигиены:**
- 1-3 карточки максимум, только primary ЛПР
- Если только 1 primary найден — 1 карточка. Не заполняй искусственно.
- Если 0 primary — стоп. Сообщи: «не нашёл persona-level контекста для personalized outreach, рекомендую company-level подход».

**Хороший пример (Documentolog → Канафин):**

```
В приоритете сейчас:
• Cross-border расширение через Контур (фев 2026)
• Запуск D8N.ai в продакшен + укомплектование AI Research dept
• Формирование международной партнёрской функции (открыта senior позиция)

С чего зайти:
Открытая вакансия руководителя международных партнёрских продаж —
formation moment коммерческой функции после Контур. Конкретное публичное событие, можно сослаться напрямую.

Стык с нашим оффером:
Канафин уже строит вертикальный AI (D8N.ai) и инвестирует в B2B AI
(MOST Fund). Наш стек — «D8N.ai для коммерческих команд». Категория,
которую он сам признаёт ценной.
```

**Плохой пример (без personalization):**

```
В приоритете сейчас:
• Развивает компанию
• Расширяется
• Внедряет AI

С чего зайти:
Поговорить про AI

Стык с нашим оффером:
У нас есть AI-решение для продаж.
```

### Stage 5: HTML Report Generation & Auto-Open

**Цель:** красивый, копируемый HTML-отчёт, автоматически открывается в браузере.

1. **Подготовь данные** для подстановки в шаблон (все собранные на Stage 1-4 факты + сообщения).

2. **Прочитай шаблон**:
   ```
   Read: .claude/skills/enrich-and-pitch/templates/report.html
   ```
   В конце шаблона есть HTML-комментарий с описанием всех placeholder'ов и структур повторяющихся блоков (LPR_CARDS, DOSSIERS, SOURCES) — следуй им строго.

3. **Сгенерируй заполненный HTML**:
   - **Header / meta**: `{{COMPANY_NAME}}`, `{{BIN}}`, `{{TIMESTAMP}}` (формат «20 мая 2026, 14:30»), `{{STACK_TAG}}`, `{{STACK}}`, `{{RUNTIME}}`
   - **`{{PERSON_CARDS}}`** ⭐ — главная новая секция «Главное»: 1-3 person-action карточки из Stage 4a (только PRIMARY ЛПР). Структура карточки в template-комментариях.
   - **`{{COMPANY_FACTS_CORE}}` + `{{COMPANY_FACTS_LEGAL}}`** — компания: core видимые факты + legal в collapsed `<details>`
   - **Hiring intelligence**: `{{HIRING_STATS}}` (1 строка), `{{HIRING_WHO}}` (3-5 буллетов), `{{HIRING_GAPS}}` (2-3 буллета), `{{HIRING_MEANING}}` (1-2 фразы вывода)
   - **`{{LPR_CARDS}}`** — list всех найденных ЛПР с ролями + tier (primary / secondary)
   - **`{{DOSSIERS}}`** — полные досье per-ЛПР из Stage 3+4b. Для каждого dossier секции добавь `id="dossier-<slug>"` чтобы anchor-ссылки из person-cards работали. Внутри message-box добавь `id="message-<slug>"`.
   - **`{{DEEP_RESEARCH_PROMPT}}`** ⭐ — готовый промпт для Step 2 (см. Stage 6 ниже)
   - **`{{SOURCES}}`** — все использованные URL из всех стейджей

4. **Определи путь файла**:
   ```bash
   COMPANY_SLUG=$(echo "<COMPANY_NAME>" | tr '[:upper:]' '[:lower:]' | tr ' ' '-' | tr -cd 'a-z0-9-')
   TIMESTAMP=$(date +%Y%m%d-%H%M)
   REPORT_PATH="reports/${COMPANY_SLUG}-${TIMESTAMP}.html"
   ```

5. **Создай папку `reports/` если её нет + добавь в `.gitignore`** (чтобы пользовательские отчёты не пачкали репо):
   ```bash
   mkdir -p reports
   if [ -f .gitignore ] && ! grep -q "^reports/$" .gitignore; then
     echo "reports/" >> .gitignore
   fi
   ```

6. **Сохрани HTML** через Write tool на `$REPORT_PATH`.

7. **Автооткрытие в браузере** (cross-platform):
   ```bash
   open "$REPORT_PATH" 2>/dev/null \
     || xdg-open "$REPORT_PATH" 2>/dev/null \
     || start "" "$REPORT_PATH" 2>/dev/null \
     || echo "Отчёт сохранён: $REPORT_PATH (открой вручную)"
   ```

8. **Финальный ответ пользователю**:
   ```
   ✅ Готово. Отчёт открыт в браузере.
   Файл: reports/<slug>-<timestamp>.html
   Использовано: <stack>
   ```

### Stage 5b: Предложить Шаг 2 в чате

**Сразу после Stage 5** (после генерации и открытия HTML) — в чате Claude Code предложи запуск Stage 6.

В `{{STEP2_SECTION}}` HTML вставь STUB-state:

```html
<section class="step2-stub">
  <h2>📍 Шаг 2: Market context — не запущен</h2>
  <p>Добавь контекст сектора и конкурентов — скажи в чате Claude Code <code>«запусти Шаг 2»</code></p>
</section>
```

В чате после summary дай предложение:

```
🔬 Шаг 2 — Market context

Хочешь добавить понимание сектора, конкурентов и динамики рынка?
Это займёт ещё ~3-5 минут — тот же HTML обновится с market research.

Скажи «да» / «запусти Шаг 2» — пойду собирать market context.
```

Ожидай реакции пользователя. Если «да» / «запусти Шаг 2» / эквивалент → переходи к Stage 6.

Если «нет» / «пока хватит» / молчание → завершай. Пользователь может позже сам запустить.

---

### Stage 6: Market Research (только если пользователь сказал «да» в Stage 5b)

**Цель:** через WebSearch + WebFetch собрать market context. Не уходи в claude.ai — работаешь теми же тулзами что и в Stages 1-4.

Параллельные WebSearch'и:

1. **Конкуренты компании в секторе:**
   - `"{сектор}" Казахстан конкуренты топ-5`
   - `"{сектор}" Казахстан крупнейшие компании`
   - `"{сектор}" {company} конкуренты`
   - Из ответов извлеки 3-5 имён с краткой позицией (~10-15 слов про каждого)
   - Сгруппируй: KZ-локальные / региональные (РФ/CIS) / глобальные

2. **Динамика рынка 2024-2026:**
   - `"{сектор}" Казахстан рынок 2026`
   - `"{сектор}" Казахстан объём рост`
   - `"{сектор}" регулирование Казахстан 2026`
   - Ищи: объём рынка, темпы роста, регуляторные изменения, мандатное внедрение, ключевые milestones

3. **AI / технологические тренды в категории:**
   - `"{сектор}" AI внедрение тренды 2026`
   - `"{сектор}" автоматизация AI Казахстан`
   - Какие именно use cases AI в этой категории, кто из конкурентов внедряет, что мейнстрим

4. **Что меняется в 2026 (макро):**
   - Год AI в РК → как это влияет на сектор
   - Регуляторные изменения
   - Cross-border / regional shifts

5. **Уязвимости лидера сегмента:**
   - Угрозы со стороны глобалов / новых entrants / regulator
   - Где компания МОЖЕТ потерять долю
   - На что AI-native новички могут заходить

**Hallucination guard:** те же правила что Stage 1 — каждый факт с источником, неточные пометить.

---

### Stage 7: Update HTML — Step 2 + regenerate hooks (v1 → v2)

**Двойная задача:**
1. Заменить STUB Step 2 section на FULL market context
2. **Regenerate person-card hooks** с учётом market context → новая версия v2

После Stage 6:

#### 7a. Regenerate hooks (Шаг 2 evolution)

Для каждого PRIMARY ЛПР:
- Возьми CURRENT hook (v1) — он стал PREVIOUS
- Сгенерируй НОВЫЙ hook (v2) с учётом market context (AЗРK pressure, competitor moves, regulatory tailwinds и т.д.)
- v2 должен быть **резко сильнее v1** — если не сильнее, не меняй (нет смысла в evolution)

**Пример Documentolog:**
- v1 (Step 1): «Контур + partnership lead hiring — formation moment коммерческой функции»
- v2 (Step 2 + market): «Вы под AЗРK давлением + scaling cross-border + sales-стек — третий критичный вектор. Защита доли требует все три»

v2 ВКЛЮЧАЕТ контекст v1 + слой market awareness. Защита тезиса через market intelligence.

#### 7b. Update HTML

1. **Найди STUB Step 2 section** → замени на FULL market context (как раньше)

2. **Найди person-card `С чего зайти` block** → перестрой как hook-versions:

```html
<div class="person-block">
  <div class="block-label">С чего зайти</div>
  <div class="hook-versions">

    <div class="hook-version current">
      <div class="hook-version-meta">
        <span class="hook-version-badge">v2 · + Market</span>
        <span class="hook-version-status">Current — после Шага 2</span>
      </div>
      <div class="hook-version-body">{{HOOK_V2_BODY}}</div>
    </div>

    <div class="hook-version previous">
      <div class="hook-version-meta">
        <span class="hook-version-badge">v1 · Free baseline</span>
        <span class="hook-version-status">Предыдущая версия</span>
      </div>
      <div class="hook-version-body">{{HOOK_V1_BODY}}</div>
    </div>

    <div class="hook-version stub">
      <div class="hook-version-meta">
        <span class="hook-version-badge">v3 · + Paid</span>
        <span class="hook-version-status">Шаг 3 не запущен</span>
      </div>
      <div class="hook-version-body">
        Требует <code>$APIFY_API_KEY</code> в .env. Apify LinkedIn deep dive по ЛПР → личные посты, активность, network signals. Хук станет ещё точнее.
      </div>
    </div>

  </div>
</div>
```

3. **Также regenerate message draft** в dossier с учётом v2 hook (показывает evolved version в чате/email).

4. Сохрани файл, открой:
   ```bash
   open "$REPORT_PATH"
   ```

5. В чате: «✅ Шаг 2 добавлен. Обнови вкладку (Cmd+R). Заметь: hook эволюционировал — v2 теперь учитывает регуляторный контекст и конкурентов.»

---

### Stage 8: Шаг 3 — Paid sources (опционально, если есть keys)

**Сразу после Stage 7** (после Шаг 2 готов) — снова предложение в чате:

```
🔬 Шаг 3 — Paid sources (LinkedIn deep dive + goszakup)

Хочешь добавить личные данные ЛПР через paid sources?
- Apify LinkedIn Profile + последние 10 постов
- (опционально) goszakup history если есть token

Это ещё ~2-3 мин, hook эволюционирует в v3 с laser-точным контекстом.

Скажи «да» / «запусти Шаг 3».
```

**Если пользователь сказал «да»:**

1. Проверь `$APIFY_API_KEY` в env:
   ```bash
   if [ -f .env ]; then source .env; fi
   if [ -z "$APIFY_API_KEY" ]; then
     echo "Нет APIFY_API_KEY. Положи в .env."; exit
   fi
   ```

2. Найди LinkedIn URL primary ЛПР (из Stage 3 — если был snippet, или WebSearch заново)

3. Вызови Apify LinkedIn Profile Scraper:
   ```bash
   curl -X POST "https://api.apify.com/v2/acts/dev_fusion~linkedin-profile-scraper/run-sync-get-dataset-items?token=$APIFY_API_KEY" \
     -H "Content-Type: application/json" \
     -d '{"profileUrls": ["https://www.linkedin.com/in/<lpr-handle>"]}'
   ```

   Альтернативный actor если первый не работает: `bebity/linkedin-premium-actor`, `curious_coder/linkedin-profile-scraper`.

4. Из ответа извлеки:
   - Recent posts (последние 10)
   - About / Summary
   - Skills и эндорсменты
   - Network connections (high-profile)
   - Recent activity (likes, comments, reposts)

5. **Regenerate hook v3** — на основе личных постов + market context + everything:
   - v3 должен включать **конкретный recent LinkedIn signal** (цитата из поста, недавняя активность)
   - «Видел ваш пост от [дата] про [тема] — пересекается с тем что вы делаете в [продукт] + текущий контекст рынка»

6. Update HTML — Stage 9.

---

### Stage 9: Update HTML — Step 3 → v3 current

1. **Найди person-card hook-versions**:
   - v2 (current) → переезжает в .previous
   - v3 (новый) → становится .current на самом верху
   - v1 (previous) → остаётся .previous, но ниже v2
   - Stub v3 → удаляется

2. **Также regenerate message** в dossier с v3 хуком.

3. **Опционально:** добавить новую секцию `step3-personal-deep-dive` с raw LinkedIn data (свёрнуто по умолчанию через `<details>`).

4. Open + chat:
   ```
   ✅ Шаг 3 готов. Hook теперь v3 — с personal LinkedIn context.
   Обнови вкладку (Cmd+R).
   ```

---

### Market context HTML template (для замены STUB в Stage 7)

```html
<section class="step2-full">
  <h2>📍 Шаг 2: Market context</h2>
  <div class="summary-subtitle">Сектор, конкуренты, динамика, что из этого торчит</div>

  <dl class="market-grid">
    <dt>Сектор</dt>
    <dd>...определение сегмента (1 фраза)...</dd>

    <dt>Положение компании</dt>
    <dd>...лидер / challenger / niche, размер доли если известно...</dd>

    <dt>Конкуренты</dt>
    <dd>
      <ul>
        <li><strong>Имя</strong> — позиционирование (KZ-локальный)</li>
        <li><strong>Имя</strong> — позиционирование (региональный)</li>
        <li><strong>Имя</strong> — позиционирование (глобал)</li>
      </ul>
    </dd>

    <dt>Динамика</dt>
    <dd>
      <ul>
        <li class="up">что растёт / drives the market</li>
        <li class="up">регуляторные tailwinds</li>
        <li class="down">что давит / угрозы / headwinds</li>
      </ul>
    </dd>
  </dl>

  <div class="market-insight">
    <div class="market-insight-label">Что из этого торчит для нашего outreach</div>
    <div class="market-insight-body">
      1-2 предложения вывод. Как контекст сектора усиливает наш заход к этому ЛПР.
    </div>
  </div>

  <div class="market-sources">
    Источники: <a href="...">name1</a>, <a href="...">name2</a>, <a href="...">name3</a>
  </div>
</section>
```

## Output Format (для пользователя в чате)

**После Stage 5 (Шаг 1 готов):**

```
✅ Контекст-360 по <Company> готов. Отчёт открыт в браузере.

📄 reports/<slug>-<timestamp>.html

Найдено ЛПР: <N> (из них primary: <N>)
Сгенерировано сообщений: <N>
Использовано: <stack>
Время: ~<X> сек

—————————————————————————
🔬 Шаг 2 — Market context

Хочешь добавить понимание сектора, конкурентов и динамики рынка?
~3-5 мин. Тот же HTML обновится с market research.

Скажи «да» / «запусти Шаг 2» — пойду собирать.
```

Ожидай реакции пользователя.

**После Stage 7 (если Шаг 2 был запущен):**

```
✅ Шаг 2 добавлен в отчёт.

Использовано: <stack> + market research
Время Шага 2: ~<X> мин
Источников добавлено: <N>

Обнови вкладку браузера (Cmd+R) — Market context появится после Источников.
```

## Hallucination Guards (КРИТИЧНО)

- **Каждый факт** в output должен иметь URL-источник.
- Если факт не подтверждён → пометь `[не подтверждено]` или опусти.
- Если человек / компания не верифицируется → не выдумывай: «не нашёл публичных данных о X».
- **НИКОГДА не выдумывай:** имена ЛПР, должности, факты биографии, финансовые данные, конкретные события, цитаты из интервью.
- Если goszakup данные не доступны → не выдумывай историю контрактов.
- Если LinkedIn профиль не найден → не выдумывай содержание постов.

## Tool Detection (auto-graceful degradation)

В начале работы проверь доступные env vars:
- `$APIFY_API_KEY` → enables Apify LinkedIn calls (Stage 2-3)
- `$GOSZAKUP_TOKEN` → enables goszakup API (Stage 1)
- `$SCRAPEDO_TOKEN` → enables Scrape.do для заблокированных сайтов (Stage 1)
- `$YANDEX_SEARCH_API_KEY` → enables yandex-search-api для глубокого RU/KZ SERP

В output финальная строка перечисляет что использовалось: `Использовано: free стек` или `Использовано: free + apify + goszakup`.

## Failure Modes

- **Компания не найдена в adata.kz**: попробуй разные варианты написания (русский / казахский / English). Fallback: только WebSearch.
- **0 ЛПР найдено**: report и стоп. Не генери generic сообщения.
- **ЛПР без public footprint**: сообщение можно сделать на основе только company context, но честно отметить «контекст по человеку ограничен».
- **Все paid недоступны**: продолжай на free стеке, в output пометь.

## Расчётное время выполнения

- Free стек: 60-120 сек на компанию
- + Apify: +30-60 сек
- + goszakup: +20 сек

## Примеры запроса от пользователя

✅ «Сделай контекст-360 на Globalink Logistics. Мы продаём AI-инструменты для отдела продаж: лидген, скоринг входящих, аналитика звонков.»

✅ «Обогати БИН 110842011929 и подготовь outreach. Наш оффер: SaaS для логистических компаний — управление документооборотом и отслеживание грузов.»

✅ «Найди ЛПР в Documentolog и напиши первое сообщение. Продаём: коучинг отделов продаж через AI-анализ переписки.»

## Что НЕ делает этот skill

- Не отправляет письма (только генерирует драфты)
- Не сохраняет в CRM (только производит markdown)
- Не работает с компаниями вне Казахстана как primary use case (для них adata.kz и goszakup не релевантны — нужен другой набор источников)
- Не делает audio/voice анализ (это отдельная Surfnote-история)
