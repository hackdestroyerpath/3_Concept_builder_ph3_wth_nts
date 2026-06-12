# Inbox

Parent: [Concept Builder](../README.md)  
Owner issue: `EXEC-006`  
Источник истины: `Inbox/README.md`  
Связанные протоколы: [new_issue_protocol.md](../Protocols/service_protocols/new_issue_protocol.md), [issue_retention_protocol.md](../Protocols/service_protocols/issue_retention_protocol.md)  
Status: `available-empty-entrypoint`  
Updated: `2026-06-05T09:52:51Z`

## Назначение

`Inbox` хранит входные материалы пользователя до и после превращения их в `issue`: текст точки входа, attachments, manifest, связь с issue и traceability. Это staging-зона источников, а не место для решений, requirements или output.

Смысл простой: сначала сохранить вход и связь, потом анализировать. Обратный порядок удобен только тем, кто любит спорить с собственной памятью.

## Что хранится в Inbox

| Объект | Назначение |
|---|---|
| `manifest.md` | главный readable manifest input packet |
| `entrypoint.md` или исходный entrypoint-файл | основной материал, от которого создаются issue |
| `raw_user_message.md` | исходное сообщение пользователя, если оно нужно для traceability |
| `attachments_manifest.jsonl` | машиночитаемая связь attachments с entrypoint и issue |
| `attachments/` | приложенные файлы, если они должны храниться в рабочем дереве |

Файлы внутри runtime input-папок создаются только через [new_issue_protocol.md](../Protocols/service_protocols/new_issue_protocol.md). Этот README сам по себе не разрешает неограниченное хранение произвольных материалов.

## Runtime layout

```text
Inbox/<input_id>/
├── manifest.md
├── entrypoint.md
├── raw_user_message.md
├── attachments_manifest.jsonl
└── attachments/
```

Если entrypoint является приложенным файлом с исходным расширением, `manifest.md` обязан указать его имя и роль. Если файл слишком тяжёлый, бинарный или хранится вне repository, manifest хранит ссылку/описание и status, а не выдуманный пересказ.

## Формат `input_id`

| Scope | Формат | Пример |
|---|---|---|
| root service input | `INBOX-YYYYMMDD-NNNN` | `INBOX-20260604-0001` |
| concept-local input | определяется concept template позже | `pending_EXEC-011` |

Номер монотонен внутри даты и не переиспользуется после cleanup, archive или tombstone.

## Минимальный `manifest.md`

`manifest.md` input packet должен содержать:

| Поле | Значение |
|---|---|
| `input_id` | стабильный ID input packet |
| `created_at` | timestamp получения |
| `source` | сообщение пользователя, uploaded file или external reference |
| `entrypoint` | главный входной материал |
| `attachments` | список приложений или `none` |
| `linked_issue_ids` | issue, созданные или связанные с этим input |
| `status` | `pending_entrypoint`, `saved`, `linked`, `archived`, `tombstone_pending` |
| `retention_note` | что удерживает input от cleanup |

## Минимальный `attachments_manifest.jsonl`

Каждая строка описывает один attachment:

```json
{"attachment_id":"ATT-0001","filename":"example.pdf","role":"supporting_attachment","linked_issue_ids":["SVC-ISS-0001"],"content_status":"stored_or_referenced","notes":"краткая проверяемая сводка"}
```

Если файл не прочитан, `content_status` должен явно говорить `not_read`, `binary_uninspected`, `external_reference` или другой честный статус. Сводка не должна притворяться чтением файла.

## Связь с issue

- [Issue registry](../Issues/issue_registry.md) хранит `input_ref` на `Inbox/<input_id>/manifest.md`.
- [Issue registry JSONL](../Issues/issue_registry.jsonl) хранит машинный `input_ref` и `linked_issue_ids`.
- `Issues/<issue_id>/reason.md` должен ссылаться на input packet, если issue возник из Inbox.
- `Issues/<issue_id>/state.md` должен указывать active input_ref, пока issue не закрыт или не архивирован.

## Правила хранения

1. Input packet остаётся доступным, пока хотя бы один linked issue имеет статус `creating`, `proposed`, `needs_discussion`, `approved`, `active`, `blocked`, `deferred`, `closed` или `archived`.
2. Rejected issue не удаляет input автоматически: сначала нужна retention-проверка.
3. Cleanup Inbox выполняется только через [issue_retention_protocol.md](../Protocols/service_protocols/issue_retention_protocol.md).
4. Tombstone должен сохранять `input_id`, linked issue, reason свёртки и cleanup reason.
5. Temporary или тяжёлые attachments можно удалять только после tombstone и проверки зависимостей.

## Что запрещено

- хранить requirements, solution, contract или output в `Inbox`;
- создавать input-папки без manifest;
- удалять attachment, если на него ссылается active/archived issue;
- смешивать несколько независимых issue без явной связи в manifest;
- записывать секреты или приватные данные без явного решения пользователя и понятного retention reason;
- создавать runtime files без обновления [../State/page_registry.jsonl](../State/page_registry.jsonl), если это Markdown-файлы.

## Связанные файлы

- [Root README](../README.md)
- [New issue protocol](../Protocols/service_protocols/new_issue_protocol.md)
- [Service start protocol](../Protocols/service_protocols/service_start_protocol.md)
- [Issue registry](../Issues/issue_registry.md)
- [Dependency graph](../Issues/dependency_graph.jsonl)
- [Persistence protocol](../Protocols/common/persistence_protocol.md)
- [Issue Retention protocol](../Protocols/service_protocols/issue_retention_protocol.md)
- [Page registry](../State/page_registry.jsonl)
