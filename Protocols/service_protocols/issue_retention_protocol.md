# Протокол Issue Retention

Parent: [Каталог протоколов](../catalog.md)  
Owner issue: `USER-006` / `EXEC-005`  
Protocol ID: `service/issue_retention`  
Источник истины: `Protocols/service_protocols/issue_retention_protocol.md`  
Status: `available`  
Updated: `2026-06-05T09:52:51Z`

## Назначение

Этот протокол управляет хранением, архивированием, tombstone-свёрткой и допустимым удалением файлов `issue` и связанных `Inbox`-пакетов. Его цель — уменьшать рабочий шум без потери traceability.

Главное правило: cleanup не равен забыванию. `issue_id`, reason, decision, dependency refs, input refs и итоговые output refs должны оставаться проверяемыми даже после tombstone. Полное исчезновение истории запрещено, кроме временных и тяжёлых файлов, которые явно разрешены к удалению после tombstone-проверки.

## Политика по умолчанию

По умолчанию используется `batch cleanup`, а не immediate cleanup.

| Вариант | Когда допустим | Ограничение |
|---|---|---|
| `no_cleanup` | issue активен, blocked, deferred или нужен как dependency context | ничего не удалять и не перемещать |
| `archive_full` | issue closed/rejected и полная история ещё нужна | переместить или сохранить runtime artifacts в `_archive` |
| `tombstone_only` | полная история больше не нужна, но traceability нужна всегда | оставить минимальный tombstone и registry pointer |
| `delete_nonhistorical` | временные, тяжёлые или дублирующие attachments после tombstone | удалить только после сохранения manifest/hash/reason |
| `external_reference` | файл хранится вне репозитория | оставить ссылку, статус проверки и reason |

Immediate cleanup допустим только для явно временного файла, который не является source, reason, requirements, contract, output, validation, dependency evidence или user attachment с активной ссылкой.

## Когда использовать

| Trigger | Действие |
|---|---|
| Issue получил status `closed` | проверить archive-readiness и перенести историю в `_archive` при необходимости |
| Issue получил status `rejected` | сохранить decision reason, затем archive или tombstone |
| Нужно очистить закрытый issue от тяжёлых attachments | создать tombstone и удалить только разрешённые файлы |
| Пользователь просит удалить, очистить, архивировать или уплотнить issue | выполнить retention decision, не удалять без traceability |
| `Inbox/<input_id>/` связан только с закрытыми/rejected/tombstone issue | свернуть input packet до tombstone manifest или оставить archive reason |
| Dependency graph указывает на archived/tombstone issue | проверить, достаточно ли tombstone для dependents |
| Page registry показывает устаревший runtime Markdown после archive/move | обновить registry и backlinks |

## Обязательные входы

| Вход | Источник |
|---|---|
| Issue registry | [../../Issues/issue_registry.md](../../Issues/issue_registry.md) и [../../Issues/issue_registry.jsonl](../../Issues/issue_registry.jsonl) |
| Dependency graph | [../../Issues/dependency_graph.jsonl](../../Issues/dependency_graph.jsonl) |
| Archive entry point | [../../Issues/_archive/README.md](../../Issues/_archive/README.md) |
| Tombstone entry point | [../../Issues/_tombstones/README.md](../../Issues/_tombstones/README.md) |
| Inbox rules | [../../Inbox/README.md](../../Inbox/README.md) |
| Active focus / state | [../../State/service_state.md](../../State/service_state.md) и [existing_issue_protocol.md](existing_issue_protocol.md) |
| Dependency rules | [linked_issues_protocol.md](linked_issues_protocol.md) |
| Page registry | [../../State/page_registry.jsonl](../../State/page_registry.jsonl) |
| Persistence rules | [../common/persistence_protocol.md](../common/persistence_protocol.md) |
| Protocol catalog | [../catalog.md](../catalog.md) и [../catalog.jsonl](../catalog.jsonl) |

