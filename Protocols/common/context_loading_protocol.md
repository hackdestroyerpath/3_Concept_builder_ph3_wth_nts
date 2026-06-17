# Протокол загрузки контекста

Parent: [Каталог протоколов](../catalog.md)  
Owner issue: `EXEC-004` / `OPT-003` / `CB-STAGE-03`  
Protocol ID: `common/context_loading`  
Источник истины: `Protocols/common/context_loading_protocol.md`  
Status: `available`  
Updated: `2026-06-17T21:27:32Z`

## Назначение

Этот протокол задаёт, какие файлы агент загружает при старте, продолжении, восстановлении фокуса и перед выбором следующего действия. Цель - получить минимальный достаточный контекст, а не читать весь репозиторий без причины.

Лидерская архитектура сохраняется: основной state остаётся читаемым Markdown в `State/service_state.md` и `State/execution_index.md`. Машинные поля focus/integrity используются как компактные companion markers и не заменяют Markdown-state.

## Preconditions

- Root [README](../../README.md) существует.
- Relevant state существует:
  - [State/service_state.md](../../State/service_state.md) для `Service Mode`;
  - [State/execution_index.md](../../State/execution_index.md) для `Execution Mode`.
- [State/page_registry.jsonl](../../State/page_registry.jsonl) существует.
- [Protocols/catalog.md](../catalog.md) существует.
- Перед утверждением persistence/readback статуса применён текущий persistence protocol.

## Входы

| Вход | Обязательность | Назначение |
|---|---|---|
| `mode` | required or inferred | `service`, `execution` или `common` |
| `user_trigger` | required | команда пользователя или причина старта |
| `active_state_path` | required | путь к state текущего режима |
| `active_issue_id` | optional | заполняется после создания или выбора issue-модели |
| `active_concept_slug` | optional | только для `Execution Mode` |
| `reload_reason` | optional | `startup`, `resume`, `focus_loss`, `state_conflict`, `user_request`, `readback`, `validation`, `repair` |

## Порядок выполнения

1. **Entry read**: открыть [../../README.md](../../README.md).
2. **Mode state read**:
   - `Service Mode`: открыть [../../State/service_state.md](../../State/service_state.md);
   - `Execution Mode`: открыть [../../State/execution_index.md](../../State/execution_index.md).
3. **State marker capture**: зафиксировать `state_revision_loaded`, `state_updated_at_loaded`, `state_integrity_marker` и `state_readback_ref`, если они доступны из GitHub readback, blob SHA, commit SHA или metadata state.
4. **Registry check**: открыть [../../State/page_registry.jsonl](../../State/page_registry.jsonl) и проверить, что target Markdown-файлы имеют parent/backlink или entry.
5. **Protocol selection**: открыть [../catalog.md](../catalog.md), выбрать самый локальный protocol по trigger, mode, status, phase и issue type.
6. **Focus packet**: собрать schema ниже и загрузить только active state, active issue summary, выбранные protocols, affected files и прямые dependencies.
7. **Context lift**: если данных недостаточно, зафиксировать недостающий факт, подняться на один уровень к parent summary, прочитать только нужный файл и вернуться в локальный scope.
8. **Response marker check**: перед user-facing или handoff response проверить, что loaded context, persistence status и next step совпадают с фактически прочитанными state/log/readback evidence.
9. **Drift check**: перед mutation или финальным статусом проверить, что loaded context всё ещё соответствует active state и next-step marker.

## Context budget

| Уровень | Что можно загрузить | Когда допустимо |
|---|---|---|
| `entry` | root README, active state, page registry, catalog | default |
| `focused` | active issue state/reason, selected protocol, affected files | active issue выбран |
| `expanded` | parent summary, dependency summary, linked issue summary | без этого нельзя проверить blocker или requirement |
| `full_scope` | весь service или весь concept scope | финальная проверка или approved refactor |
| `repository_wide` | широкий обход всего репозитория | запрещён по умолчанию; только с reason и validation need |

Эти уровни являются authoritative для лидера. Donor-схемы могут уточнять поля focus packet, но не заменяют `entry`, `focused`, `expanded`, `full_scope`, `repository_wide` и не превращают обычный старт в repository-wide scan.

## Focus packet schema

Focus packet - компактный runtime-снимок того, что агент реально загрузил и где продолжать работу. Он помогает восстановить задачу без чтения всего репозитория.

```yaml
context_bundle_id: string
mode: service|execution|common
state_file: path
state_revision_loaded: string|integer|null
state_updated_at_loaded: iso8601|null
state_integrity_marker: string|null
state_readback_ref: string|null
current_focus: string|null
current_entity_id: string|null
active_scope: string
active_issue_id: string|null
active_concept_slug: string|null
parent_anchor: string|null
active_protocols: []
allowed_context: []
blocked_context: []
source_files_loaded: []
affected_files: []
pending_user_action: null|needs_continue|needs_user_answers|needs_user_approval|repair_required|blocked
next_expected_step: string
reload_reason: startup|resume|focus_loss|state_conflict|user_request|readback|validation|repair
context_confidence: high|medium|low
health_signal:
  state_loaded: true|false
  catalog_loaded: true|false
  registry_loaded: true|false
  protocols_loaded: true|false
  pending_action_checked: true|false
  persistence_evidence_loaded: true|false
```

