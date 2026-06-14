# Протокол Linked Issues

Parent: [Каталог протоколов](../catalog.md)  
Owner issue: `EXEC-010`  
Protocol ID: `service/linked_issues`  
Источник истины: `Protocols/service_protocols/linked_issues_protocol.md`  
Status: `available`  
Updated: `2026-06-05T09:52:44Z`

## Назначение

Этот протокол управляет связями между `issue`, которые не образуют parent/child-дерево. Связь нужна, когда один issue зависит от результата, решения, файла, output, contract, validation или контекста другого issue.

Источник истины для root service scope — [../../Issues/dependency_graph.jsonl](../../Issues/dependency_graph.jsonl). Registry хранит только краткие поля `linked_issue_ids`, `dependency_refs`, `dependency_ready` и `blocking_reason`; полная логика связи живёт в graph.

Для parent/child-разложения используется [complex_issue_protocol.md](complex_issue_protocol.md). Linked issue не должен маскировать complex decomposition, а complex issue не должен притворяться простой ссылкой, если есть настоящие blocking dependencies.

## Когда использовать

| Состояние | Действие |
|---|---|
| Issue зависит от output другого issue | создать blocking edge `requires_output` |
| Issue зависит от attachment или source material другого issue | создать blocking edge `requires_attachment` |
| Issue зависит от approved contract другого issue | создать blocking edge `requires_contract` |
| Issue можно закрыть только после validation другого issue | создать blocking edge `requires_validation` |
| Issue полезен как контекст, но не блокирует выполнение | создать non-blocking edge `informs` |
| Два issue дублируют или пересекают scope | создать `duplicates_or_overlaps` и запросить merge/split decision |
| Requirements или solution конфликтуют | создать `conflicts_with` и заблокировать closure до решения |
| Target изменился после использования source | отметить dependent issue как `stale` |

## Обязательные входы

| Вход | Источник |
|---|---|
| Source / target issue entries | [../../Issues/issue_registry.jsonl](../../Issues/issue_registry.jsonl) |
| Current graph | [../../Issues/dependency_graph.jsonl](../../Issues/dependency_graph.jsonl) |
| Active focus | [existing_issue_protocol.md](existing_issue_protocol.md) и [../../State/service_state.md](../../State/service_state.md) |
| Requirements / solution / output pointers | registry paths и issue artifacts, если они существуют |
| Parent/child summary | [complex_issue_protocol.md](complex_issue_protocol.md), если связь внутри complex scope |
| Page registry | [../../State/page_registry.jsonl](../../State/page_registry.jsonl) |
| Persistence rules | [../common/persistence_protocol.md](../common/persistence_protocol.md) |
| Protocol catalog | [../catalog.md](../catalog.md) и [../catalog.jsonl](../catalog.jsonl) |

## Preconditions

1. Оба issue существуют в registry текущего scope или создаются одной controlled transaction.
2. Reason связи понятен: какой artifact, decision или state требуется и почему.
3. Агент проверил duplicate/overlap risk перед созданием нового edge.
4. Blocking edge не создаёт цикл в текущем graph.
5. Registry и graph можно обновить в одной persistence transaction.
6. Если связь меняет scope или блокирует active issue, пользовательское решение требуется до execution.

## Направление edge

В текущем root graph используется convention: `source_issue_id` — issue, который предоставляет prerequisite или required artifact; `target_issue_id` — issue, который зависит от source. Metadata graph прямо фиксирует: edge идёт от dependency/source к dependent/target.

Чтобы runtime edge не был двусмысленным, новые записи должны также указывать смысл связи в `reason` и, при необходимости, добавлять mirror-поля:

| Поле | Значение |
|---|---|
| `source_issue_id` | issue, от которого зависит другой issue в текущей graph convention |
| `target_issue_id` | issue, который blocked или informed |
| `dependent_issue_id` | optional mirror: issue, который зависит |
| `dependency_issue_id` | optional mirror: issue, который предоставляет dependency |

Если future migration поменяет convention, graph metadata должен быть обновлён вместе со всеми edge readers. До этого агент не смешивает две семантики в одном graph.

## Минимальная запись edge

Новые runtime edges должны быть богаче bootstrap-записей. Минимальная нормализованная запись:

```json
{
  "record_type": "edge",
  "edge_id": "EDGE-<dependency>-TO-<dependent>",
  "source_issue_id": "<dependency_issue_id>",
  "target_issue_id": "<dependent_issue_id>",
  "relation_type": "requires_output | requires_attachment | requires_contract | requires_validation | informs | duplicates_or_overlaps | conflicts_with",
  "relation": "<legacy mirror if needed>",
  "blocking": true,
  "required_artifacts": ["output/report.md"],
  "readiness": "ready | blocked | stale | cycle_blocked",
  "status": "ready | blocked | stale | cycle_blocked | satisfied_for_draft | unsatisfied",
  "stale_reason": null,
  "reason": "почему dependent issue не может двигаться без dependency",
  "created_at": "<timestamp>",
  "updated_at": "<timestamp>"
}
```

Bootstrap edges могут сохранять старые поля `relation` и `status`, но linked issue protocol при чтении нормализует их в `relation_type` и `readiness` для принятия решения.

## Типы связей

| `relation_type` | Когда использовать | Blocking по умолчанию |
|---|---|---|
| `requires_output` | dependent использует output dependency issue | да |
| `requires_attachment` | dependent использует attachment/source file dependency issue | да |
| `requires_contract` | dependent требует approved contract dependency issue | да |
| `requires_validation` | dependent можно продолжать только после validation dependency issue | да |
| `informs` | dependency полезен как context reference, но не блокирует выполнение | нет |
| `duplicates_or_overlaps` | issue пересекаются и нужен merge/split/defer decision | да до решения |
| `conflicts_with` | requirements, solution или outputs конфликтуют | да до разрешения |

