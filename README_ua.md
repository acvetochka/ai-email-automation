# AI Email Assistant — n8n

AI Email Assistant — це автоматизований помічник для обробки вхідної електронної пошти IT-фрилансера. Система аналізує листи, визначає необхідність відповіді, допомагає організовувати робочі зустрічі, створює чернетки відповідей у Gmail та зберігає історію обробки в Google Sheets.

Workflow побудований в n8n із використанням AI-моделі GPT-4o-mini через OpenRouter та інтеграцій Gmail, Google Calendar, Google Sheets і Telegram.

---

# 1\. Історія версій
<!--
| Версія | Що змінилося |
| --- | --- |
| **1.0 — Initial Release** | Початкова версія AI Email Assistant. Додано отримання листів із Gmail, AI-класифікацію, визначення зустрічей, перевірку Google Calendar на конфлікти та створення чернеток відповідей. |
| **1.1** | Додано Telegram-сповіщення про створені події календаря та чернетки Gmail. Покращено системні промпти класифікації та генерації відповідей, обробку тем листів і багатомовну підтримку. |
| **1.2** | Уточнено правила визначення дат зустрічей. Додано чіткіші правила інтерпретації відносних дат на основі часу отримання вихідного листа, із використанням часового поясу Europe/Berlin. |
| **1.3** | Розширено класифікацію листів: додано категорію spam, уточнено професійну область компетенції асистента та обробку скарг. Додано інтеграцію Google Sheets для журналювання листів, чернеток і подій календаря. Розширено Telegram-сповіщення інформацією про текст чернетки. |
| **1.4** | Змінено послідовність виконання: `Append Email to Sheet` інтегровано в основний потік перед `Route Email`. Це забезпечує створення початкового запису до подальшого оновлення інформації про подію або чернетку. Перейменовано вузли Google Sheets. Додано сім Sticky Notes з документацією основних компонентів workflow. Видалено сценарій `meeting_creation_error` із промпта генерації чернеток. |
-->