Минимальный packet не обязан иметь все optional IDs заполненными, если state явно говорит `none`. Если активная сущность заявлена, но `current_entity_id`, `active_issue_id` или `active_concept_slug` не восстановлены, `context_confidence` становится `low`.

## State integrity marker

State integrity marker - lightweight companion evidence, а не новая mandatory JSON-state система.

Разрешённые маркеры:

- `state_revision_loaded`: revision, timestamp, status marker или иной readable revision из Markdown state;
- `state_updated_at_loaded`: значение `Updated` из шапки state, если оно есть;
- `state_integrity_marker`: optional checksum, commit SHA, blob SHA, branch ref или combined marker;
- `state_readback_ref`: ссылка-описание на GitHub readback evidence, например `branch=<branch>; path=<path>; blob=<sha>`.

Правила:

1. Markdown state остаётся первичным источником состояния.
2. JSON/JSONL companion допустим только при явной операционной пользе: registry, manifest, log, catalog или compact machine marker.
3. Отсутствие checksum не блокирует MVP-работу, если state прочитан, marker честно равен `null`, а recovery не требует точного hash.
4. Mandatory SHA/hash вводится только для конкретного файла или protocol, который уже требует такой marker.
5. Нельзя мигрировать leader state wholesale в JSON или заменять читаемые Markdown state machine-only форматом.

## Response packet / marker discipline

User-facing и handoff response packet должен быть короткой проекцией загруженного state, а не самостоятельным источником фактов.

Минимальные поля, когда ответ меняет состояние, передаёт работу или просит продолжение:

```text
mode:
active_scope:
active_issue:
active_concept:
stage:
loaded_context:
context_confidence:
persistence_status:
next_step:
return_anchor:
```

Правила маркера:

1. `loaded_context` перечисляет только файлы, которые реально были прочитаны в этом ходе или подтверждены свежим readback.
2. `persistence_status` не использует `synced`, `committed`, `passed`, `ready` или финальные эквиваленты без persistence/readback evidence.
3. Если `context_confidence=low`, обычная execution/mutation останавливается и запускается recovery path.
4. `next_step` должен быть одним ближайшим шагом, а не всем roadmap.
5. `return_anchor` должен указывать существующий рабочий файл, state marker или registry entry.
6. Response packet не заменяет persistence log, validation report, issue registry или GitHub readback.

## Pending user action

`pending_user_action` показывает, можно ли продолжать без нового пользовательского решения.

| Значение | Поведение |
|---|---|
| `null` | Нет пользовательского ожидания; можно продолжать по ближайшему протоколу. |
| `needs_continue` | Ждать внешнюю команду `продолжай`; не делать новую mutation до неё. |
| `needs_user_answers` | Ждать конкретные ответы пользователя; простого `продолжай` недостаточно. |
| `needs_user_approval` | Не выполнять mutation, merge, export или irreversible cleanup до явного approval. |
| `repair_required` | Обычный workflow остановлен; сначала выбрать или выполнить recovery/repair path. |
| `blocked` | Продолжение запрещено до снятия blocker или user decision. |

Если state, packet и response marker расходятся по pending action, приоритет имеет актуальный state/readback. До сверки агент не должен утверждать, что ожидание закрыто.

## Drift / low-confidence recovery

Recovery запускается, если обнаружена одна из причин:

```text
missing_state
missing_catalog
missing_page_registry
registry_state_conflict
active_scope_unknown
active_issue_missing
active_concept_missing
stale_state_marker
loaded_context_mismatch
persistence_status_unknown
```

Recovery behavior:

1. остановить normal mutation и не писать target files по памяти;
2. выставить `context_confidence=low`;
3. загрузить только entry/state/registry/catalog/local protocol, достаточные для диагностики;
4. записать blocker или repair write set через persistence protocol, если восстановление требует изменения файлов;
5. продолжить обычный workflow только после recovery evidence или явного user decision.

Recovery не является разрешением читать весь репозиторий. Подъём выше `expanded` требует reason; `repository_wide` остаётся исключением для validation need или approved repair.

## Completion signal

Протокол завершён, когда агент может назвать:

- текущий `mode`;
- active state path;
- active issue или reason, почему issue ещё нет;
- active concept или reason, почему concept ещё нет;
- выбранный protocol или blocker;
- минимальный focus packet;
- state integrity marker или honest `null`;
- response marker fields, если ответ требует handoff/status;
- next-step marker.

## Failure behavior

Если relevant state, catalog или registry отсутствуют, агент не продолжает выполнение по памяти. Он создаёт `blocked_on_missing_state_or_catalog`, `context_confidence=low` или ограниченный package draft и фиксирует write set для восстановления.

Если loaded context расходится с state/log/readback, агент не заявляет финал и не выполняет normal mutation до recovery.

## Связанные файлы

- [Persistence protocol](persistence_protocol.md)
- [Service state](../../State/service_state.md)
- [Execution index](../../State/execution_index.md)
- [Page registry](../../State/page_registry.jsonl)
