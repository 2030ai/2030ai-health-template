# 2030ai-health-template

> **TL;DR (EN).** An Obsidian-compatible vault + git template for a **personal health knowledge base**. Markdown with YAML frontmatter, 16 data categories, reusable templates, Dataview dashboards, and an AI-agent skill that pulls data from Gmail, Telegram, and local inbox. **Use "Use this template"** on GitHub, or paste the agent prompt below into Claude Code / Cursor / ChatGPT and let your agent scaffold it for you. Free, MIT-licensed, data stays on your machine.

Шаблон персональной базы знаний о здоровье. Структура, спецификация данных, пайплайн сбора и промпт для ИИ-агента — всё, что нужно, чтобы за час поднять у себя рабочий vault и начать накапливать медицинские данные в одном месте.

Шаблон выведен из реального проекта, который уже больше года используется автором (150+ записей, 7 внешних источников, импорты из OneNote за 2011–2019, экспорты Apple Health, анализы 8 лабораторий). Сюда вынесено только переиспользуемое ядро — без персональных медицинских данных.

---

## Что внутри и для кого

**Для кого:** человек, который хочет собрать в одно место разбросанные по Gmail / Telegram / лабораторным PDF / устройствам данные о своём здоровье и получить возможность анализировать их через ИИ — без SaaS, без сервера, без чужого облака.

**Что получаете сразу:**

- Дерево каталогов на **16 типов данных** (анализы, витальные, лекарства, визиты, прививки, сон, активность, симптомы, ментальное, стоматология, офтальмология, семейный анамнез, импорт устройств, питание, имиджинг, прочее).
- **Спецификацию YAML frontmatter** — обязательные поля, провенанс, именование, wiki-links.
- **Шаблоны** для каждого типа записи (Obsidian Templates plugin).
- **Dataview-дашборды** — последние записи, активные лекарства, предстоящие визиты, непроверенные результаты.
- **Скилл `health-update`** для Claude Code — пайплайн автоматического сбора из Gmail + Telegram + `inbox/`.
- **Каталог источников данных** ([`agent_docs/guides/data-sources.md`](agent_docs/guides/data-sources.md)) — проверенные на практике и ещё не подключённые, с инструкциями извлечения.
- **Документация для ИИ-агентов**: [AGENTS.md](AGENTS.md), [CLAUDE.md](CLAUDE.md), `agent_docs/` — чтобы любой агент понял контракт и начал работать без ручной настройки.

**Рабочий процесс** (one-liner): дропаете PDF / фото / экспорт в `inbox/` → зовёте агента (`/health-update` или промпт ниже) → получаете структурированную запись в `data/<type>/YYYY-MM-DD-описание.md`, оригинал уезжает в локальное/iCloud-хранилище вне git.

---

## Быстрый старт

### Вариант A. "Use this template" на GitHub (рекомендуется)

1. Нажмите **Use this template** → **Create a new repository**. Сделайте его **приватным** (данные о здоровье — чувствительные).
2. Склонируйте репозиторий себе:
   ```bash
   git clone git@github.com:<your-user>/<your-repo>.git my-health
   cd my-health
   ```
3. Откройте папку как Obsidian vault (File → Open folder as vault). Включите плагины **Dataview** и **Templates** (Core).
4. Создайте локальную папку для оригиналов (PDF, фото, экспорты) вне репозитория. По умолчанию — `~/Documents/health-originals/`. Этот путь прописывается в [AGENTS.md](AGENTS.md) вместо плейсхолдера `<ORIGINALS_DIR>`.
5. Удалите `EXAMPLE-*.md` из `data/*/`, когда добавите первые настоящие записи.
6. Опционально — настройте remote на шаблон, чтобы подтягивать улучшения:
   ```bash
   git remote add template https://github.com/2030ai/2030ai-health-template.git
   git fetch template
   ```

### Вариант B. Скопировать в существующий vault

`rsync -av --exclude='.git' --exclude='README.md' 2030ai-health-template/ ~/my-existing-vault/` — перенесите структуру и гайды, не затирая README.

### Вариант C. Поручить ИИ-агенту

Скопируйте промпт ниже в новый чат Claude Code / Cursor / ChatGPT — агент сам создаст файлы и задаст пару уточняющих вопросов. Это быстрее, чем B, если вам хочется диалога вместо копипасты.

---

## 🧠 Промпт для ИИ-агента

