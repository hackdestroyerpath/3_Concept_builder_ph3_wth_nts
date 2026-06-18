# Протокол Linked Issues

Parent: [Каталог протоколов](../catalog.md)  
Owner issue: `EXEC-010`  
Protocol ID: `service/linked_issues`  
Источник истины: `Protocols/service_protocols/linked_issues_protocol.md`  
Status: `available`  
Updated: `2026-06-18T12:50:00Z`

## Назначение

Этот протокол управляет связями между `issue`, которые не образуют parent/child tree. Связь нужна, когда один issue зависит от результата, решения, файла, output, contract, validation или контекста другого issue.

Источник истины для root service scope — [../../Issues/dependency_graph.jsonl](../../Issues/dependency_graph.jsonl). Registry хранит только mirror-поля: `linked_issue_ids`, `dependency_refs`, `dependency_ready`, `blocking_reason`. Полная логика edge живёт в graph.

Parent/child decomposition выполняется через [complex_issue_protocol.md](complex_issue_protocol.md). Linked edge не должен маскировать complex issue, а complex issue не должен притворяться простой ссылкой, если есть real blocking dependencies.

## Когда использовать

| Состояние | Действие |
|---|---|
| Issue зависит от output другого issue | создать blocking edge `requires_output` |
| Issue зависит от attachment/source другого issue | создать blocking edge `requires_attachment` |
| Issue зависит от approved contract другого issue | создать blocking edge `requires_contract` |
| Issue можно закрыть только после validation другого issue | создать blocking edge `requires_validation` |
| Issue использует другой issue только как context | создать non-blocking edge `informs` |
| Issue пересекаются или дублируются | создать `duplicates_or_overlaps` и запросить merge/split/defer decision |
| Requirements/solution/output конфликтуют | создать `conflicts_with` и block closure до resolution |
| Source artifact изменился после использования | отметить dependent as `stale` |

## Обязательные входы

| Вход | Источник |
|---|---|
| Source/target issue rows | [../../Issues/issue_registry.jsonl](../../Issues/issue_registry.jsonl) |
| Current graph | [../../Issues/dependency_graph.jsonl](../../Issues/dependency_graph.jsonl) |
| Active focus | [existing_issue_protocol.md](existing_issue_protocol.md) и [../../State/service_state.md](../../State/service_state.md) |
| Requirements/solution/output pointers | registry paths и issue artifacts, если существуют |
| Parent/child summary | [complex_issue_protocol.md](complex_issue_protocol.md), если связь внутри complex scope |
| Page registry | [../../State/page_registry.jsonl](../../State/page_registry.jsonl) |
| Persistence rules | [../common/persistence_protocol.md](../common/persistence_protocol.md) |
| Catalog | [../catalog.md](../catalog.md) и [../catalog.jsonl](../catalog.jsonl) |

## Preconditions

1. Оба issue существуют в registry текущего scope или создаются одной controlled transaction.
2. Reason связи понятен: какой artifact, decision или state требуется и почему.
3. Duplicate/overlap risk проверен перед созданием edge.
4. Blocking edge не создаёт cycle в текущем graph.
5. Registry и graph можно обновить в одной persistence transaction.
6. Если связь меняет scope или блокирует active issue, user decision требуется до execution.
7. Runtime issue folders не создаются для одной лишь связи; state update выполняется только если issue folder уже существует или phase требует state artifact.

## Direction convention

В root graph используется convention:

| Поле | Значение |
|---|---|
| `source_issue_id` | dependency issue, которое предоставляет prerequisite |
| `target_issue_id` | dependent issue, которое blocked или informed |
| `dependent_issue_id` | optional mirror для читабельности |
| `dependency_issue_id` | optional mirror для читабельности |

Если future migration поменяет convention, graph metadata должен быть обновлён вместе со всеми edge readers. В одном graph нельзя смешивать две direction semantics.

## Минимальная запись edge

Новые runtime edges должны быть богаче bootstrap-записей:

```json
{
  "record_type": "edge",
  "edge_id": "EDGE-<dependency>-TO-<dependent>",
  "source_issue_id": "<dependency_issue_id>",
  "target_issue_id": "<dependent_issue_id>",
  "dependent_issue_id": "<dependent_issue_id>",
  "dependency_issue_id": "<dependency_issue_id>",
  "relation_type": "requires_output | requires_attachment | requires_contract | requires_validation | informs | duplicates_or_overlaps | conflicts_with",
  "relation": "<legacy mirror if needed>",
  "blocking": true,
  "required_artifacts": ["output/report.md"],
  "readiness": "ready | blocked | stale | cycle_blocked | satisfied_for_draft | unsatisfied",
  "status": "ready | blocked | stale | cycle_blocked | satisfied_for_draft | unsatisfied",
  "stale_reason": null,
  "reason": "почему dependent issue не может двигаться без dependency",
  "created_at": "<timestamp>",
  "updated_at": "<timestamp>"
}
```

Bootstrap edges могут сохранять legacy `relation` и `status`, но protocol при чтении обязан нормализовать их перед lifecycle decision.

### Legacy normalization

| Legacy input | Normalized relation/readiness | Lifecycle meaning |
|---|---|---|
| `relation = blocks_until_ready`, `status = satisfied` | relation remains blocking; `readiness = ready` | prerequisite подтверждён, dependent может продолжать |
| `status = satisfied_for_draft` | `readiness = satisfied_for_draft` | разрешены analysis, requirements и explicitly scoped implementation draft; runtime execution approval, validation и closure ещё заблокированы |
| `status = unsatisfied` | `readiness = unsatisfied` | prerequisite отсутствует, dependent blocked |
| missing `relation_type` with known legacy `relation` | map to equivalent specific relation when evidence exists; otherwise keep legacy relation and record normalization note | semantics не выдумываются без reason/artifact evidence |

Нормализация является read-time decision и не требует массовой migration bootstrap graph. Если edge изменяется по существу, write transaction сохраняет normalized fields, не удаляя нужные legacy mirrors.

## Relation types

| `relation_type` | Когда использовать | Blocking по умолчанию |
|---|---|---|
| `requires_output` | dependent использует output dependency issue | да |
| `requires_attachment` | dependent использует attachment/source dependency issue | да |
| `requires_contract` | dependent требует approved contract dependency issue | да |
| `requires_validation` | dependent можно продолжать только после validation dependency issue | да |
| `informs` | dependency полезен как context reference | нет |
| `duplicates_or_overlaps` | нужен merge/split/defer decision | да до решения |
| `conflicts_with` | requirements/solution/outputs конфликтуют | да до resolution |

Legacy `blocks_until_ready` допустима для bootstrap implementation edges. Новые runtime edges используют конкретные relation types.

## Readiness rule

| Readiness | Значение | Что делать |
|---|---|---|
| `ready` | required artifact/state доступен и подходит dependent issue | dependent может продолжать |
| `blocked` | dependency ещё не дала нужный artifact/state | dependent не идёт в execution/validation/closed |
| `stale` | dependency изменилась после использования dependent issue | dependent возвращается к affected requirements/contract check |
| `cycle_blocked` | blocking edge создал бы cycle | active edge не сохраняется; нужен decision/repair |
| `satisfied_for_draft` | bootstrap prerequisite достаточно закрыта только для явно ограниченного draft | разрешить draft work; не разрешать runtime execution approval, validation или closure до `ready`/legacy `satisfied` |
| `unsatisfied` | prerequisite не реализован | dependent remains blocked |

QA и requirements dependent issue могут продолжаться, если им не нужен отсутствующий artifact. Runtime execution approval, validation и closure запрещены при active blocking edge со readiness `blocked`, `stale`, `cycle_blocked`, `satisfied_for_draft` или `unsatisfied`. Исключение для `satisfied_for_draft` ограничено bootstrap implementation draft и должно быть явно отражено в scope/validation notes.

## Создание связи

1. **Выбрать direction**: определить dependency/source и dependent/target.
2. **Определить relation**: выбрать конкретный `relation_type`, blocking flag и required artifacts.
3. **Проверить duplicate**: existing active edge с тем же reason не дублируется.
4. **Проверить overlap**: если scope совпадает, route к duplicate decision, а не создавать параллельный issue.
5. **Проверить cycle**: смоделировать candidate blocking edge.
6. **Если cycle найден**: не сохранять active blocking edge; dependent получает `cycle_blocked` или `needs_discussion`.
7. **Записать edge**: добавить строку в graph, обновить metadata `edge_count`, `cycle_check`, `updated_at`.
8. **Обновить registry**: `dependency_refs`, `linked_issue_ids`, `dependency_ready`, `blocking_reason`.
9. **Обновить state**: если runtime `state.md` существует, показать blocker, required artifact, safe actions и next step.
10. **Сохранить transaction**: добавить [../../State/persistence_log.jsonl](../../State/persistence_log.jsonl) entry.

