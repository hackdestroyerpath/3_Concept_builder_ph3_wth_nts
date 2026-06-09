# Tombstone issue

Parent: [Issues/issue_registry.md](../issue_registry.md)  
Owner issue: `USER-006` / `EXEC-005`  
Источник истины: `Issues/_tombstones/README.md`  
Связанный протокол: [issue_retention_protocol.md](../../Protocols/service_protocols/issue_retention_protocol.md)  
Status: `available-empty-entrypoint`  
Updated: `2026-06-05T10:46:09Z`

## Назначение

Эта папка хранит минимальные исторические записи по issue, чья полная архивная история была свёрнута после cleanup. Tombstone не заменяет registry и dependency graph: он связывает ID, reason, decision, input refs и cleanup reason в компактную проверяемую запись.

## Когда разрешён tombstone

Tombstone создаётся только если выполнены условия:

1. issue уже `closed`, `rejected` или `archived`, либо есть отдельный retention decision для registry-only записи;
2. нет active blocking dependency на полный архив issue;
3. reason, decision, parent/children/linked refs и output refs можно свернуть без потери проверяемой истории;
4. cleanup reason сохранён;
5. [issue_registry.jsonl](../issue_registry.jsonl), [dependency_graph.jsonl](../dependency_graph.jsonl) и [../../State/page_registry.jsonl](../../State/page_registry.jsonl) обновлены;
6. действие выполняется по [issue_retention_protocol.md](../../Protocols/service_protocols/issue_retention_protocol.md).

## Минимальное содержимое tombstone-файла

```text
Issues/_tombstones/<issue_id>.md
```

Файл должен содержать:

- `issue_id`;
- title;
- previous status;
- reason summary;
- decision summary;
- input refs;
- parent/children/linked refs;
- dependency refs;
- output refs;
- archive refs, если были;
- cleanup timestamp;
- cleanup reason;
- deleted paths или `none`.

## Запреты

- Tombstone-файл не удаляется.
- Registry row не удаляется.
- Dependency edge не удаляется без отдельной migration/repair transaction.
- Удаление тяжёлых или временных файлов допустимо только после tombstone и validation cleanup.
- ID не переиспользуется после tombstone или deletion.

Связанный архивный вход: [Issues/_archive/README.md](../_archive/README.md).

## Связанные файлы

- [Issue registry](../issue_registry.md)
- [Issue registry JSONL](../issue_registry.jsonl)
- [Dependency graph](../dependency_graph.jsonl)
- [Issue retention protocol](../../Protocols/service_protocols/issue_retention_protocol.md)
- [Archive entry point](../_archive/README.md)
- [Inbox](../../Inbox/README.md)
