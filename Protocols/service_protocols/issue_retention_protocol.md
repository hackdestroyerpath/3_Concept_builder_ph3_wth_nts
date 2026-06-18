# Протокол Issue Retention

Parent: [Каталог протоколов](../catalog.md)  
Owner issue: `USER-006` / `EXEC-005`  
Protocol ID: `service/issue_retention`  
Источник истины: `Protocols/service_protocols/issue_retention_protocol.md`  
Status: `available`  
Updated: `2026-06-18T02:05:00Z`

## Назначение

Этот протокол управляет archive, tombstone, delete semantics и Inbox cleanup для service-level issue. Его цель — уменьшать рабочий шум без потери traceability.

Основное правило: cleanup не равен забыванию. `issue_id`, reason, decision, dependency refs, input refs, output refs и validation evidence должны оставаться проверяемыми даже после tombstone. Полное исчезновение истории запрещено, кроме temporary/nonhistorical files, явно разрешённых к удалению после tombstone check.

## Политика по умолчанию

По умолчанию используется `batch cleanup`, а не immediate cleanup.

| Вариант | Когда допустим | Ограничение |
|---|---|---|
| `no_cleanup` | issue active, approved, blocked, deferred или нужен как dependency context | ничего не удалять и не перемещать |
| `archive_full` | issue closed/rejected и full history ещё нужна | сохранить runtime artifacts в `_archive` |
| `tombstone_only` | full history больше не нужна, но traceability нужна всегда | оставить tombstone и registry pointer |
| `delete_nonhistorical` | temporary/heavy/duplicate attachments после tombstone | удалить только после manifest/hash/reason |
| `external_reference` | файл хранится вне repo | оставить ссылку, status проверки и reason |

Immediate cleanup допустим только для явно temporary file, который не является source, reason, requirements, contract, output, validation, dependency evidence или active user attachment.

## Когда использовать

| Trigger | Действие |
|---|---|
| Issue получил `closed` | проверить archive-readiness |
| Issue получил `rejected` | сохранить rejection reason, затем archive/tombstone decision |
| Нужно очистить closed issue от heavy attachments | создать tombstone/manifest, затем delete_nonhistorical |
| Пользователь просит удалить/очистить/архивировать | выполнить retention decision, не удалять без traceability |
| `Inbox/<input_id>/` связан только с closed/rejected/tombstone/deleted issue | свернуть input packet до tombstone manifest или archive reason |
| Dependency graph указывает на archived/tombstone issue | проверить, достаточно ли tombstone для dependents |
| Page registry показывает устаревший runtime Markdown | обновить registry/backlinks через retention transaction |

## Обязательные входы

| Вход | Источник |
|---|---|
| Issue registry | [../../Issues/issue_registry.md](../../Issues/issue_registry.md) и [../../Issues/issue_registry.jsonl](../../Issues/issue_registry.jsonl) |
| Dependency graph | [../../Issues/dependency_graph.jsonl](../../Issues/dependency_graph.jsonl) |
| Archive entry point | [../../Issues/_archive/README.md](../../Issues/_archive/README.md) |
| Tombstone entry point | [../../Issues/_tombstones/README.md](../../Issues/_tombstones/README.md) |
| Inbox rules | [../../Inbox/README.md](../../Inbox/README.md) |
| Active focus/state | [../../State/service_state.md](../../State/service_state.md) и [existing_issue_protocol.md](existing_issue_protocol.md) |
| Dependency rules | [linked_issues_protocol.md](linked_issues_protocol.md) |
| Page registry | [../../State/page_registry.jsonl](../../State/page_registry.jsonl) |
| Persistence rules | [../common/persistence_protocol.md](../common/persistence_protocol.md) |
| Catalog | [../catalog.md](../catalog.md) и [../catalog.jsonl](../catalog.jsonl) |

## Preconditions

1. Target issue существует в [../../Issues/issue_registry.jsonl](../../Issues/issue_registry.jsonl).
2. Status допускает retention action: `closed`, `rejected`, `archived`, `tombstone` или explicit cleanup for temporary files.
3. Для `active`, `approved`, `blocked`, `needs_discussion`, `creating` и `proposed` retention запрещён, кроме read-only analysis.
4. Для `deferred` cleanup запрещён без separate user/system decision, потому что deferred остаётся backlog candidate.
5. Active blocking dependencies проверены.
6. Tombstone/archive packet сохраняется в той же transaction, что registry update.
7. Для deletion есть cleanup reason, affected paths и proof отсутствия active refs.
8. Empty archive/tombstone/runtime folders не создаются. Registry-only issue остаётся registry-only, если нет actual retained artifact.