## Preconditions

1. Target issue существует в [../../Issues/issue_registry.jsonl](../../Issues/issue_registry.jsonl).
2. Status issue допускает retention-действие: `closed`, `rejected`, `archived`, `tombstone` или явный cleanup для временных файлов.
3. Для `active`, `approved`, `blocked`, `needs_discussion`, `creating` и `proposed` retention-действие запрещено, кроме read-only анализа.
4. Для `deferred` cleanup запрещён без отдельного user/system decision, потому что deferred остаётся backlog-записью.
5. Все active blocking dependencies из [../../Issues/dependency_graph.jsonl](../../Issues/dependency_graph.jsonl) проверены.
6. Сохранение tombstone или archive packet возможно в той же persistence transaction, что и обновление registry.
7. Для удаления любого файла есть cleanup reason, список affected paths и проверка отсутствия active refs.

## Retention states

`status` в registry остаётся lifecycle-статусом. Для более точной политики runtime issue может дополнительно использовать optional поля `retention_status`, `archive_path`, `tombstone_path`, `cleanup_reason`, `cleanup_at` и `deleted_paths`.

| `retention_status` | Значение |
|---|---|
| `active_runtime` | issue живёт в основной очереди, cleanup запрещён |
| `closed_full_history` | issue закрыт, но полный context ещё нужен |
| `archived_full_history` | полная история сохранена в `_archive` |
| `tombstone_summary` | полная история свёрнута до tombstone |
| `deleted_nonhistorical_files` | удалены только разрешённые временные/тяжёлые файлы |
| `external_reference_only` | исходные тяжёлые файлы хранятся вне репозитория, в tombstone есть ссылка/status |

## Разрешённые переходы

| From | To | Условие |
|---|---|---|
| `closed` | `archived` | validation/contract pass сохранён, active blockers отсутствуют или не требуют full runtime folder |
| `rejected` | `archived` | rejection reason и decision сохранены |
| `closed` / `rejected` | `tombstone` | tombstone содержит минимальную историю и active dependencies не требуют полного архива |
| `archived` | `tombstone` | archive больше не нужен для active dependents |
| `tombstone` | `deleted` | удаляются только nonhistorical files; tombstone, registry row и graph history остаются |
| `deferred` | `archived` | только после отдельного decision, что issue больше не является backlog-кандидатом |

Переходы `active -> archived`, `blocked -> tombstone` и `approved -> deleted` запрещены. Если пользователь просит такое действие, агент возвращает blocker и объясняет, какие зависимости или статусы мешают.

## Retention decision packet

Перед любым archive/tombstone/delete действием агент собирает короткий packet:

| Поле | Что записать |
|---|---|
| `issue_id` | target issue |
| `current_status` | status и phase до действия |
| `requested_action` | archive, tombstone, cleanup или delete_nonhistorical |
| `reason` | зачем действие нужно сейчас |
| `history_required_by` | active dependents, parent, children, linked issues или `none` |
| `input_refs` | `Inbox` refs и attachment refs |
| `artifact_refs` | reason/requirements/solution/contract/output/validation refs |
| `decision` | выполнить, отложить, заблокировать или запросить user decision |
| `write_set` | файлы, которые будут созданы, изменены, перемещены или удалены |

Packet сохраняется в runtime `state.md`, archive `decision_log.md` или tombstone-файле. Если issue registry-only и отдельной папки нет, packet отражается в registry `notes` и persistence log.

## Archive transaction

