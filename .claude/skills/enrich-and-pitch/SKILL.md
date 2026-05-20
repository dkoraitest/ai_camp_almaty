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

End-to-end пайплайн для персонализированного outbound: от названия компании до 1-3 готовых opening сообщений под конкретных ЛПР с публично подтверждаемым контекстом.

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

4. **WebSearch hh.kz**:
   - `site:hh.kz "<company>" вакансии` → найди employer URL вида `hh.kz/employer/{id}`
   - **WebFetch** employer-страницу → количество открытых вакансий + сфера найма
   - Hiring-сигналы = приоритеты компании

5. **WebSearch news**:
   - `"<company>" 2026 OR 2025 новости`
   - `"<company>" интервью OR пресс-релиз`
   - `"<company>" "назначен" OR "возглавил" OR "новый руководитель"`

6. **(Опционально, paid)** Если `$GOSZAKUP_TOKEN` доступен:
   - REST: `GET https://ows.goszakup.gov.kz/v3/subject/biin/{BIN}` + Bearer header → детали + сотрудники
   - REST: `GET https://ows.goszakup.gov.kz/v3/contract?supplier_biin={BIN}` → выигранные госконтракты
   - REST: `GET https://ows.goszakup.gov.kz/v3/rnu/{BIN}` → проверка по реестру недобросовестных

**Output Stage 1:** структурированный профиль компании, каждый факт привязан к источнику (URL).

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

### Stage 4: Synthesis & Generation

**Цель:** 1-3 готовых opening messages.

Для **каждого** идентифицированного ЛПР:

1. **Найти angle (hook)**: пересечение их recent context × наш оффер.
   - Примеры:
     - «Только что наняли Head of Sales → растят коммерческую функцию → AI sales tools релевантны»
     - «Спикер на конференции про Eurasian corridor → рост грузопотока → наше решение усиливает»
     - «Награда Altyn Sapa за 2024 → компания зрелая, ищет efficiency, не survival → подходит формат экспертных кейсов»

2. **Составить opening message** по структуре:
   - **1 предложение**: конкретный personal context (что-то конкретное из их публичной активности — цитата / факт / событие)
   - **1 предложение**: связь с нашим оффером — почему мы пишем именно сейчас
   - **1 предложение**: специфичная value prop, релевантная их контексту (НЕ generic «у нас классный продукт»)
   - **1 предложение**: low-friction next step (15 мин call / ссылка на case / ответ on email)

3. **Правила стиля сообщения:**
   - Русский язык (если ЛПР не коммуницирует публично только на английском)
   - Прямой тон, без лести, без buzzwords
   - 3-5 предложений максимум
   - Specific, не generic: если можно заменить название компании и смысл не сломается — слишком общее
   - Источник упомянут (чтобы ЛПР видел: мы реально читали что они говорят)
   - НЕТ слов: «надеюсь, у вас все хорошо», «коротко о нас», «уделите 5 минут», «думаю, вам будет интересно»

### Stage 5: HTML Report Generation & Auto-Open

**Цель:** красивый, копируемый HTML-отчёт, автоматически открывается в браузере.

1. **Подготовь данные** для подстановки в шаблон (все собранные на Stage 1-4 факты + сообщения).

2. **Прочитай шаблон**:
   ```
   Read: .claude/skills/enrich-and-pitch/templates/report.html
   ```
   В конце шаблона есть HTML-комментарий с описанием всех placeholder'ов и структур повторяющихся блоков (LPR_CARDS, DOSSIERS, SOURCES) — следуй им строго.

3. **Сгенерируй заполненный HTML**:
   - Подставь `{{COMPANY_NAME}}`, `{{BIN}}`, `{{TIMESTAMP}}` (формат «20 мая 2026, 14:30»), `{{STACK_TAG}}`, `{{STACK}}`, `{{RUNTIME}}`
   - `{{COMPANY_FACTS}}` — заполни `<dt>/<dd>` парами из Stage 1 (каждый факт с источником-ссылкой)
   - `{{LPR_CARDS}}` — карточки из Stage 2 (1-3 штуки)
   - `{{DOSSIERS}}` — полные блоки per-ЛПР из Stage 3+4 (1-3 штуки)
   - `{{SOURCES}}` — все использованные URL из всех стейджей, сгруппированы

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

## Output Format (для пользователя в чате)

После Stage 5 верни короткое summary в чат (детали — в HTML):

```
✅ Контекст-360 по <Company> готов. Отчёт открыт в браузере.

📄 reports/<slug>-<timestamp>.html

Найдено ЛПР: <N>
Сгенерировано сообщений: <N>
Использовано: <stack>
Время: ~<X> сек
```

Полное содержание (профиль, ЛПР, контекст, сообщения, источники) — в HTML-отчёте.

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
