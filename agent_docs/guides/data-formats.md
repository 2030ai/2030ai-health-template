# Спецификация форматов данных

## Универсальные поля frontmatter

Каждый файл в `data/` — markdown с YAML frontmatter. Обязательные поля:

| Поле | Значения | Описание |
|---|---|---|
| `type` | `blood-test`, `vital`, `sleep`, `activity`, `medication`, `doctor-visit`, `symptom`, `mental`, `dental`, `vision`, `vaccination`, `family-history`, `nutrition`, `device-import`, `imaging`, `other` | Тип записи |
| `date` | `YYYY-MM-DD` | Дата события |
| `tags` | `[тег1, тег2]` | Теги для поиска |
| `source` | `inbox`, `manual`, `apple-health`, `fitbit`, `gmail`, `telegram`, `google-drive`, `portal` | Откуда данные |
| `status` | `raw`, `verified`, `reviewed`, `example` | Статус обработки |

Значение `status: example` зарезервировано за файлами-заглушками из шаблона; удаляйте их, когда добавите реальные записи.

## Поля провенанса

Опциональные. Если `source: manual` — поля провенанса **опускаются** (не оставляются пустыми):

| Поле | Описание |
|---|---|
| `origin_url` | URL первоисточника (email, telegram, портал) |
| `origin_type` | `email`, `telegram`, `pdf`, `photo`, `manual`, `api` |
| `origin_file` | Имя файла-оригинала внутри `<ORIGINALS_DIR>` |
| `origin_date` | Дата получения оригинала (может отличаться от `date`) |

## Опциональные доменные поля

Для некоторых типов есть полезные доменные поля (используйте по необходимости):

- `blood-test`: `lab` (название лаборатории), `referred_by` (направил врач)
- `doctor-visit`: `doctor`, `clinic`
- `medication`: `drug`, `dose`, `frequency`, `prescribed_by`, `start_date`, `end_date`, `reason`
- `symptom`: `severity` (1–10), `duration`
- `vaccination`: `vaccine`, `batch`

Эти поля не являются обязательными, но помогают Dataview-дашбордам делать полезные таблицы.

## Именование файлов

Конвенция: `YYYY-MM-DD-<описание>.md`

Примеры:
- `data/blood-tests/2025-04-20-biochemistry.md`
- `data/doctor-visits/2025-04-18-therapist-smith.md`
- `data/vitals/2025-04-20.md`

При дублях в один день — суффикс: `2025-04-20-biochemistry-2.md`.

Язык описания — на ваше усмотрение (русский/английский/любой). Важно: используйте дефисы, а не пробелы — Obsidian wiki-links их поддерживают, но дефисы безопаснее для git diff и CLI.

## Связи между записями

Wiki-links `[[]]` для связей: визит → анализ → лекарство → симптом.

- Использовать **полное имя файла** без `.md`: `[[2025-04-18-therapist-smith]]`
- Опережающие ссылки допустимы (Obsidian покажет как несуществующие до создания файла).

## Оригиналы файлов

- Оригиналы (PDF, фото, экспорты) хранятся в `<ORIGINALS_DIR>`.
- Рекомендуемая структура подпапок внутри `<ORIGINALS_DIR>`: `blood-tests/`, `doctor-visits/`, `imaging/`, `dental/`, `vision/`, `devices/`, `prescriptions/`, `other/`.
- В markdown-файле ссылка через `origin_file` — относительный путь от `<ORIGINALS_DIR>`.