Скопируйте **весь блок ниже** в новый чат с ИИ-агентом (Claude Code, Cursor, ChatGPT с доступом к файлам, любой другой). Агент получит всё, что нужно: референс на шаблон, контракт данных, алгоритм установки, список источников, которые стоит у вас запросить.

````markdown
Ты помогаешь пользователю поднять персональную базу знаний о здоровье —
Obsidian vault + git. Референс-шаблон: https://github.com/2030ai/2030ai-health-template

**Цель.** Создать в текущей директории (или в указанной пользователем) vault
со структурой из шаблона, настроить политику хранения оригиналов, подключить
доступные источники данных и сделать первую запись.

**Ключевые свойства базы, которые нужно сохранить:**

1. Obsidian-совместимый markdown-vault под git. Один файл = одно медицинское
   событие. Никаких баз данных, JSON-стор, SaaS.
2. Каждая запись — markdown с YAML frontmatter. Обязательные поля:
   `type`, `date` (YYYY-MM-DD), `tags` (массив), `source`, `status`
   (`raw` | `verified` | `reviewed`).
3. Поля провенанса (опциональные, опускаются при `source: manual`):
   `origin_url`, `origin_type`, `origin_file`, `origin_date`.
4. Именование файлов: `YYYY-MM-DD-<короткое-описание>.md`. Связи между
   записями — Obsidian wiki-links `[[полное-имя-файла]]`.
5. **Оригиналы файлов (PDF, фото, DICOM, экспорты) НЕ коммитятся в git.**
   Они хранятся в отдельной папке (по умолчанию `~/Documents/health-originals/`,
   в идеале с iCloud/Dropbox-синхронизацией). В markdown на оригинал
   ссылается поле `origin_file`.
6. Анализ данных делает ИИ-агент по запросу. Скриптов, серверов,
   автоматических расчётов не нужно — всё через диалог.
7. Дисклеймер: любой анализ — информационный, не медицинский совет.

**Структура vault, которую нужно создать:**

```
<vault>/
├── README.md                # короткое описание, что это
├── AGENTS.md                # правила для ИИ-агента (см. шаблон)
├── CLAUDE.md                # инструкции под Claude Code (см. шаблон)
├── .gitignore               # игнорит temp/, .obsidian/workspace, .env
├── data/                    # 16 поддиректорий:
│   blood-tests/ vitals/ activity/ sleep/ nutrition/ symptoms/
│   mental/ medications/ doctor-visits/ dental/ vision/ vaccinations/
│   family-history/ devices/ imaging/ other/
├── templates/               # заготовки frontmatter для каждого типа
├── dashboards/              # Dataview-обзоры (overview, blood-tests,
│                            # medications, symptoms)
├── agent_docs/
│   ├── index.md             # навигация
│   ├── architecture.md
│   ├── adr.md               # решения, пусто или 2 базовых ADR
│   ├── development-history.md
│   └── guides/
│       ├── data-formats.md
│       ├── data-sources.md  # каталог источников
│       ├── health-update-workflow.md
│       ├── dod.md
│       └── archiving-and-temp.md
└── inbox/                   # сырые файлы для обработки
```

Возьми содержимое каждого файла из шаблона:
https://github.com/2030ai/2030ai-health-template

**Шаги установки, которые ты должен выполнить:**

1. **Спроси пользователя о среде:**
   - где разместить vault (путь)?
   - где хранить оригиналы (путь, вне git)? Подставь значение в AGENTS.md
     вместо плейсхолдера `<ORIGINALS_DIR>`.
   - git init и частный репозиторий — нужно ли сразу?
   - язык заметок (русский / английский / другой)?

2. **Создай структуру и файлы** из шаблона. Замени плейсхолдеры
   (`<ORIGINALS_DIR>`, `<YOUR_NAME>`, `<YOUR_TG_CHAT>`, `<YOUR_GMAIL_QUERY>`)
   на пользовательские значения. Пустые каталоги в `data/` помечай `.gitkeep`
   или одним EXAMPLE-файлом с `status: example`.

