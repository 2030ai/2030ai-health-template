# Каталог источников health-данных

Справочник: откуда брать данные, в каком виде они приходят, как парковать в vault. Разделён на три блока:

- **A. Проверено на практике** — использовано в референс-проекте, работает, есть примеры.
- **B. Потенциальные источники** — рекомендовано подключить, но в референсе не тестировалось.
- **C. Интеграционные каналы** — MCP, API, ручные процедуры.

Каждая позиция: *что даёт*, *как извлечь*, *периодичность*, *формат результата*, *куда в vault*.

---

## A. Проверено на практике

### A.1. Медицинские лаборатории (PDF-отчёты)

Лаборатории, которые подтверждены как работающие источники в референс-проекте:

- **Россия/СНГ**: Инвитро, КДЛ, Гемотест, Хеликс, ЦитиЛаб, ДНКОМ, РДКМ
- **US**: Labcorp, Quest Diagnostics
- **EU**: Synlab, Eurofins
- **Локальные**: любая лаборатория, которая отдаёт результаты PDF'ом на email или через личный кабинет

*Что даёт:* общий анализ крови (CBC), биохимия, гормоны, щитовидка, витамины, иммунология, онкомаркеры, инфекции, аллергены, спермограмма, анализ мочи, кал, ПЦР.

*Как извлечь:*
1. Личный кабинет лаборатории → скачать PDF.
2. Или дождаться письма с результатом → скачать attachment.
3. Положить в `inbox/` → запустить `/health-update` → агент создаст запись в `data/blood-tests/YYYY-MM-DD-<test>.md`, оригинал уйдёт в `<ORIGINALS_DIR>/blood-tests/`.

*Периодичность:* по факту сдачи (обычно раз в 3–12 месяцев для рутины).

*Формат результата:* PDF, иногда с водяными знаками. Хорошо OCR-ится; агент парсит таблицу с показателями в markdown.

*Куда в vault:* `data/blood-tests/`.

### A.2. Gmail (письма от клиник и лабораторий)

*Что даёт:* нотификации о готовности анализов, счета, напоминания о визитах, digital-отчёты.

*Как извлечь:*
- Через MCP `google-mcp-readonly`: `gmail_search` с фильтром `from:` по домену лаборатории или ключевым словам.
- **Важно:** MCP не скачивает attachments — PDF-вложения нужно тянуть вручную через веб-клиент, потом класть в `inbox/`.

*Периодичность:* автоматический поиск в `/health-update` запускайте раз в 1–4 недели.

*Формат результата:* тело письма (markdown) + PDF вложения (отдельно).

*Куда в vault:* зависит от типа — чаще всего `data/blood-tests/` или `data/doctor-visits/`.

### A.3. Telegram — личный health-чат

*Что даёт:* фото упаковок лекарств, скриншоты назначений от врача, короткие заметки «болит голова третий день», голосовые → транскрипты.

*Как извлечь:*
- MCP `telegram-read` → `chat_ops`, `message_ops`.
- Для media используйте `download_media` или заранее настроенный `download_dir`.

*Периодичность:* при каждом запуске `/health-update` подтягиваются новые сообщения с последнего watermark.

*Формат результата:* текст + ссылки на media (jpg, ogg, pdf).

*Куда в vault:* зависит от типа, определяется агентом по контенту.

*Настройка:* имя чата указывается как `<TG_CHAT>` в [health-update-workflow.md](health-update-workflow.md).

### A.4. Google Drive / OneDrive / iCloud Drive

*Что даёт:* исторический архив PDF-ов, сканов, фотографий, загруженных вручную за годы.

*Как извлечь:*
- MCP `google-mcp-readonly: drive_search` + `drive_download` (если есть).
- Или открыть папку и скачать batch вручную.
- Положить в `inbox/` → обработать.

*Периодичность:* одноразовый или редкий импорт.

*Формат результата:* PDF, JPG, DOCX.

*Куда:* рассортировать по `data/<type>/`.

### A.5. OneNote / Apple Notes / Notion / DayOne / Evernote

*Что даёт:* архив заметок о здоровье за 5–15 лет (визиты, самочувствие, эксперименты с диетами).

