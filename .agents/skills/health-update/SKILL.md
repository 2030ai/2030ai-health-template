---
name: health-update
description: Use when the user asks to collect, update, import, or structure personal health data from inbox files, Gmail, or Telegram into the health vault; route Telegram reads only through the global Windows Telegram Gateway.
---

# /health-update

Единая точка входа для сбора health-данных из всех источников. Проверяет inbox, Gmail и Telegram, парсит найденное, создаёт структурированные записи в Obsidian vault.

**Vault:** текущая директория (корень репозитория, созданного из шаблона).
**Оригиналы:** `<ORIGINALS_DIR>` — путь пользователя (например, `~/Documents/health-originals/`). Путь фиксируется при скаффолдинге в `AGENTS.md`.

## Настройка под себя

Замените плейсхолдеры на свои значения (один раз, перед первым запуском):

- `<ORIGINALS_DIR>` — путь к папке оригиналов вне git.
- `<TG_CHAT>` — имя Telegram-чата с заметками о здоровье. Если Telegram не используется — пропустите шаг 3.
- `<GMAIL_QUERY>` — ключевые слова Gmail-поиска. Базовый набор:
  - `"lab OR results OR blood test OR clinic OR analysis"`
  - Плюс имена ваших локальных лабораторий (пример для RU: `"инвитро OR гемотест OR хеликс OR ситилаб"`; для US: `"labcorp OR quest OR kaiser"`).

## Перед началом

1. Прочитать `agent_docs/guides/data-formats.md` — спецификация frontmatter.
2. Прочитать `agent_docs/guides/health-update-workflow.md` — полный workflow.
3. Прочитать `agent_docs/guides/data-sources.md` — каталог источников.
4. Проверить последнюю запись в `agent_docs/development-history.md` — контекст.

## Алгоритм

### Шаг 1. Проверить `inbox/`

```
Glob inbox/* → для каждого файла:
  - Read файл (PDF, фото, текст)
  - Определить тип данных
  - Создать .md в data/<type>/
  - Переместить оригинал: cp → <ORIGINALS_DIR>/<type>/
  - Удалить из inbox/ после подтверждения копирования
```

### Шаг 2. Поиск в Gmail

Использовать `google-mcp-readonly: gmail_search` с `<GMAIL_QUERY>` за последние 30 дней (или с последнего watermark).

Для каждого релевантного письма:
- Прочитать содержимое: `gmail_get_message`.
- Если есть вложения — отметить для ручного скачивания (MCP не умеет attachment download).
- Создать `.md` в `data/<type>/` с `origin_url` = ссылка на письмо, `origin_type: email`.
- Пропускать письма, для которых уже есть запись (проверить по `origin_url` в существующих файлах).

### Шаг 3. Чтение Telegram

Если Telegram используется — обращаться только к глобальному
`$telegram-gateway`. Не подключать Telegram MCP, Desktop, Telethon или запасной
локальный кэш:

```
1. Проверить Gateway: status
2. Разрешить чат: resolve --kind dialog --name "<TG_CHAT>"
3. Прочитать bounded period: recent --dialog-ref <opaque-ref> --start ... --end ...
4. Для каждого нового сообщения:
   - Определить тип данных по содержимому
   - Создать .md в data/<type>/
   - origin_type: telegram
```

При `indexing_pending`, неоднозначном чате или недоступном Gateway остановить
только Telegram-шаг и показать безопасный статус; другой транспорт запрещён.

### Шаг 4. Отчёт

Вывести сводку:

```markdown
## Health Update — отчёт

### Новые записи
- [тип] описание → data/path/filename.md

### Требует внимания
- Отклонения от нормы
- Просроченные курсы лекарств
- Давно не обновлявшиеся категории

### Статистика
- Всего записей в vault: N
- Новых за этот запуск: N
- Категории с данными: [список]
- Пустые категории: [список]
```

## Формат создаваемых файлов

### Обязательные поля frontmatter

```yaml
---
type: <тип>
date: <YYYY-MM-DD>
tags: [<релевантные теги>]
source: <inbox | gmail | telegram | manual | apple-health | fitbit | ...>
status: raw
origin_url: <URL источника>     # опускать если manual
origin_type: <email | telegram | pdf | photo>
origin_file: <имя файла>        # если есть оригинал
origin_date: <YYYY-MM-DD>       # дата получения
---
```

### Именование

`YYYY-MM-DD-<описание>.md` — на языке пользователя, через дефис. Примеры:
- `2025-04-20-biochemistry.md`
- `2025-04-18-therapist-smith.md`
- `2025-04-22-headache.md`

### Wiki-links

Использовать `[[полное-имя-файла]]` для связей. Опережающие ссылки допустимы.

## Типы данных → папки

| Тип | Папка | Примеры контента |
|---|---|---|
| blood-test | data/blood-tests/ | Анализы крови, мочи, биохимия, гормоны |
| vital | data/vitals/ | Давление, пульс, вес, температура |
| sleep | data/sleep/ | Сон |
| activity | data/activity/ | Шаги, тренировки |
| nutrition | data/nutrition/ | Питание, диеты |
| medication | data/medications/ | Лекарства, БАДы, курсы |
| doctor-visit | data/doctor-visits/ | Визиты к врачам, заключения |
| symptom | data/symptoms/ | Симптомы, жалобы |
| mental | data/mental/ | Настроение, стресс, энергия |
| dental | data/dental/ | Стоматология |
| vision | data/vision/ | Зрение |
| vaccination | data/vaccinations/ | Прививки, скрининги |
| family-history | data/family-history/ | Семейный анамнез |
| imaging | data/imaging/ | КТ/МРТ/УЗИ заключения |
| device-import | data/devices/ | Сырые экспорты устройств |
| other | data/other/ | Генетика, биохакинг, прочее |

## Оригиналы файлов

Перемещать в `<ORIGINALS_DIR>/<тип>/`:

```bash
cp inbox/файл.pdf <ORIGINALS_DIR>/<тип>/
# Проверить, что файл скопировался
ls <ORIGINALS_DIR>/<тип>/файл.pdf
# Удалить из inbox
rm inbox/файл.pdf
```

## Дедупликация

Перед созданием записи проверить существующие файлы:

```
Grep origin_url в data/ → если найден URL, пропустить
Grep по дате + типу + ключевым словам → если похожая запись есть, пропустить
```

## Дисклеймер

При обнаружении отклонений от нормы в анализах — добавить в отчёт:

> ⚕️ Это информационный анализ, а не медицинский совет. Обсудите результаты с лечащим врачом.

## MCP-инструменты

| Инструмент | Использование | Обязателен? |
|---|---|---|
| `google-mcp-readonly: gmail_search` | Поиск писем от лабораторий | Нет |
| `google-mcp-readonly: gmail_get_message` | Чтение письма | Нет |
| `$telegram-gateway: status/resolve/recent` | Поиск и чтение Telegram | Нет |
| Glob, Read, Write | Работа с `inbox/` и `data/` | Да |
| Bash (cp, rm) | Перемещение оригиналов | Да |

Если Gmail-коннектор или Telegram Gateway недоступны — используйте только шаги
1 и 4 (ручной дроп в `inbox/`).