| Версія | Автор | Зміни | Завантаження |
| --- | --- | --- | ------| 
| [v1.0](https://github.com/acvetochka/ai-email-automation/tree/v1.0) | [Alona Kuznietsova](https://github.com/acvetochka) | Початкова реалізація AI Email Assistant MVP | [завантажити v1.0](https://github.com/acvetochka/ai-email-automation/releases#release-v1.0) |
| [v1.1](https://github.com/acvetochka/ai-email-automation/tree/v1.1) | [Alona Kuznietsova](https://github.com/acvetochka) | Telegram-сповіщення, покращення класифікації та генерації відповідей | [завантажити v1.1](https://github.com/acvetochka/ai-email-automation/releases#release-v1.1) |
| [v1.2](https://github.com/acvetochka/ai-email-automation/tree/v1.2) | [Alona Kuznietsova](https://github.com/acvetochka) | Уточнення визначення дат і часу зустрічей |[завантажити v1.2](https://github.com/acvetochka/ai-email-automation/releases#release-v1.2) |
| [v1.3](https://github.com/acvetochka/ai-email-automation/tree/v1.3) | [Alona Kuznietsova](https://github.com/acvetochka) | Розширена класифікація, виявлення спаму, інтеграція Google Sheets | [завантажити v1.3](https://github.com/acvetochka/ai-email-automation/releases#release-v1.3) |
| [v1.4](https://github.com/acvetochka/ai-email-automation/tree/v1.4) | [Alona Kuznietsova](https://github.com/acvetochka) | Послідовне журналювання, перейменування вузлів, документація workflow | [завантажити v1.4](https://github.com/acvetochka/ai-email-automation/releases#release-v1.4) |

---

# 2\. Огляд: що робить сценарій

**Назва процесу:**  
AI Email Assistant — автоматизований помічник для електронної пошти.

**Опис бізнес-задачі:**  
Автоматизувати аналіз і попередню обробку вхідних робочих листів, підготовку відповідей та організацію зустрічей, зменшуючи час на ручне сортування електронної пошти.

**Що робить процес:**

- Отримує нові листи Gmail та аналізує їхній зміст за допомогою AI.
- Визначає категорію листа, необхідність відповіді та наявність пропозиції про зустріч.
- Перевіряє доступність часу в Google Calendar і створює події, якщо немає конфліктів.
- Генерує чернетки відповідей у Gmail, зберігає результати в Google Sheets та надсилає сповіщення в Telegram.

**Основні категорії листів:**

| Категорія | Опис |
| --- | --- |
| `question` | Робочі питання, запити, скарги та пропозиції |
| `meeting` | Організація або підтвердження зустрічі |
| `information` | Інформаційні повідомлення |
| `advertising` | Рекламні та маркетингові розсилки |
| `auto_reply` | Автоматичні відповіді |
| `spam` | Спам, фішинг і підозрілі повідомлення |
| `other` | Інші листи та запити поза професійною компетенцією |

**Важливі обмеження:**

- Асистент працює в контексті професійної діяльності IT-фрилансера: веброзробка, програмні рішення та автоматизація бізнес-процесів.
- Не надає юридичних, медичних, фінансових, психологічних та інших непрофільних консультацій.
- Створює лише чернетки листів у Gmail. Автоматичне надсилання електронної пошти не виконується.
- Подія календаря створюється лише за наявності необхідних даних про дату та час і за відсутності конфліктів.

---

# 3\. Логіка та архітектура

## 3.1. Загальна схема процесу

``` mermaid
flowchart TD
    A[Gmail Trigger] --> B[Prepare Email Data]
    B --> C[AI Classify Email Agent]
    C --> D[Determine Route]
    D --> E[Append Email to Sheet]
    E --> F[Route Email]

    F -->|Meeting + Reply| G[Check Date and Time]
    F -->|Meeting only| G
    F -->|Reply only| H[Draft — Reply]
    F -->|No Action| I[End]

    G -->|Date/time available| J[Prepare Event Data]
    G -->|Missing data| K[Draft — Missing Data]

    J --> L[Get many events]
    L --> M[Detect Calendar Conflicts]
    M --> N{Is Event Free?}

    N -->|Yes| O[Create an Event]
    N -->|No| P[Draft — Conflict]

    O --> Q[Update Calendar Event]
    O --> R[Telegram Event Notification]
    O --> S[Draft — Created]

    H --> T[Check Draft Required]
    K --> T
    P --> T
    S --> T

    T -->|Yes| U[AI Generate Email Draft]
    U --> V[Create Gmail Draft]
    U --> W[Update Gmail Draft]
    U --> X[Telegram Draft Notification]
```

**Принцип роботи:** кожен новий лист спочатку отримує класифікацію та записується в Google Sheets. Лише після успішного виконання вузла `Append Email to Sheet` запускається маршрутизація та подальша обробка.

## 3.2. Основні етапи

**1. Отримання та підготовка листа**

- `Gmail Trigger` — перевіряє Gmail на нові листи кожну хвилину.
- `Prepare Email Data` — формує структурований набір даних: ID листа, тему, текст, відправника, ім'я та дату отримання.

**2. AI-класифікація**

- `AI Classify Email Agent` — аналізує лист і визначає його категорію, необхідність відповіді та наявність зустрічі.
- `Structured Output Parser` — перевіряє структуру результату AI відповідно до заданої JSON-схеми.
- `Determine Route` — визначає маршрут обробки та відповідний аркуш Google Sheets.

**3. Початкове журналювання**

- `Append Email to Sheet` — створює початковий запис в Google Sheets.
- Зберігаються дані листа та результати класифікації.
- Після цього запускається `Route Email`.

**4. Обробка зустрічей**

- `isDateAndTimeExist` — перевіряє наявність дати й часу.
- `Prepare Event Data` — формує дату початку та завершення зустрічі, учасника й часовий пояс.
- `Get many events` — отримує події календаря в заданому часовому проміжку.
- `Detect Calendar Conflicts` — визначає наявність конфліктних подій.
- `isEventFree` — перевіряє результат.
- `Create an Event` — створює подію в Google Calendar, якщо час вільний.
- `Update Calendar Event` — записує інформацію про створену подію в Google Sheets.
- `Telegram - Event Created Notification` — надсилає сповіщення про створену подію.

Якщо дата або час відсутні, створюється контекст `Draft — Missing Data`. Якщо виявлено конфлікт, використовується `Draft — Conflict`.

**5. Генерація чернетки відповіді**

- `Draft — Reply`, `Draft — Missing Data`, `Draft — Conflict`, `Draft — Created` — формують контекст відповідного сценарію.
- `isDraftRequired` — визначає, чи потрібна чернетка.
- `AI Generate Email Draft` — генерує відповідь на основі вихідного листа та контексту обробки.
- `Create Gmail Draft` — створює чернетку в Gmail.
- `Structured Output Parser1` — перевіряє структуру результату генерації.
- `Update Gmail Draft` — зберігає текст відповіді в Google Sheets.
- `Telegram - Draft Created Notification` — надсилає повідомлення про створену чернетку, її тему, отримувача та текст.

**6. Завершення обробки**

Якщо відповідь не потрібна, workflow завершує обробку після початкового журналювання. Якщо створено подію або чернетку, відповідні результати зберігаються в Google Sheets та надсилаються через Telegram.

## 3.3. Опис ключових вузлів

| Назва вузла | Тип | Призначення | Вхідні дані | Вихідні дані |
| --- | --- | --- | --- | --- |
| Gmail Trigger | Gmail Trigger | Отримання нових листів | Gmail Inbox | Дані вхідного листа |
| Prepare Email Data | Set | Підготовка даних листа | Дані Gmail | Структуровані дані листа |
| AI Classify Email Agent | AI Agent | Класифікація листа | Тема, текст, дата | Тип листа, ознаки відповіді та зустрічі |
| Structured Output Parser | Output Parser | Перевірка структури класифікації | Результат AI | Валідований JSON |
| Determine Route | Set | Визначення маршруту та аркуша | Результат класифікації | `route`, `xls_list` |
| Append Email to Sheet | Google Sheets | Створення початкового запису | Дані листа та класифікація | Запис у відповідному аркуші |
| Route Email | Switch | Розподіл листів за сценаріями | `route` | Відповідна гілка workflow |
| Prepare Event Data | Set | Підготовка параметрів зустрічі | Дата, час, тривалість, відправник | Дані події |
| Get many events | Google Calendar | Отримання подій для перевірки | Часовий інтервал | Список подій |
| Detect Calendar Conflicts | Code | Виявлення конфліктів | Події календаря | Ознака доступності часу |
| Create an Event | Google Calendar | Створення зустрічі | Дата, час, назва, учасник | Дані створеної події |
| Update Calendar Event | Google Sheets | Оновлення запису про зустріч | Дані події та Email ID | Посилання й параметри події в таблиці |
| AI Generate Email Draft | AI Agent | Генерація відповіді | Вихідний лист і контекст сценарію | Статус, тема та текст відповіді |
| Create Gmail Draft | Gmail Tool | Створення чернетки | Отримувач, тема, текст | Чернетка Gmail |
| Update Gmail Draft | Google Sheets | Збереження тексту відповіді | Email ID та текст чернетки | Оновлений запис |
| Telegram Notifications | Telegram | Повідомлення про результати | Дані події або чернетки | Сповіщення в Telegram |

---

# 4\. Промпти та моделі

У workflow використовуються два AI-агенти. Обидва працюють із моделлю `openai/gpt-4o-mini` через OpenRouter.

## 4.1. AI Classify Email Agent

**Призначення:** аналіз вхідних листів, визначення категорії, необхідності відповіді та параметрів робочої зустрічі.

**Основні функції:**

- Визначення типу листа.
- Визначення необхідності відповіді.
- Виявлення пропозицій про зустріч.
- Витягування назви, дати, часу та тривалості зустрічі.
- Інтерпретація відносних дат на основі дати отримання листа.
- Дотримання професійної області компетенції IT-фрилансера.

**Модель:** `openai/gpt-4o-mini` через OpenRouter.

**Параметри:** додаткові параметри моделі в `options` не задані; використовуються налаштування за замовчуванням.

**Формат результату:** структурований JSON із полями:

- `email_type`
- `needs_reply`
- `is_meeting`
- `event_title`
- `event_date`
- `event_time`
- `event_duration_minutes`

**Системний промпт:** [AI Classify Email Agent — System Prompt](https://github.com/acvetochka/ai-email-automation/blob/main/AI-Agent1-prompt-v1.3.md)

## 4.2. AI Generate Email Draft

**Призначення:** створення професійних чернеток відповідей на вхідні робочі листи.

**Основні функції:**

- Генерація відповіді мовою вихідного листа.
- Дотримання професійного та ввічливого стилю.
- Обробка звичайних запитів.
- Уточнення відсутньої дати або часу зустрічі.
- Реакція на конфлікти в календарі.
- Підтвердження створеної зустрічі та відповіді на додаткові питання.
- Формування теми та тексту чернетки.

**Модель:** `openai/gpt-4o-mini` через OpenRouter.

**Параметри:** додаткові параметри моделі в `options` не задані; використовуються налаштування за замовчуванням.

**Інструмент:** `Create Gmail Draft` — створення чернетки в Gmail.

**Формат результату:** структурований JSON із полями:

- `status`
- `message`
- `subject`
- `text`

**Системний промпт:** [AI Generate Email Draft — System Prompt](https://github.com/acvetochka/ai-email-automation/blob/main/AI-Agent2-prompt-v1.3.md)

**Обґрунтування вибору моделі та параметрів:**

GPT-4o-mini використовується для обох AI-агентів через OpenRouter. Модель застосовується для класифікації текстових повідомлень, витягування структурованих даних та генерації відповідей. Єдина модель для обох завдань спрощує конфігурацію та обслуговування workflow.

---

# 5\. Залежності та налаштування

## 5.1. Зовнішні сервіси та credentials

| Сервіс | Credentials у n8n | Обліковий запис | Необхідні права |
| --- | --- | --- | --- |
| Gmail | `180926_Kuznietsova_Gmail` | Підключений Gmail-акаунт | Отримання листів, читання повідомлень, створення чернеток |
| Google Calendar | `180626_Kuznietsova_Google_Calendar` | Підключений Google-акаунт | Читання подій календаря, створення подій |
| Google Sheets | `180926_Kuznietsova_Google Sheets` | Підключений Google-акаунт | Додавання та оновлення рядків у таблиці |
| Telegram | `180926_Kuznietsova_Telegram` | Підключений Telegram-бот | Надсилання повідомлень у визначений чат |
| OpenRouter | `180926_Kuznietsova_OpenRouter` | Обліковий запис OpenRouter | Доступ до API та використання моделі GPT-4o-mini |

*Секретні значення credentials, API-ключі та токени в документації не зберігаються.*

## 5.2. Основні налаштування workflow

| Параметр | Значення |
| --- | --- |
| Платформа | n8n Community Edition |
| Часовий пояс workflow | `Europe/Berlin` |
| Gmail Trigger | Перевірка кожну хвилину |
| AI Provider | OpenRouter |
| AI Model | `openai/gpt-4o-mini` |
| Calendar | Google Calendar |
| Database / Logging | Google Sheets |
| Notifications | Telegram |
| Email processing | Gmail |
| Email sending | Не використовується — створюються лише чернетки |

## 5.3. Google Sheets

**Назва таблиці:** `AI_Email_Assistant_Email_Database`

Для розподілу листів використовуються такі аркуші:

| Тип листа | Аркуш |
| --- | --- |
| `question` | Main |
| `meeting` | Main |
| `information` | Info |
| `advertising` | Advertising |
| `auto_reply` | Info |
| `spam` | Spam |
| `other` | Other |

**Структура запису:**

| Поле | Призначення |
| --- | --- |
| `Email_id` | Унікальний ідентифікатор листа |
| `Date` | Дата отримання |
| `Name` | Ім'я відправника |
| `Email` | Адреса відправника |
| `Subject` | Тема листа |
| `Body` | Текст листа |
| `Type` | Категорія листа |
| `Needs reply` | Чи потрібна відповідь |
| `Is meeting` | Чи містить лист зустріч |
| `Reply` | Згенерований текст відповіді |
| `Event` | Посилання на подію Google Calendar |
| `Event title` | Назва зустрічі |
| `Event date` | Дата зустрічі |
| `Event time` | Час початку |
| `Event duration` | Тривалість у хвилинах |

**Логіка збереження:**

1. `Append Email to Sheet` створює початковий запис після класифікації листа.
2. `Update Calendar Event` оновлює відповідний запис після створення події.
3. `Update Gmail Draft` додає текст згенерованої відповіді до відповідного запису.

Для пошуку запису використовується `Email_id`.

## 5.4. Вимоги до налаштування

Перед запуском workflow необхідно:

1. Імпортувати JSON-файл workflow у n8n.
2. Підключити власні credentials Gmail, Google Calendar, Google Sheets, Telegram та OpenRouter.
3. Перевірити правильність вибраного календаря та Google Sheets.
4. Переконатися, що в Google Sheets створені всі необхідні аркуші та колонки.
5. Перевірити відповідність назв аркушів значенням у `Determine Route`.
6. Налаштувати Telegram-бота та цільовий чат.
7. Перевірити роботу обох AI-агентів і створення чернеток.
8. Виконати тестування всіх основних сценаріїв перед активацією workflow.

---

# 6\. Важливі особливості та обмеження

- Дата зустрічі визначається на основі дати отримання вихідного листа, а не дати запуску workflow.
- Для інтерпретації дат і часу використовується `Europe/Berlin`.
- Якщо дата або час зустрічі відсутні чи неоднозначні, AI не повинен вигадувати їх.
- Якщо в календарі виявлено конфлікт, подія не створюється, а готується чернетка з проханням узгодити інший час.
- Інформація про конфліктуючі події не розкривається відправнику.
- Початковий запис листа в Google Sheets створюється до запуску маршрутизації.
- Подальше оновлення запису виконується за `Email_id`.
- AI-агент не надсилає листи самостійно — усі відповіді залишаються чернетками Gmail.
- Telegram використовується лише для внутрішніх сповіщень про результати обробки.

---

# 7\. Структура workflow

У workflow використовуються такі групи вузлів:

| Група | Основні вузли |
| --- | --- |
| Отримання листів | Gmail Trigger, Prepare Email Data |
| AI-класифікація | AI Classify Email Agent, Structured Output Parser |
| Маршрутизація | Determine Route, Route Email |
| Журналювання | Append Email to Sheet, Update Calendar Event, Update Gmail Draft |
| Робота з календарем | isDateAndTimeExist, Prepare Event Data, Get many events, Detect Calendar Conflicts, isEventFree, Create an Event |
| Підготовка відповіді | Draft — Created, Draft — Conflict, Draft — Missing Data, Draft — Reply, isDraftRequired |
| AI-генерація | AI Generate Email Draft, Structured Output Parser1, Create Gmail Draft |
| Сповіщення | Telegram - Event Created Notification, Telegram - Draft Created Notification |

У workflow також додано Sticky Notes із поясненнями призначення та логіки основних груп вузлів.


---

**Проєкт:** AI Email Assistant  
**Платформа:** n8n Community Edition  
**AI Provider:** OpenRouter  
**AI Model:** GPT-4o-mini  
**Timezone:** Europe/Berlin  
**Поточна версія:** 1.4
