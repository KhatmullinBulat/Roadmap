---
type: dashboard
---

# Учебная панель

## Roadmap

```dataview
TABLE status AS "Статус", priority AS "Приоритет", created AS "Создано"
FROM "01 Roadmap"
WHERE type = "roadmap"
SORT priority ASC, file.name ASC
```

## Незавершённые темы

```tasks
not done
path includes 01 Roadmap
group by filename
sort by path
```

## Недавно изменённые заметки

```dataview
TABLE file.mtime AS "Изменено"
FROM "02 Knowledge"
SORT file.mtime DESC
LIMIT 10
```
