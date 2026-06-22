# Development History

Журнал итераций. Каждая заметная задача — одна запись. Последняя запись — вверху.

## Формат

```markdown
## [YYYY-MM-DD] Короткое название
- Что сделано
- Зачем (если не очевидно из задачи)
- Затронутые файлы/каталоги
```

## Ротация

При достижении ~30 записей — переносить самые старые в `development-history-archive.md`, сохраняя в этом файле последние 10.

---

## [2026-06-22] Skill command-name metadata

- Skill `/health-update` нормализован: `name`, description, H1 и `agents/openai.yaml display_name` совпадают со slash-командой.
- Добавлен `agents/openai.yaml` для UI metadata.
- Проверка: metadata-scan по публичным template/upstream skills показывает `issues 0`.

## [YYYY-MM-DD] Vault scaffolded from 2030ai-health-template

- Создана базовая структура vault из шаблона [2030ai/2030ai-health-template](https://github.com/2030ai/2030ai-health-template).
- Настроен `<ORIGINALS_DIR>` — путь хранения оригиналов.
- Подключены MCP: (указать какие).
- Первая запись: (указать файл).

*(Удалите этот placeholder и начните свой журнал.)*