*Как извлечь:*
- OneNote: экспорт в `.onepkg` или массовая печать в PDF. Или `onenote-md-exporter` (сторонняя утилита).
- Apple Notes: Share → Send a Copy → Markdown (macOS 15+), или `apple-notes-exporter` утилитой.
- Notion: Export → Markdown.
- DayOne: Export → JSON.
- Evernote: Export → ENEX → конвертер в markdown.

*Периодичность:* одноразовая миграция.

*Формат результата:* markdown или HTML (в лучшем случае), PDF (в худшем).

*Куда:* Агент должен рассортировать по типу и дате, создавая по одной записи на событие. Ссылки на медиа (из attachments) переносятся в `<ORIGINALS_DIR>/`.

### A.6. Apple Health

*Что даёт:* пульс, HRV, давление, вес, сон, активность, тренировки, ЭКГ (Apple Watch), слух, падения, менструальный цикл.

*Как извлечь:*
- **Shortcut** в iOS: `Health → Find Health Sample → export to CSV → share to Files/iCloud`. Положить CSV в `inbox/`.
- Или массовый экспорт: `Health app → profile icon → Export All Health Data` → `export.zip` → распаковать → `export.xml`. Это bulk-формат; класть в `data/devices/apple-health-YYYY-MM-DD.md` как `device-import`, затем раскладывать агрегаты в `vitals/`, `sleep/`, `activity/`.

*Периодичность:* bulk раз в полгода-год; узкие срезы (давление на неделе) по запросу.

*Формат результата:* XML / CSV.

*Куда:* сырой импорт — `data/devices/`; отфильтрованные дневные агрегаты — `data/vitals/`, `data/sleep/`, `data/activity/`.

### A.7. Fitbit / Garmin / Oura / Whoop / Withings / Polar / Google Fit

*Что даёт:* сон (стадии, HRV), readiness score, тренировки, GPS-треки, VO2max, вес, SpO2.