Edge не считается действующим, пока graph и registry mirror не сохранены.

## Cycle detection

Cycle check выполняется только по active blocking edges текущего scope. Non-blocking `informs` не создаёт blocking cycle.

Algorithm:

1. Построить directed graph по active blocking edges.
2. Добавить candidate edge dependency -> dependent.
3. Проверить, появляется ли path от dependent обратно к dependency.
4. Если path есть, candidate edge не записывается как active blocking.
5. Сохранить blocker/discussion marker для dependent.
6. Предложить repair: убрать связь, поменять direction, сделать `informs`, объединить issue, создать parent complex issue через [complex_issue_protocol.md](complex_issue_protocol.md).

## Stale propagation

Если dependency issue изменил requirements, solution, contract, output или validation после того, как dependent использовал его result:

1. Найти direct dependents через graph.
2. Для blocking dependents поставить `dependency_ready = stale`.
3. В `blocking_reason` указать changed artifact и required recheck.
4. Не закрывать dependent issue до повторной проверки affected requirements, solution или contract.
5. Если dependency closed with notes, dependent принимает notes как risk или blocker.
6. Если dependency rejected/deferred/tombstone, blocking dependents получают `blocked` до пересмотра solution.

Propagation не требует repository-wide context. Достаточно edges selected issue, direct dependencies и parent summary.

## Blocked-status handling

Если dependency блокирует issue, runtime `state.md` dependent issue должен содержать:

```text
## Dependency blocker

Blocked by: <dependency_issue_id> via <edge_id>
Required artifact/state: <...>
Safe actions before unblock: <...>
Next step: wait / switch to dependency / change edge / ask user decision
```

Registry mirror:

- `dependency_refs`: edge IDs;
- `dependency_ready`: `blocked`, `stale` или `cycle_blocked`;
- `blocking_reason`: short unblock condition.

## Repair и удаление связи

| Ситуация | Действие |
|---|---|
| Dependency стала ready | обновить edge readiness/status и dependent mirror |
| Edge direction ошибочный | создать corrected edge и пометить old edge superseded/rejected, если policy есть |
| Связь стала non-blocking | сменить relation to `informs`, `blocking = false`, снять blocker |
| Issue объединяются | создать merge decision в affected issue state/registry |
| Issue разделяются | использовать [complex_issue_protocol.md](complex_issue_protocol.md) или new linked issue |
| Target deleted/tombstone | dependent получает blocker до пересмотра |

Полное удаление edge без следа запрещено, если на него уже ссылался registry или output. Минимум должен остаться reason в graph или affected issue state.

## Failure behavior

| Сбой | Статус | Действие |
|---|---|---|
| Issue отсутствует в registry | `blocked_on_missing_issue` | создать/восстановить issue или отклонить связь |
| Relation reason слабый | `needs_dependency_reason` | запросить concrete artifact/state |
| Duplicate edge или duplicate issue risk | `needs_dedup_decision` | merge/split/link/defer decision |
| Cycle detected | `cycle_blocked` | не активировать edge; предложить repair |
| Required artifact отсутствует | `blocked_on_dependency_artifact` | block dependent или allow QA/requirements-only work |
| Graph/status расходится с registry | `blocked_on_graph_registry_mismatch` | repair transaction |
| Persistence недоступен | `blocked_on_persistence` | не считать edge созданным |

## Completion signal

Протокол завершён, когда dependency edge сохранён или отклонён с reason, registry mirror обновлён, affected issue state отражает blocker/readiness, а persistence transaction записана. Если edge blocked или stale, next routing показывает dependent issue как неготовый к execution/validation/closure.

## Связанные файлы

- [Catalog](../catalog.md)
- [Catalog JSONL](../catalog.jsonl)
- [Existing issue protocol](existing_issue_protocol.md)
- [Complex Issue protocol](complex_issue_protocol.md)
- [Solution / Contract / Output protocol](solution_contract_output_protocol.md)
- [Issue Retention protocol](issue_retention_protocol.md)
- [Issue registry](../../Issues/issue_registry.md)
- [Issue registry JSONL](../../Issues/issue_registry.jsonl)
- [Dependency graph](../../Issues/dependency_graph.jsonl)
- [Service state](../../State/service_state.md)
- [Page registry](../../State/page_registry.jsonl)
- [Persistence protocol](../common/persistence_protocol.md)
