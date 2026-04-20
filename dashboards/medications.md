# Лекарства

## Активные курсы
```dataview
TABLE drug, dose, frequency, start_date
FROM "data/medications"
WHERE (!end_date OR end_date >= date(today)) AND status != "example"
SORT start_date DESC
```

## Завершённые
```dataview
TABLE drug, dose, start_date, end_date, reason
FROM "data/medications"
WHERE end_date AND end_date < date(today) AND status != "example"
SORT end_date DESC
```
