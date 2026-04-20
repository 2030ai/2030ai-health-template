# Предстоящие визиты

```dataview
TABLE date, doctor, clinic, tags
FROM "data/doctor-visits"
WHERE date >= date(today) AND status != "example"
SORT date ASC
```
