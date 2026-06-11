# Протокол сохранения состояния

Parent: [Каталог протоколов](../catalog.md)  
Owner issue: `EXEC-004`  
Protocol ID: `common/persistence_transaction`  
Источник истины: `Protocols/common/persistence_protocol.md`  
Status: `available`  
Updated: `2026-06-05T11:45:45Z`

## Назначение

Этот протокол задаёт transaction-like порядок записи рабочих файлов. GitHub не обязан давать атомарность на пачку файлов, поэтому агент обязан делать запись проверяемой и восстановимой.

## Preconditions

- Active scope определён через [context_loading_protocol.md](context_loading_protocol.md).
- Target files входят в approved scope текущего issue/protocol.
- Write set составлен до изменения файлов.
- Для GitHub-записи известны repository, branch и допустимый write scope. Если они неизвестны, операция остаётся `package_draft_not_committed` или `blocked_on_persistence`.

## Write set

Перед записью агент фиксирует список объектов:

| Поле | Значение |
|---|---|
| `operation` | `create`, `update`, `move`, `archive`, `delete` |
| `path` | относительный путь от root репозитория |
| `owner_issue` | issue или backlog ID |
| `reason` | почему файл меняется |
| `source_of_truth` | какой файл становится главным источником |
| `registry_updates` | какие entries должны измениться |

## Порядок записи

1. **Preflight read**: перечитать latest [service_state.md](../../State/service_state.md) или relevant concept state, [page_registry.jsonl](../../State/page_registry.jsonl), registry и target files.
2. **Conflict check**: если target изменился после загрузки, reload и безопасный merge; при конфликте — abort с blocker.
3. **Apply content first**: сохранить primary artifacts: instructions, protocol, issue file, requirements, solution, output, page или export.
4. **Update indexes after content**: обновить state, issue registry, page registry, backlinks, dependency graph и parent summaries.
5. **Light validation**: проверить ссылки изменённых Markdown-файлов, наличие parent link, orphan-risk и границу production/development.
6. **Commit marker last**: последней записью добавить JSONL-строку в [persistence_log.jsonl](../../State/persistence_log.jsonl).
7. **Response after persistence**: ответить пользователю только тем статусом, который уже отражён в сохранённых файлах.

## Формат persistence log entry

Минимальные поля строки [persistence_log.jsonl](../../State/persistence_log.jsonl):

```json
{"transaction_id":"...","timestamp":"...","mode":"Service Mode","issue_id":"...","scope":"...","repository":"...","branch":"...","committed":false,"status":"package_draft_not_committed","write_set":["..."],"next_state":"...","notes":"..."}
```

`committed=false` означает, что создан package draft или локальный checkpoint, но GitHub commit не подтверждён. `committed=true` допустим только после фактической успешной записи в GitHub и проверки commit marker.

## Failure behavior

| Сбой | Действие |
|---|---|
| Нет write-инструмента GitHub | вернуть `package_draft_not_committed`, приложить архив или write set |
| Нет прав или branch protected | вернуть `blocked_on_persistence`, не заявлять сохранение |
| Conflict target file | reload, merge или abort с blocker |
| Запись оборвалась до commit marker | записать failed transaction, если возможно; иначе сообщить uncertain write set |
| Validation failed | не закрывать issue; сохранить report и next repair step |

## Completion signal

Операция завершена, когда выполнено одно из двух:

1. `committed=true`: commit marker записан последним, changed files и state согласованы;
2. `committed=false`: package draft или blocker честно отражён в state/log, а пользователь получил ссылку на артефакт или write set.

## Связанные файлы

- [Context loading protocol](context_loading_protocol.md)
- [Service state](../../State/service_state.md)
- [Persistence log](../../State/persistence_log.jsonl)
- [Page registry](../../State/page_registry.jsonl)