3. **Подключи источники данных** — предложи пользователю пройтись по
   каталогу `agent_docs/guides/data-sources.md` и выбрать, что импортировать
   первым. Стандартное меню:
   - **Лаборатории** (Инвитро, КДЛ, Гемотест, Хеликс, ЦитиЛаб, ДНКОМ,
     Labcorp, Quest, локальные лаборатории пользователя) — PDF из email
     или из личного кабинета.
   - **Gmail** — письма от лабораторий и клиник (поиск по ключевым словам).
     MCP: `google-mcp-readonly` (если есть).
   - **Telegram** — тематический чат «Здоровье / Мед / Я» с заметками.
     MCP: `telegram-read` (если есть).
   - **Apple Health / Google Fit / Fitbit / Garmin / Oura / Whoop / Withings** —
     экспорты JSON/CSV через приложение или Shortcuts.
   - **Генетика** — 23andMe / MyHeritage / Atlas / Invitae — SNP-экспорт.
   - **Медкарта / ЭМК** — Госуслуги / MyChart / Epic / Helsi / ЕМИАС / страховая.
   - **Имиджинг** — КТ/МРТ/УЗИ: заключения текстом в git, DICOM — в
     `<ORIGINALS_DIR>/imaging/`.
   - **Историческая миграция** — OneNote / Apple Notes / Notion / DayOne /
     Google Keep / Evernote — поиск по тегу "здоровье".

4. **Сделай первую запись** — предложи внести "нулевую" запись
   (например, свежий анализ крови или текущие лекарства). Убедись, что
   frontmatter корректен, файл в правильной подпапке, wiki-links
   оформлены. Оригинал, если есть — копируется в `<ORIGINALS_DIR>/`.

5. **Проверь инсталляцию:**
   - `git status` должен показывать чистую структуру без оригиналов.
   - Откройте vault в Obsidian: дашборды работают, Dataview не падает.
   - Обновите `agent_docs/development-history.md` записью вида:
     `[YYYY-MM-DD] Vault scaffolded from 2030ai-health-template`.

**Что НЕ делай:**

- Не изобретай новые поля frontmatter — используй только из спецификации.
- Не пиши коммерческих / диагностических выводов. Ты не врач.
- Не коммить PDF/фото/DICOM в git. Всегда — в `<ORIGINALS_DIR>`.
- Не удаляй оригиналы из `inbox/` до подтверждения, что они сохранены вовне.
- Не собирай и не отправляй данные пользователя никуда наружу.

**Когда закончишь — покажи:**
1. Дерево созданного vault (`tree` или аналог).
2. Пример первой записи (пустая, но с корректным frontmatter).
3. Подсвеченный список источников, которые пользователь выбрал, с планом,
   как их импортировать дальше.
````

---

## Структура vault

```
<vault>/
├── README.md               # краткое описание репо пользователя (перепишите)
├── AGENTS.md               # универсальные правила для ИИ-агента
├── CLAUDE.md               # инструкции под Claude Code
├── LICENSE                 # MIT (наследуется от шаблона)
├── .gitignore
├── data/                   # основные данные, 16 категорий
├── templates/              # заготовки frontmatter
├── dashboards/             # Obsidian Dataview-обзоры
├── agent_docs/             # документация для ИИ
│   ├── index.md
│   ├── architecture.md
│   ├── adr.md
│   ├── development-history.md
│   └── guides/
│       ├── data-formats.md
│       ├── data-sources.md          # каталог источников
│       ├── health-update-workflow.md
│       ├── dod.md
│       └── archiving-and-temp.md
├── inbox/                  # сырые файлы, ждут обработки
├── .agents/skills/health-update/SKILL.md   # canonical project-local skill
├── .claude/skills/health-update            # Claude Code mirror
├── .codex/skills/health-update             # Codex mirror
└── .cursor/skills/health-update            # Cursor mirror
```

---

## YAML frontmatter — спецификация (краткая)

**Обязательные** поля каждой записи в `data/`:

| Поле | Значения | Пример |
|---|---|---|
| `type` | одно из 14 значений (см. ниже) | `blood-test` |
| `date` | YYYY-MM-DD | `2025-04-20` |
| `tags` | массив строк | `[CBC, fasting]` |
| `source` | `manual` \| `inbox` \| `gmail` \| `telegram` \| `apple-health` \| `fitbit` \| `google-drive` и т.п. | `gmail` |
| `status` | `raw` \| `verified` \| `reviewed` | `raw` |

**Значения `type`:** `blood-test`, `vital`, `sleep`, `activity`, `medication`, `doctor-visit`, `symptom`, `mental`, `dental`, `vision`, `vaccination`, `family-history`, `nutrition`, `device-import`, `imaging`, `other`.

**Поля провенанса** (опциональные, опускайте при `source: manual`): `origin_url`, `origin_type`, `origin_file`, `origin_date`.