Relation `blocks_until_ready` допустима для bootstrap implementation edges. Для новых runtime issue предпочтительны конкретные relation types выше.

## Readiness rule

| Readiness | Значение | Что делать |
|---|---|---|
| `ready` | required artifact/state доступен и подходит dependent issue | dependent может продолжать |
| `blocked` | dependency ещё не дала нужный artifact/state | dependent не идёт в execution/validation/closed |
| `stale` | dependency изменилась после использования dependent issue | dependent возвращается к affected requirements/contract check |
| `cycle_blocked` | blocking edge создал бы цикл | edge не активируется; нужен user/parent decision |
| `satisfied_for_draft` | bootstrap dependency достаточно закрыта для draft-реализации | можно двигать implementation draft, но финальная validation ещё нужна |
| `unsatisfied` | prerequisite ещё не реализован | dependent остаётся blocked |

QA и requirements dependent issue могут продолжаться, если им не нужен отсутствующий artifact. Execution, validation и closure запрещены при active blocking edge со status `blocked`, `stale`, `cycle_blocked` или `unsatisfied`.

## Создание связи

1. **Выбрать direction**: определить dependency issue и dependent issue по текущей graph convention.
2. **Определить relation**: выбрать конкретный `relation_type` и required artifacts.
3. **Проверить duplicate**: убедиться, что edge не повторяет существующую active связь.
4. **Проверить cycle**: смоделировать добавление blocking edge в graph.
5. **Если cycle найден**: не сохранять active blocking edge; dependent получает `dependency_ready = cycle_blocked` или `needs_discussion`.
6. **Записать edge**: добавить строку в [../../Issues/dependency_graph.jsonl](../../Issues/dependency_graph.jsonl), обновить metadata `edge_count`, `cycle_check`, `updated_at`.
7. **Обновить registry**: добавить `dependency_refs`, `linked_issue_ids`, итог `dependency_ready`, `blocking_reason`.
8. **Обновить state**: если runtime `state.md` существует, показать `blocked_by`, required artifact, safe actions и next step.
9. **Сохранить transaction**: добавить запись в [../../State/persistence_log.jsonl](../../State/persistence_log.jsonl).

Edge не считается действующим, пока graph и registry не сохранены. Устная зависимость в чате не блокирует lifecycle, потому что чат не является graph.

## Cycle detection

Cycle check выполняется только по blocking edges текущего scope. Non-blocking `informs` не создаёт blocking cycle.

Алгоритм:

1. Построить directed graph по активным blocking edges.
2. Добавить candidate edge.
3. Проверить, появляется ли путь от dependent обратно к dependency.
4. Если путь есть, candidate edge не записывается как active blocking.
5. Сохранить blocker или discussion marker для dependent issue.
6. Предложить варианты: убрать связь, поменять direction, сделать `informs`, объединить issue, создать parent complex issue через [complex_issue_protocol.md](complex_issue_protocol.md).

## Propagation при изменении dependency

Если dependency issue изменил requirements, solution, contract, output или validation после того, как dependent issue использовал его результат:

1. Найти direct dependents через graph.
2. Для blocking dependents поставить `dependency_ready = stale`.
3. В `blocking_reason` указать changed artifact и required recheck.
4. Не закрывать dependent issue до повторной проверки affected requirements, solution или contract.
5. Если dependency закрыт с notes, dependent должен явно принять notes как риск или превратить их в blocker.
6. Если dependency rejected/deferred/tombstone, blocking dependents получают `blocked` до пересмотра solution.

Propagation не требует загрузки всего repository. Для локального шага достаточно edges выбранного issue, direct dependencies и parent summary.

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

- `dependency_refs`: список edge IDs;
- `dependency_ready`: `blocked`, `stale` или `cycle_blocked`;
- `blocking_reason`: короткое условие снятия блока.

## Repair и удаление связи

Связь можно изменить только с reason:

| Ситуация | Действие |
|---|---|
| Dependency стала ready | обновить edge readiness/status и dependent `dependency_ready` |
| Edge direction ошибочный | записать corrected edge и пометить старый как superseded/rejected, если такая политика уже есть |
| Связь стала non-blocking | сменить relation to `informs`, `blocking = false`, снять blocker |
| Issue объединяются | создать merge decision в affected issue state/registry |
| Issue разделяются | использовать [complex_issue_protocol.md](complex_issue_protocol.md) или new linked issue |
| Target удалён/tombstone | dependent получает blocker до пересмотра |

Полное удаление edge без следа запрещено, если на него уже ссылался registry или output. Минимум должен остаться reason в graph или affected issue state.

## Failure behavior

| Сбой | Статус | Действие |
|---|---|---|
| Issue отсутствует в registry | `blocked_on_missing_issue` | создать/восстановить issue или отклонить связь |
| Relation reason слабый | `needs_dependency_reason` | запросить конкретный required artifact/state |
| Cycle detected | `cycle_blocked` | не активировать blocking edge; предложить repair options |
| Required artifact отсутствует | `blocked_on_dependency_artifact` | заблокировать dependent или разрешить QA/requirements-only work |
| Graph/status расходится с registry | `blocked_on_graph_registry_mismatch` | выполнить repair transaction |
| Persistence недоступен | `blocked_on_persistence` | не считать edge созданным |

## Completion signal

Протокол завершён, когда dependency edge сохранён или отклонён с reason, registry mirror обновлён, affected issue state отражает blocker/readiness, а [../../State/persistence_log.jsonl](../../State/persistence_log.jsonl) содержит transaction entry. Если edge blocked или stale, следующий routing показывает dependent issue как неготовый к execution/validation/closure.

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
