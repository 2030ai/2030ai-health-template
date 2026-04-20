# Симптомы

## Последние
```dataview
TABLE date, severity, duration, tags
FROM "data/symptoms"
WHERE status != "example"
SORT date DESC
LIMIT 30
```

## Частые (по тегам)
```dataview
TABLE length(rows) AS "Кол-во", rows.date AS "Даты"
FROM "data/symptoms"
WHERE status != "example"
FLATTEN tags AS tag
GROUP BY tag
SORT length(rows) DESC
```