## Retention states

`status` в registry остаётся lifecycle status. Optional retention fields уточняют cleanup policy: `retention_status`, `archive_path`, `tombstone_path`, `cleanup_reason`, `cleanup_at`, `deleted_paths`.

| `retention_status` | Значение |
|---|---|
| `active_runtime` | issue живёт в основной очереди, cleanup запрещён |
| `closed_full_history` | issue закрыт, но full context ещё нужен |
| `archived_full_history` | full history сохранена в `_archive` |
| `tombstone_summary` | history свёрнута до tombstone |
| `deleted_nonhistorical_files` | удалены только temporary/heavy duplicate files |
| `external_reference_only` | heavy files outside repo, tombstone содержит ref/status |

## Разрешённые переходы

| From | To | Условие |
|---|---|---|
| `closed` | `archived` | validation/contract pass сохранён, blockers не требуют active folder |
| `rejected` | `archived` | rejection reason и decision сохранены |
| `closed` / `rejected` | `tombstone` | tombstone содержит minimum history; dependents не требуют full archive |
| `archived` | `tombstone` | archive больше не нужен active dependents |
| `tombstone` | `deleted` | удаляются только nonhistorical files; tombstone/registry/graph remain |
| `deferred` | `archived` | только после decision, что issue больше не backlog candidate |

`active -> archived`, `blocked -> tombstone`, `approved -> deleted` запрещены. Если пользователь просит такое действие, agent возвращает blocker и условия, которые мешают.

## Retention decision packet

Перед archive/tombstone/delete agent собирает packet:

| Поле | Что записать |
|---|---|
| `issue_id` | target issue |
| `current_status` | status/phase до действия |
| `requested_action` | archive, tombstone, cleanup, delete_nonhistorical |
| `reason` | зачем действие нужно сейчас |
| `history_required_by` | active dependents, parent, children, linked issues или `none` |
| `input_refs` | Inbox refs and attachment refs |
| `artifact_refs` | reason/requirements/solution/contract/output/validation refs |
| `dependency_refs` | direct graph edges and dependent conditions |
| `decision` | execute, defer, block или ask user decision |
| `write_set` | files created/changed/moved/deleted |

Packet сохраняется в runtime `state.md`, archive `decision_log.md` или tombstone file. Если issue registry-only, packet отражается в registry notes и persistence log.

## Archive transaction

1. Проверить target issue в registry.
2. Проверить inbound/outbound dependencies через [../../Issues/dependency_graph.jsonl](../../Issues/dependency_graph.jsonl).
3. Проверить parent/children: parent нельзя архивировать как completed, пока required children не closed/rejected/deferred with reason.
4. Собрать retention decision packet.
5. Если runtime issue folder существует, перенести или скопировать его в `Issues/_archive/<issue_id>/`.
6. Archive entry содержит at least `state.md`, `reason.md` или equivalents, `decision_log.md`, output/validation refs если они были.
7. Если runtime issue folder не существует, не создавать пустой `Issues/_archive/<issue_id>/`.
8. Обновить registry JSONL: `status = archived`, `retention_status = archived_full_history`, `archive_path`, `updated_at`, `next_action`.
9. Обновить [../../Issues/issue_registry.md](../../Issues/issue_registry.md), если human snapshot меняется.
10. Обновить [../../State/page_registry.jsonl](../../State/page_registry.jsonl) для Markdown moved/created/deactivated.
11. Добавить [../../State/persistence_log.jsonl](../../State/persistence_log.jsonl) entry.

Archive сохраняет проверяемую историю до следующего batch cleanup.

## Tombstone transaction

