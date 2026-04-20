---
type: device-import
date: 2025-01-01
tags: [apple-health, example]
source: apple-health
status: example
device: Apple Watch Series 9
export_range_start: 2024-10-01
export_range_end: 2024-12-31
origin_type: api
origin_file: devices/2025-01-01-apple-health-Q4.xml
---

> **EXAMPLE FILE.** Заглушка для показа формата bulk-импортов. Удалите после первого реального экспорта.

## Что в экспорте

- Формат: Apple Health XML
- Объём: ~30 MB
- Типы данных: HeartRate, HRV, Steps, ActiveEnergy, SleepAnalysis, BloodPressure

## Обработка

- [ ] Агрегаты по дням вынесены в `data/vitals/`, `data/sleep/`, `data/activity/`
- [x] Оригинал в `<ORIGINALS_DIR>/devices/`