1. Проверить target issue в registry.
2. Проверить inbound/outbound dependencies через [../../Issues/dependency_graph.jsonl](../../Issues/dependency_graph.jsonl).
3. Проверить parent/children: parent нельзя закрывать и архивировать как завершённый, пока children не закрыты, не rejected или не deferred с reason.
4. Собрать retention decision packet.
5. Если runtime issue folder существует, перенести или скопировать его в `Issues/_archive/<issue_id>/`.
6. Убедиться, что archive entry содержит как минимум `state.md`, `reason.md` или их краткие equivalents, `decision_log.md`, output/validation refs, если они существовали.
7. Если runtime issue folder не существует, не создавать пустой `Issues/_archive/<issue_id>/`. Registry-only bootstrap issue остаётся registry-only.
8. Обновить [../../Issues/issue_registry.jsonl](../../Issues/issue_registry.jsonl): `status = archived`, `retention_status = archived_full_history`, `archive_path`, `updated_at`, `next_action`.
9. Обновить [../../Issues/issue_registry.md](../../Issues/issue_registry.md) summary, если меняется bootstrap snapshot или lifecycle documentation.
10. Обновить [../../State/page_registry.jsonl](../../State/page_registry.jsonl) для всех Markdown-файлов, которые были созданы, перемещены или сняты с active navigation.
11. Добавить запись в [../../State/persistence_log.jsonl](../../State/persistence_log.jsonl).

Archive не обязан быть компактным. Его задача — сохранить проверяемую историю до следующего batch cleanup.

## Tombstone transaction

1. Проверить, что issue имеет status `closed`, `rejected` или `archived`, либо есть отдельное решение о tombstone для registry-only записи.
2. Проверить, что active dependents не требуют full archive artifacts.
3. Создать `Issues/_tombstones/<issue_id>.md` или обновить существующий tombstone.
4. Tombstone должен содержать минимальные поля:
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
6. Сохранить historical edges в [../../Issues/dependency_graph.jsonl](../../Issues/dependency_graph.jsonl). Edge не удаляется только потому, что issue свёрнут.
7. Если archive folder после tombstone больше не нужен, удалить только после отдельного шага `delete_nonhistorical` и записи `deleted_paths`.
8. Обновить [../../State/page_registry.jsonl](../../State/page_registry.jsonl) для tombstone Markdown.
9. Добавить persistence entry.

Tombstone — минимальный исторический объект. Он должен быть достаточно информативным, чтобы агент будущего чата понял, почему ID нельзя переиспользовать и какие решения уже были приняты.

## Deletion transaction

Удаление допустимо только после tombstone transaction и только для файлов, которые не являются обязательной историей.

| Объект | Можно удалить? | Условие |
|---|---|---|
| `issue_registry.jsonl` row | нет | row остаётся всегда |
| dependency graph edge | нет | edge может быть помечен historical/superseded, но не исчезает без migration issue |
| tombstone file | нет | это минимальный след |
| `reason.md` | только после tombstone | reason summary должен быть в tombstone |
| `requirements.md`, `contract.md`, `validation_report.md` | только если tombstone содержит summary и refs | для спорных или audit-critical issue лучше оставить archive |
| `output/` | осторожно | итоговые refs должны сохраняться; удалять только тяжёлые attachments или дубликаты |
| temporary files | да | после `deleted_paths` и cleanup reason |
| binary/heavy attachments | да | если manifest/hash/external ref сохранены и active refs отсутствуют |

После deletion registry может иметь `status = deleted`, но это не означает исчезновение `issue_id`. Это означает, что разрешённые nonhistorical files удалены, а tombstone/registry/graph остались.

## Inbox cleanup

`Inbox` очищается отдельно от issue folder, но только через те же traceability gates.

1. Найти `Inbox/<input_id>/manifest.md` и все `linked_issue_ids`.
2. Если хотя бы один linked issue имеет status `creating`, `proposed`, `needs_discussion`, `approved`, `active`, `blocked`, `deferred`, `closed` или `archived`, input packet остаётся.
3. Если все linked issue имеют status `rejected`, `tombstone` или `deleted`, можно свернуть input packet до tombstone manifest.
4. Tombstone manifest в `Inbox/<input_id>/manifest.md` должен сохранить `input_id`, linked issue IDs, source summary, attachment refs/status, cleanup reason и timestamp.
5. Heavy attachments можно удалить только если они отражены в tombstone manifest или issue tombstone через filename, role, hash/checksum if available, storage status и reason.
6. `input_id` не переиспользуется после cleanup.
7. Если input содержит приватные, секретные или внешние материалы, cleanup decision должен явно сказать, что именно остаётся в репозитории.

