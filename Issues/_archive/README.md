# Архив issue

Parent: [Issues/issue_registry.md](../issue_registry.md)  
Owner issue: `USER-006` / `EXEC-005`  
Источник истины: `Issues/_archive/README.md`  
Связанный протокол: [issue_retention_protocol.md](../../Protocols/service_protocols/issue_retention_protocol.md)  
Status: `available-empty-entrypoint`  
Updated: `2026-06-05T10:46:09Z`

## Назначение

Эта папка хранит архивированные issue, которые закрыты или отклонены, но ещё нужны для traceability. Сейчас архив может быть пустым: runtime issue ещё не создавались, а bootstrap implementation issue существуют в registry-only форме.

Архив не является активной очередью. Он хранит историю, decision и evidence до тех пор, пока [issue_retention_protocol.md](../../Protocols/service_protocols/issue_retention_protocol.md) не разрешит tombstone-свёртку.

## Правила архивации

Issue может попасть в `_archive` только после одного из событий:

1. status `closed` и validation/contract показывают pass;
2. status `rejected` и сохранены reason, decision и input refs;
3. active queue больше не нуждается в полном issue context;
4. [dependency_graph.jsonl](../dependency_graph.jsonl) не содержит active blocking dependency, требующей issue в основной папке;
5. retention decision packet сохранён через [issue_retention_protocol.md](../../Protocols/service_protocols/issue_retention_protocol.md).

## Минимальное содержимое архивной записи

```text
Issues/_archive/<issue_id>/
├── state.md
├── reason.md
├── decision_log.md
├── requirements.md
├── solution.md
├── contract.md
├── output/
└── validation_report.md
```

Фактический набор может быть меньше или больше, если это обосновано retention-протоколом. Для registry-only bootstrap issue пустая архивная папка не создаётся: достаточно registry marker и persistence log.

## Требования к `decision_log.md`

Архивная запись должна объяснять:

- почему issue был закрыт, отклонён или выведен из active queue;
- какие input refs и attachments были использованы;
- какие dependencies были проверены;
- какие output/validation refs остаются значимыми;
- можно ли в будущем выполнить tombstone-свёртку.

## Связь с tombstones

После batch cleanup архивная запись может быть свёрнута в tombstone: [Issues/_tombstones/README.md](../_tombstones/README.md). До создания tombstone удалять исторически значимые файлы нельзя.

## Запреты

- Не создавать archive folder для registry-only bootstrap issue без runtime artifacts.
- Не удалять original input, если он единственный источник reason или scope.
- Не переносить Markdown-файл без обновления [../../State/page_registry.jsonl](../../State/page_registry.jsonl).
- Не заявлять GitHub cleanup, если запись существует только в checkpoint package.

## Связанные файлы

- [Issue registry](../issue_registry.md)
- [Issue registry JSONL](../issue_registry.jsonl)
- [Dependency graph](../dependency_graph.jsonl)
- [Issue retention protocol](../../Protocols/service_protocols/issue_retention_protocol.md)
- [Tombstone entry point](../_tombstones/README.md)
- [Inbox](../../Inbox/README.md)
