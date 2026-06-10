# Inbox

Parent: [README](../README.md)  
Owner issue: `EXEC-006` / `USER-006`  
Источник истины: `Inbox/README.md`  
Связанные протоколы: [new_issue_protocol.md](../Protocols/service_protocols/new_issue_protocol.md), [issue_retention_protocol.md](../Protocols/service_protocols/issue_retention_protocol.md)  
Status: `available-empty-entrypoint`  
Updated: `2026-06-05T10:46:09Z`

## Назначение

`Inbox/` хранит входящие пользовательские материалы, attachments и input packets до создания или обновления issue. Эта папка является staging-областью с traceability, а не долгосрочным архивом.

Файлы внутри runtime input-папок создаются только через [new_issue_protocol.md](../Protocols/service_protocols/new_issue_protocol.md). Этот README сам по себе не разрешает неограниченное хранение произвольных материалов.

## Runtime layout

```text
Inbox/
├── README.md
└── <input_id>/
    ├── manifest.md
    ├── raw_user_input.md
    ├── attachments/
    └── intake_decision.md
```

Runtime input folder создаётся только если пользовательский input требует сохранения перед созданием issue или attachment нельзя безопасно свернуть в registry summary.

## `input_id`

Формат:

```text
INBOX-YYYYMMDD-HHMMSS-NNNN
```

`input_id` не должен переиспользоваться. Если input ошибочный или rejected, ID сохраняется в tombstone/retention trace.

## `manifest.md`

Минимальные поля:

| Field | Значение |
|---|---|
| `input_id` | стабильный ID input packet |
| `created_at` | ISO timestamp |
| `source` | `user_message`, `uploaded_file`, `external_reference`, `manual_note` |
| `linked_issue_ids` | issue IDs, которые используют input |
| `attachment_count` | число attachments |
| `retention_status` | `active`, `archived`, `tombstone`, `deleted_nonhistorical` |
| `cleanup_reason` | required only after cleanup |

## `raw_user_input.md`

Хранит исходную формулировку пользователя, если она нужна для reason, requirements или later audit. Если input содержит чувствительные данные, агент обязан сохранить только минимальный summary и указать `source_redacted=true` в manifest.

## Attachments

Attachments хранятся в `Inbox/<input_id>/attachments/` только если они нужны для issue reasoning или output evidence. Для каждого attachment нужен manifest entry:

| Field | Значение |
|---|---|
| `attachment_id` | локальный ID |
| `filename` | исходное имя или normalized name |
| `role` | source, evidence, reference, output-input |
| `size` | if known |
| `hash` | if available |
| `linked_issue_ids` | issue IDs |
| `retention_status` | active/archive/tombstone/delete_nonhistorical |

## Intake decision

`intake_decision.md` объясняет, как input стал issue или почему был отклонён.

Минимальные разделы:

```text
# Intake decision

Input ID: <input_id>
Decision: create_issue | attach_to_existing_issue | reject | defer
Reason:
Linked issue:
Protocol used:
Next action:
```

## Правила использования

1. Не создавать `Inbox/<input_id>/` без reason.
2. Не хранить attachments без manifest entry.
3. Не переносить input в issue без backlink из issue `reason.md` или registry.
4. Не удалять input packet, пока linked issue не закрыты или не tombstone.
5. Cleanup Inbox выполняется только через [issue_retention_protocol.md](../Protocols/service_protocols/issue_retention_protocol.md).
6. Tombstone manifest должен сохранять `input_id`, linked issue, reason свёртки и cleanup reason.
7. Temporary или тяжёлые attachments можно удалять только после tombstone и проверки зависимостей.

## Add-during-approval branch

Если пользователь во время approval/review добавляет новый материал, который меняет issue scope:

1. Сохранить новый material как новый `input_id` или attachment к существующему input packet.
2. Обновить linked issue `reason.md` или registry notes.
3. Вернуть issue в нужную phase:
   - `requirements_reopen`, если меняются требования;
   - `solution_reopen`, если меняется план/contract;
   - `qa_reopen`, если появился materially important unknown.
4. Не считать approval прежним, если новый input меняет scope.

## Cleanup и retention

Inbox cleanup не выполняется через ad-hoc delete. Порядок:

1. Открыть [issue_retention_protocol.md](../Protocols/service_protocols/issue_retention_protocol.md).
2. Найти linked issue.
3. Проверить status linked issue.
4. Проверить, что input не является единственным источником reason/requirements/contract.
5. Создать tombstone manifest или archive pointer.
6. Обновить page registry/persistence log, если Markdown-файлы изменились.

## Связанные файлы

- [New issue protocol](../Protocols/service_protocols/new_issue_protocol.md)
- [Issue retention protocol](../Protocols/service_protocols/issue_retention_protocol.md)
- [Service start protocol](../Protocols/service_protocols/service_start_protocol.md)
- [Issue registry](../Issues/issue_registry.md)
- [Issue registry JSONL](../Issues/issue_registry.jsonl)
- [Page registry](../State/page_registry.jsonl)
- [Persistence log](../State/persistence_log.jsonl)