Этот протокол не создаёт отдельную папку `Inbox/_tombstones/` по умолчанию. Минимальный след input остаётся в runtime packet manifest или в issue tombstone, чтобы не плодить новый верхний маршрут без отдельного решения.

## Registry update rules

При retention-действии registry обновляется в той же transaction:

| Поле | Правило |
|---|---|
| `status` | lifecycle status после действия |
| `phase` | обычно `null`, кроме active validation repair |
| `dependency_ready` | пересчитать с учётом archived/tombstone dependencies |
| `blocking_reason` | указать blocker, если archive/tombstone невозможен |
| `reason_path` / phase paths | обновить при move в archive или сохранить `null` для registry-only issue |
| `archive_path` | optional path к `Issues/_archive/<issue_id>/` |
| `tombstone_path` | optional path к `Issues/_tombstones/<issue_id>.md` |
| `retention_status` | точное состояние хранения |
| `next_action` | что должен сделать следующий агент |
| `updated_at` | timestamp transaction |

Если schema consumer не знает optional retention fields, он должен игнорировать их, но не удалять.

## Dependency handling

Перед archive/tombstone/delete агент проверяет три группы связей:

| Связь | Проверка |
|---|---|
| Parent/child | parent не считается завершённым, если child ещё active/blocked без explicit reason |
| Blocking dependency | dependent issue не должен потерять required artifact |
| Non-blocking link | tombstone должен сохранить related context summary |

Если archived/tombstone issue является dependency для active issue, dependent получает одно из решений:

- `dependency_ready = ready`, если tombstone/архив содержит нужный artifact/ref;
- `dependency_ready = stale`, если dependent должен перепроверить использованный artifact;
- `dependency_ready = blocked`, если full artifact удалён или недоступен;
- `dependency_ready = cycle_blocked`, если repair создаёт цикл.

Обновление связей выполняется по [linked_issues_protocol.md](linked_issues_protocol.md), а не вручную через одно поле registry.

## Failure behavior

| Сбой | Status | Действие |
|---|---|---|
| Issue отсутствует в registry | `blocked_on_missing_issue` | восстановить registry или отклонить cleanup request |
| Нет cleanup reason | `blocked_on_missing_retention_reason` | запросить конкретную причину или выбрать `no_cleanup` |
| Active dependency требует artifact | `blocked_on_active_dependency` | оставить archive/full history до снятия dependency |
| Tombstone не содержит required fields | `blocked_on_incomplete_tombstone` | не удалять archive/runtime folder |
| Page registry не обновлён после move/delete | `blocked_on_page_registry_mismatch` | выполнить repair transaction |
| Inbox packet связан с active issue | `blocked_on_active_input_refs` | не чистить input packet |
| Persistence недоступен | `blocked_on_persistence` | не заявлять cleanup выполненным |

## Completion signal

Протокол завершён, когда retention decision сохранён, registry отражает новый lifecycle/retention state, dependency graph проверен, page registry обновлён для Markdown-перемещений или tombstone, Inbox refs не потеряны, а [../../State/persistence_log.jsonl](../../State/persistence_log.jsonl) содержит запись transaction. Если любой gate не пройден, результатом является явный blocker, а не частично удалённая история.

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
- [Inbox](../../Inbox/README.md)
- [Service state](../../State/service_state.md)
- [Page registry](../../State/page_registry.jsonl)
- [Persistence protocol](../common/persistence_protocol.md)
- [Persistence log](../../State/persistence_log.jsonl)
