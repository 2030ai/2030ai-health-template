# Анализы крови

## Все анализы
```dataview
TABLE date, lab, tags, status
FROM "data/blood-tests"
WHERE status != "example"
SORT date DESC
```

## Требуют внимания
```dataview
TABLE date, lab, tags
FROM "data/blood-tests"
WHERE (contains(file.content, "❌") OR contains(file.content, "⚠️")) AND status != "example"
SORT date DESC
```