1. Проверить status `closed`, `rejected`, `archived`, либо отдельное decision о tombstone для registry-only entry.
2. Проверить, что active dependents не требуют full archive artifacts.
3. Создать `Issues/_tombstones/<issue_id>.md` или обновить existing tombstone.
4. Tombstone содержит minimum fields:
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
5. Обновить registry: `status = tombstone`, `retention_status = tombstone_summary`, `tombstone_path`, `updated_at`, `next_action`.
6. Сохранить historical edges в graph. Edge не удаляется из-за tombstone.
7. Archive folder после tombstone удаляется только отдельным `delete_nonhistorical` step и `deleted_paths` record.
8. Обновить page registry для tombstone Markdown.
9. Добавить persistence entry.

Tombstone должен быть достаточен, чтобы future agent понял, почему ID нельзя переиспользовать и какие decisions уже приняты.

## Deletion transaction

Deletion допустим только после tombstone transaction и только для nonhistorical files.

| Объект | Можно удалить? | Условие |
|---|---|---|
| `issue_registry.jsonl` row | нет | row remains always |
| dependency graph edge | нет | edge can be historical/superseded, but not disappear silently |
| tombstone file | нет | minimum trace |
| `reason.md` | только после tombstone | reason summary must remain in tombstone |
| `requirements.md`, `contract.md`, validation | только если tombstone contains summary/refs | audit-critical issues should keep archive |
| `output/` | cautiously | final refs must remain; delete only heavy duplicates/attachments |
| temporary files | да | after `deleted_paths` and cleanup reason |
| binary/heavy attachments | да | if manifest/hash/external ref saved and no active refs |

После deletion registry может иметь `status = deleted`, но `issue_id` не исчезает. Это значит, что разрешённые nonhistorical files удалены, а tombstone/registry/graph остались.

## Inbox cleanup

Inbox cleanup выполняется через те же traceability gates:

1. Найти `Inbox/<input_id>/manifest.md` и `linked_issue_ids`.
2. Если хотя бы один linked issue active/approved/blocked/deferred, cleanup запрещён.
3. Если все linked issue closed/rejected/tombstone/deleted и input больше не нужен, создать/обновить tombstone manifest.
4. Heavy attachments удаляются только после manifest/hash/external ref.
5. Registry/graph refs на input сохраняются или указывают на tombstone/external ref.
6. Page registry и persistence log обновляются при Markdown changes.

## Dependency impact

Перед retention agent проверяет direct dependents:

| Dependency state | Rule |
|---|---|
| Dependent active and requires full artifact | archive/tombstone/delete blocked |
| Dependent only needs summary | tombstone allowed if summary contains required fields |
| Edge stale/blocked | retention action не скрывает blocker; graph keeps evidence |
| Dependency rejected/deleted | dependent gets blocker until solution/requirements recheck |

Retention не снимает dependency blocker автоматически.

## Failure behavior

| Сбой | Статус | Действие |
|---|---|---|
| Issue active/approved/blocked | `retention_not_allowed_for_active_issue` | read-only summary and blocker |
| Нет cleanup reason | `blocked_on_missing_cleanup_reason` | request reason / decision |
| Active dependent needs artifact | `blocked_on_active_dependency_ref` | keep full archive/no_cleanup |
| Tombstone insufficient | `blocked_on_tombstone_coverage` | add summary/refs before deletion |
| Page registry cannot be updated | `blocked_on_page_registry_update` | do not move/delete Markdown |
| Persistence unavailable | `blocked_on_persistence` | do not claim archive/tombstone/delete committed |

## Completion signal

Протокол завершён, когда retention decision packet saved, registry/graph/page registry updated as required, tombstone/archive/delete action committed, or work stopped with honest blocker. Inbox cleanup is complete only when linked issues, manifest/tombstone and deleted_paths evidence agree.

## Связанные файлы

- [Catalog](../catalog.md)
- [Catalog JSONL](../catalog.jsonl)
- [Existing issue protocol](existing_issue_protocol.md)
- [Linked Issues protocol](linked_issues_protocol.md)
- [Issue registry](../../Issues/issue_registry.md)
- [Issue registry JSONL](../../Issues/issue_registry.jsonl)
- [Dependency graph](../../Issues/dependency_graph.jsonl)
- [Issue archive](../../Issues/_archive/README.md)
- [Issue tombstones](../../Issues/_tombstones/README.md)
- [Inbox rules](../../Inbox/README.md)
- [Service state](../../State/service_state.md)
- [Page registry](../../State/page_registry.jsonl)
- [Persistence protocol](../common/persistence_protocol.md)