Полная спецификация — в [agent_docs/guides/data-formats.md](agent_docs/guides/data-formats.md).

Пример минимальной записи:

```yaml
---
type: blood-test
date: 2025-04-20
tags: [CBC, fasting]
source: gmail
status: raw
origin_type: email
origin_file: blood-tests/2025-04-20-cbc-lab-xyz.pdf
origin_date: 2025-04-21
---

## Результаты
| Показатель | Значение | Норма |
|---|---|---|
| ... |
```

---

## Источники данных

Краткая сводка (полный каталог — [agent_docs/guides/data-sources.md](agent_docs/guides/data-sources.md)):

| Категория | Что даёт | Как получить |
|---|---|---|
| **Лаборатории** (Инвитро, КДЛ, Гемотест, Хеликс, Labcorp, Quest…) | Анализы крови, мочи, гормоны, инфекции, генетика | PDF из личного кабинета или email |
| **Gmail** | Результаты лаб, счета клиник, напоминания о визитах | `google-mcp-readonly`, фильтр по отправителю |
| **Telegram** — тематический чат | Фото анализов, назначения, заметки | `telegram-read` MCP |
| **Apple Health** | HR, вес, давление, сон, активность | Shortcut → CSV → `inbox/` |
| **Fitbit / Garmin / Oura / Whoop / Withings** | Сон, HRV, тренировки, ЭКГ | API / web export |
| **Генетика** (23andMe, MyHeritage, Atlas) | SNP, предрасположенности, происхождение | Raw data export |
| **Имиджинг** (КТ/МРТ/УЗИ) | Заключения радиолога, DICOM | CD из клиники / портал |
| **ЭМК / Госуслуги / MyChart / Helsi** | Визиты, диагнозы, рецепты | Скачать PDF / screenshots |
| **Историческая миграция** (OneNote, Apple Notes, Notion, DayOne) | Заметки за прошлые годы | Поиск по тегу "здоровье" |
| **Семейный анамнез** | Болезни родственников | Google Form → вручную |
| **Аптечные чеки** | Что покупали, когда, за сколько | ФНС, СберСпасибо, Ozon Pharm |

Раздел **"Ещё можно добавить"** в каталоге источников содержит 20+ позиций: CGM (Dexcom, Libre, Levels), Eight Sleep, Cronometer, Strava, Clue/Flo, Headspace, Oral-B и т.д. Начинайте с 2–3 наиболее доступных.

---

## Privacy by design

- **В git:** только markdown с метаданными (frontmatter + заметки).
- **Не в git:** PDF, фото, DICOM, raw CSV-экспорты устройств — они живут в `<ORIGINALS_DIR>` (рекомендуется iCloud / локальный диск с бэкапом).
- **Репозиторий — приватный.** Публиковать свой личный health-vault не нужно; публичен только шаблон.
- **ИИ-агент** работает локально (через Claude Code / Cursor) или через API, которому вы доверяете. Анализ не отправляется в третьи стороны автоматически.
- **Оригиналы обезличиваются** только при вашем явном согласии. По умолчанию — не обезличиваются (вы храните их на своей технике).

---

## Обратная связь и синхронизация

- **Улучшения шаблона** (новые типы записей, гайды, источники, промпты) — через PR в этот репо. Полезные находки в вашем форке легко возвращать через PR `template/main` ← `your-fork/feature`.
- **Обновление вашей копии из шаблона:**
  ```bash
  git remote add template https://github.com/2030ai/2030ai-health-template.git
  git fetch template
  git cherry-pick <commit>     # выбирайте нужные апдейты точечно
  ```
  **Не сливайте** весь `template/main` целиком — это перепишет ваши заметки.

---

## Лицензия и дисклеймер

**Лицензия:** MIT. См. [LICENSE](LICENSE).

**Медицинский дисклеймер.** Этот шаблон и любой анализ, который делает на его основе ИИ-агент, — **информационный**, не заменяет консультацию врача. Результаты лаборатории, назначения, диагнозы обсуждайте со своим доктором. Авторы шаблона не несут ответственности за решения, принятые на его основе.

---

Референс-проект (для понимания "как выглядит живая база после года использования"): [2030ai/2030ai-health-template](https://github.com/2030ai/2030ai-health-template) собран из приватного vault-а автора; в публичный шаблон вынесено только переиспользуемое ядро, без единой реальной медицинской записи.
