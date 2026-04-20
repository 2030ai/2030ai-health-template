# Прививки и скрининги

## История прививок
```dataview
TABLE date, tags
FROM "data/vaccinations"
WHERE status != "example"
SORT date DESC
```
