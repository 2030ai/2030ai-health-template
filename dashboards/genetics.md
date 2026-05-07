# Генетические тесты

## Все импорты
```dataview
TABLE date, provider, kit_format, coverage, status
FROM "data/genetics"
WHERE status != "example"
SORT date DESC
```

## Заметки

Генетика обычно один-два импорта за всю жизнь. Дашборд служит индексом записей; raw-файлы (SNP-array, VCF, PDF-интерпретации) лежат в `<ORIGINALS_DIR>/genetics/`, не в git.
