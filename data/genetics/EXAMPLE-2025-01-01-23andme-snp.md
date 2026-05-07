---
type: genetics
date: 2025-01-01
tags: [snp, 23andme, example]
source: manual
status: example
provider: 23andMe
kit_format: SNP-array
coverage: ~650k SNPs
origin_type: api
origin_file: genetics/2025-01-01-23andme-raw-data.txt
---

> **EXAMPLE FILE.** Заглушка, показывающая формат записи о генетическом тесте. Удалите после первого реального импорта.

## Обзор экспорта

- Провайдер: 23andMe
- Покрытие: ~650 000 SNP (SNP-array, не WGS)
- Формат raw: TSV (rsID + chromosome + position + genotype)
- Raw-файл — в `<ORIGINALS_DIR>/genetics/`

## Ключевые находки

- MTHFR C677T — условное значение
- APOE — условное значение
- CYP2D6 (фармакогенетика) — условное значение

Полная интерпретация — отдельным отчётом провайдера или через сторонние сервисы (Promethease, SelfDecode).

## Заметки

Не делайте автоматических медицинских выводов по SNP. Для фармакогенетики и наследственных рисков — консультация с врачом-генетиком.
