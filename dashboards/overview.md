# Обзор здоровья

## Последние записи
```dataview
TABLE date, type, tags, status
FROM "data"
WHERE status != "example"
SORT date DESC
LIMIT 20
```

## Непроверенные (raw)
```dataview
TABLE date, type, tags
FROM "data"
WHERE status = "raw"
SORT date DESC
```

## Предстоящие визиты
```dataview
TABLE doctor, clinic
FROM "data/doctor-visits"
WHERE date >= date(today)
SORT date ASC
```

## Активные лекарства
```dataview
TABLE drug, dose, frequency, start_date
FROM "data/medications"
WHERE !end_date OR end_date >= date(today)
SORT start_date DESC
```