*Как извлечь:*
- **Fitbit**: `Settings → Data Export` (ZIP с JSON).
- **Garmin Connect**: `Export` per activity или [garmin-connect-export](https://github.com/pe-st/garmin-connect-export) утилитой.
- **Oura**: API (Cloud API v2) или ручной CSV-экспорт из web.
- **Whoop**: Developer API.
- **Withings**: `account.withings.com → Settings → Data Export`.
- **Polar**: Flow web → Export.
- **Google Fit**: Google Takeout.

*Периодичность:* ежемесячно или ежеквартально.

*Формат результата:* JSON / CSV / GPX (активности).

*Куда:* `data/devices/` (сырое) → `data/sleep/`, `data/activity/`, `data/vitals/` (агрегаты).

### A.8. Генетика (Atlas, 23andMe, MyHeritage, Invitae, AncestryDNA)

*Что даёт:* SNP-файл (500K–1M+ SNP), интерпретации, предрасположенности, фармакогенетика, происхождение.

*Как извлечь:*
- 23andMe: `Browse Raw Data → Download Raw Data`.
- MyHeritage: `DNA → Manage DNA Kits → Download raw data`.
- Atlas: email от лаборатории или кабинет.
- Invitae: PDF-отчёт + VCF по запросу.

*Периодичность:* одноразово (генетика не меняется).

*Формат результата:* txt/tsv с rsID + nucleotide calls, плюс PDF-интерпретация.

*Куда:* `data/genetics/YYYY-MM-DD-<provider>.md` (метаданные + ключевые SNP), сам raw-файл — в `<ORIGINALS_DIR>/genetics/`.

### A.9. Имиджинг (КТ / МРТ / УЗИ / рентген / стоматологические снимки)

*Что даёт:* заключения радиолога (текст), собственно изображения (DICOM).

*Как извлечь:*
- Клиника отдаёт CD/USB → DICOM-папка.
- Или портал клиники → скачать ZIP.
- Стоматология: Galileos / Sirona / Planmeca / Vatech — платформо-специфичные форматы.

*Периодичность:* по факту исследования.

*Формат результата:* заключение — PDF или текст; изображения — DICOM (`.dcm`), иногда `.mha`, `.nii`.

*Куда:*
- **Метаданные + заключение** → `data/imaging/YYYY-MM-DD-<type>.md` (в git).
- **DICOM-архивы** → `<ORIGINALS_DIR>/imaging/YYYY-MM-DD-<type>/` (НЕ в git — они могут быть по 100–1000 MB).

### A.10. Медкарта / ЭМК / порталы пациентов

Списком, по регионам:

- **US**: MyChart (Epic), Athena Patient, Cerner, Kaiser My Health Manager
- **UK**: NHS App
- **Russia**: Госуслуги.Здоровье, ЕМИАС (Москва)
- **Ukraine**: Helsi
- **Germany**: TI-Messenger, ePA
- **Israel**: Clalit, Maccabi Online
- **Global**: страховые (ДМС, Cigna, BUPA, AXA) — портал клиента.

*Что даёт:* выписки из карты, рецепты, результаты посещений, план лечения.

*Как извлечь:* скачать PDF из портала → `inbox/`.

*Периодичность:* после каждого визита или ежеквартальный ревью.

*Куда:* `data/doctor-visits/` или `data/medications/` (рецепты).

### A.11. Apple Notes / Apple Reminders — текущий быстрый input

*Что даёт:* короткие заметки «давление 120/80 утром», «выпил Vit D», «болит спина с понедельника».

*Как извлечь:* Export markdown (macOS Notes 15+) → `inbox/`. Или shortcut «Append to inbox».

*Периодичность:* раз в неделю или по триггеру.

*Куда:* агент классифицирует по типу.

### A.12. Ручной ввод

*Что даёт:* всё, что нельзя автоматизировать: рассказ бабушки про семейный анамнез, ощущения после тренировки, субъективная оценка самочувствия.

*Как извлечь:* прямой диалог с агентом: «добавь запись: сегодня мерил давление три раза — …».

*Формат:* `source: manual`, поля провенанса опускаются.

*Куда:* `data/<type>/` — по содержанию.

---

## B. Потенциальные источники (рекомендовано подключить)

### B.1. Continuous Glucose Monitors (CGM)

Dexcom G7, FreeStyle Libre, Levels, Nutrisense, Supersapiens.

*Что даёт:* глюкоза каждые 5 минут, реакция на еду и стресс, ночные тренды.

*Как извлечь:* Dexcom Clarity → Export; Libre → LibreView CSV; Levels/Nutrisense API. Ручной ежемесячный дамп в `inbox/`.

*Куда:* `data/devices/` (сырое) + `data/vitals/` (суммаризация по дням).

### B.2. Eight Sleep / Sleep Number / Tempur-Pedic smart beds

*Что даёт:* температура, HRV ночью, движения, качество сна по фазам.

*Как извлечь:* Eight Sleep web → Export (или API). Sleep Number — только через app-скриншоты.

*Куда:* `data/sleep/`.

### B.3. Питание

- **Cronometer** — самый детальный по микронутриентам. CSV export.
- **MyFitnessPal** — калории и макро. CSV export (платно).
- **Yazio / Lose It / Noom** — аналогично, CSV по подписке.
- **Фото-дневник еды** (просто фотки) — в `<ORIGINALS_DIR>/nutrition/`, описание — в `data/nutrition/`.

*Куда:* `data/nutrition/`.

### B.4. Ментальное здоровье

- **Headspace / Calm / Waking Up** — история медитаций (экспорт ограничен, в основном скриншоты).
- **Moodnotes / Daylio / How We Feel** — трекеры настроения с CSV-экспортом.
- **Finch / Fabulous** — геймифицированные habit trackers.
- **Therapy-notes** (если есть терапевт) — ручной ввод по согласованию с терапевтом.

*Куда:* `data/mental/`.

### B.5. Тренировки (structured)

- **Strava** — GPX + комментарии, экспорт ZIP из аккаунта.
- **Nike Run Club / Adidas Running** — экспорт ограничен.
- **TrainingPeaks / Final Surge** — если занимаетесь со структурированным планом; CSV + FIT.
- **Hevy / Strong / Fitbod** — силовые тренировки, CSV export.

*Куда:* `data/activity/`.

### B.6. Женские циклы

Clue, Flo, Natural Cycles, Apple Health Cycle Tracking.

*Что даёт:* фазы цикла, симптомы, корреляция с настроением/сном.

*Как извлечь:* Clue → Data Export (CSV). Apple Health Cycle → через общий Health export.

*Куда:* `data/vitals/` или отдельная категория, если пользователь добавит.

### B.7. Стоматологические и офтальмологические устройства

- Oral-B iO / Philips Sonicare — продолжительность и покрытие чистки (CSV).
- Smart flossers (Waterpik).
- Домашние тонометры глазного давления.
- OCT scans (если есть доступ).

*Куда:* `data/dental/`, `data/vision/`.

### B.8. Аптечные чеки и назначения

- **СберСпасибо / СберЗдоровье / Ozon Pharm / Apteka.ru** — история покупок.
- **Walgreens / CVS / Boots / любая сетевая аптека** — история через app.
- **ФНС России / receipt-сервисы** — электронные чеки.

*Что даёт:* реальный учёт — что и когда покупалось (более точно, чем ручные назначения).

*Куда:* `data/medications/`.

### B.9. Семейный анамнез

*Источник:* Google Form или Notion-анкета для родственников. Запрос: год рождения / год смерти / диагнозы.

*Куда:* `data/family-history/` — по одному файлу на родственника.

### B.10. Телеметрия окружения

- **Airly / PurpleAir / Atmotube / Awair** — качество воздуха.
- **Home Assistant / SmartThings** — температура, влажность, свет дома.
- **Ring / Nest / Arlo** — косвенно паттерны сна через активность.

*Куда:* `data/other/` — если коррелируете с самочувствием.

### B.11. ChatGPT / Claude / Perplexity — история медицинских чатов

Если вы обсуждали анализы с ИИ — экспорт чата в markdown.

*Куда:* `data/doctor-visits/` или `data/other/` как "AI consultation".

### B.12. Лабораторные API (для автоматизации)

- **Invitro API / Labcorp API / Quest API** — автоматический pull результатов (если пользователь готов регистрировать app).
- **HealthKit API** (iOS-приложение-мост).

*Куда:* прямой pipeline в `inbox/` через cron/shortcut.

### B.13. Обучение и книги

- **Anki deck** с медицинскими терминами.
- **Конспекты прочитанных книг** по здоровью.

*Куда:* `data/other/` — для контекста, не для трекинга.

---

## C. Интеграционные каналы

### C.1. Claude Code MCP

- **`google-mcp-readonly`** — Gmail, Drive, Calendar, Sheets, Docs (read-only). Рекомендуется всем.
- **`telegram-read`** — чтение Telegram (сообщения, контакты, кэш). Если ведёте заметки в TG.
- **`playwright`** — headless browser для порталов без API (Госуслуги, MyChart). Осторожно с 2FA.
- **Custom MCP** — можно написать свой под конкретный портал (есть гайд в `anthropic-skills:mcp-builder`).

### C.2. iOS Shortcuts

- Shortcut «Append to inbox» — выбранный файл/заметку кладёт в `inbox/` через iCloud Drive синхронизацию.
- Shortcut «Quick vital entry» — голосом → CSV → `inbox/`.
- Shortcut «Export Apple Health sample» — экспорт конкретного HKQuantityType в CSV.

### C.3. Ручной дроп в `inbox/`

Базовый fallback — всегда работает, не требует настроек. Скачали PDF → положили в `inbox/` → запустили агента. Годится для 90% источников.

### C.4. Скрипты (в крайнем случае)

Разрешены одноразовые скрипты (`temp/parse-health-export.py`), но результаты должны быть сохранены в `data/` как обычные markdown-файлы. Долго живущие pipeline не поощряются — этот проект намеренно без сервера.

---

## Приоритизация

Если начинаете с нуля, порядок подключения:

1. **inbox/** + одна лаборатория — самый быстрый эффект.
2. **Gmail** (через `google-mcp-readonly`) — автоматизирует п.1.
3. **Apple Health / Fitbit / Oura** — пассивная телеметрия, без ручного ввода.
4. **Историческая миграция** (OneNote / Apple Notes) — одноразовый проект на выходные.
5. **Расширение дашбордов** под свои приоритеты.
6. **Нишевые источники** (CGM, питание, ментальное) — если конкретный запрос появится.

Не подключайте всё сразу — это приведёт к тому, что данные будут приходить, а обрабатываться не будут. Один новый источник в месяц — здоровый темп.
