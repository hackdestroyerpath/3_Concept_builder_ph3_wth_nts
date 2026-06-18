# Протокол Complex Issue

Parent: [Каталог протоколов](../catalog.md)  
Owner issue: `EXEC-010`  
Protocol ID: `service/complex_issue`  
Источник истины: `Protocols/service_protocols/complex_issue_protocol.md`  
Status: `available`  
Updated: `2026-06-18T01:29:00Z`

## Назначение

Этот протокол управляет requalification, когда selected issue нельзя безопасно выполнить как один `simple issue`: требуются независимые work packages, разные outputs, parent/child boundaries, sequencing dependencies или отдельные validation criteria.

`Complex issue` не является способом спрятать неопределённость. Он применяется только если decomposition снижает риск, делает output проверяемым или предотвращает преждевременное execution зависимых частей.

Протокол не заменяет [requirements_protocol.md](requirements_protocol.md), [solution_contract_output_protocol.md](solution_contract_output_protocol.md) и [linked_issues_protocol.md](linked_issues_protocol.md). Он определяет parent/children-модель, dependency rules и readiness gate.

## Когда использовать

| Состояние | Действие |
|---|---|
| `active: requalification` | проверить, остаётся issue `simple` или требует `complex` |
| Approved requirements требуют независимых outputs | подготовить decomposition plan в parent `solution.md` |
| Parent decomposition approved | создать child issue registry entries и dependency edges |
| Child issue закрыт | поднять result summary в parent state/contract coverage |
| Parent готовится к validation | проверить required children, stale edges и cycle status |
| Decomposition budget исчерпан | остановиться и запросить user decision |

Если нужна только связь между независимыми issue без parent/child relation, используется [linked_issues_protocol.md](linked_issues_protocol.md). Retention lifecycle выполняется только через [issue_retention_protocol.md](issue_retention_protocol.md).

## Обязательные входы

| Вход | Источник |
|---|---|
| Parent issue registry row | [../../Issues/issue_registry.jsonl](../../Issues/issue_registry.jsonl) |
| Parent state | `Issues/<issue_id>/state.md`, если существует |
| Requirements | `Issues/<issue_id>/requirements.md`, approved через [requirements_protocol.md](requirements_protocol.md) |
| Reason / input refs | `reason.md`, registry `reason_summary`, `input_ref` |
| Dependency graph | [../../Issues/dependency_graph.jsonl](../../Issues/dependency_graph.jsonl) |
| Existing focus | [existing_issue_protocol.md](existing_issue_protocol.md) |
| Solution/contract rules | [solution_contract_output_protocol.md](solution_contract_output_protocol.md) |
| Linked dependency rules | [linked_issues_protocol.md](linked_issues_protocol.md) |
| Page registry | [../../State/page_registry.jsonl](../../State/page_registry.jsonl) |
| Persistence rules | [../common/persistence_protocol.md](../common/persistence_protocol.md) |
| Catalog | [../catalog.md](../catalog.md) и [../catalog.jsonl](../catalog.jsonl) |

## Preconditions

1. Issue выбран через existing issue protocol или уже active в state.
2. Requirements approved, либо пользователь явно просит preliminary decomposition draft без child creation.
3. Reason requalification записан: independent outputs, owners, dependency sequencing, scope conflict, large execution package или separate validation criteria.
4. Parent blocking dependencies проверены по graph.
5. Пользователь утвердил смену `type = simple` на `complex`, если issue уже был approved как simple.
6. Persistence доступен. Без подтверждённой записи child issue не считаются созданными.
7. Runtime folders для child issue не создаются пустыми; создаются только при реальном phase artifact.

## Граница simple / complex

| Признак | Решение |
|---|---|
| Один output, один contract, связный work set | оставить `simple` |
| Несколько независимых outputs или acceptance groups | предложить `complex` |
| Нужна последовательность A -> B | `complex` или linked dependencies, выбрать минимальную модель |
| Часть scope можно проверить отдельно | `complex`, если это снижает риск |
| Новая тема не входит в parent goal | создать linked issue, а не child |
| Decomposition добавляет только файлы без проверяемой пользы | оставить `simple` |

Тип issue не меняется молча. Requalification packet показывает reason, affected requirements, affected files, parent/child consequences и dependency impact.

## Decomposition budget

Каждый complex issue имеет локальный `decomposition_budget`. Он фиксируется в parent `solution.md` и кратко отражается в registry.

| Поле | Значение |
|---|---|
| `budget_reason` | почему decomposition нужна |
| `max_depth` | допустимая глубина children относительно parent |
| `max_child_count` | expected limit или reason открытого лимита |
| `child_scope_rule` | какие части scope можно отдавать children |
| `dependency_rule` | какие children блокируют parent/друг друга |
| `context_budget` | какие summaries можно читать без wide context lift |
| `stop_conditions` | когда decomposition прекращается |
| `escalation_rule` | что делать при исчерпании budget |

Если новый child нарушает budget, agent не создаёт его молча. Он ставит blocker `blocked_on_decomposition_budget` и предлагает расширить budget, объединить children, вынести scope в non-goal, создать linked issue или вернуться к parent solution.

## Decomposition plan в parent `solution.md`

```text
## Decomposition plan

Status: draft | review | approved | reopened
Parent issue: <issue_id>
Decomposition reason: <why complex>
Budget:
  max_depth: <number or local rule>
  max_child_count: <number or local rule>
  stop_conditions: <list>

| Child candidate | Purpose | Required output | Blocking parent? | Dependency refs | Acceptance link |
|---|---|---|---|---|---|
| CHILD-001 | ... | ... | yes/no | ... | REQ-... |

## Parent closure rule
- Required children: ...
- Optional / non-goal children: ...
- Parent validation evidence: ...
```

До approval это proposal, не permission на runtime creation. После approval agent создаёт child registry entries и dependency edges через controlled transaction.

## Создание child issue

Каждый child является обычным `issue`, а не отдельным типом `sub-issue`.

Минимальные действия:

1. Сгенерировать stable `issue_id` по текущей registry policy.
2. Создать registry entry с `parent_id`, `depth_level = parent.depth_level + 1`, own reason, status `proposed` или `approved` по user decision.
3. Обновить parent `children_ids`.
4. Если child блокирует parent closure или другой child, добавить edge в [../../Issues/dependency_graph.jsonl](../../Issues/dependency_graph.jsonl) по [linked_issues_protocol.md](linked_issues_protocol.md).
5. Создать runtime files child issue только если lifecycle требует `state.md`, `reason.md`, requirements или downstream artifacts. Пустые папки запрещены.
6. Обновить [../../State/page_registry.jsonl](../../State/page_registry.jsonl), если создан Markdown-файл.
7. Добавить [../../State/persistence_log.jsonl](../../State/persistence_log.jsonl) transaction entry.

Child наследует parent context summary, но не копирует весь parent text. Parent хранит purpose, status, dependency readiness, output pointer и next step.

## Dependency readiness для parent / children

| Ситуация | Правило |
|---|---|
| Child нужен для parent closure | parent не идёт в validation, пока child не closed или явно deferred/rejected с approved reason |
| Child A нужен для Child B | graph edge blocks Child B execution до ready edge |
| Child output изменился после использования parent | parent получает `dependency_ready = stale` до recheck contract coverage |
| Edge создаёт cycle | active blocking edge не сохраняется; affected issue получает `cycle_blocked` / `needs_discussion` |
| Child стал самостоятельной темой | child переводится в linked/detached issue с reason; parent scope обновляется |

Перед выбором child agent читает только parent summary, child registry row и direct dependency edges. Full parent context грузится только с reason.

## Stale dependency handling

Stale state возникает, если dependency artifact, child output, requirement, solution, contract или validation изменились после использования parent/child.

Agent обязан:

1. отметить affected parent/child `dependency_ready = stale`;
2. указать changed artifact и required recheck;
3. остановить execution/validation/closure dependent issue;
4. route к requirements, solution, contract coverage или linked dependency repair;
5. снять stale только после committed recheck.

## Возврат результата child в parent

После closure child issue agent обязан:

1. сохранить child output и validation pointer;
2. обновить parent `state.md`, если существует;
3. обновить parent registry: closed child, remaining children, dependency readiness, parent contract coverage pointer;
4. свернуть details child до summary;
5. выбрать следующий child, parent solution route или user decision;
6. обновить next-step marker.

Parent не закрывается только потому, что последний child закрыт. Parent должен пройти own validation и contract coverage.

## Parent closure gate

Complex issue можно закрыть только если:

- все required child issue terminal `closed`, либо scope explicitly non-goal/deferred/rejected with reason;
- parent contract coverage показывает child outputs per requirement;
- graph не содержит active `blocked`, `stale` или `cycle_blocked` edges для parent scope;
- parent validation report сохранён;
- registry, parent state и page registry согласованы.

## Failure behavior

| Сбой | Статус | Действие |
|---|---|---|
| Нет approved requirements | `blocked_on_requirements` | вернуться к [requirements_protocol.md](requirements_protocol.md) |
| User не утвердил смену типа | `needs_requalification_approval` | сохранить proposal и ждать решения |
| Budget исчерпан | `blocked_on_decomposition_budget` | запросить расширение budget или scope decision |
| Child scope дублирует existing issue | `needs_dedup_decision` | use linked/merge/defer decision |
| Dependency cycle | `cycle_blocked` | не сохранять active blocking edge; предложить repair |
| Stale child/parent dependency | `blocked_on_stale_dependency` | recheck affected requirements/contract/output |
| Нельзя записать registry/state | `blocked_on_persistence` | не считать children созданными |
| Parent closure без children coverage | `blocked_on_parent_contract_coverage` | вернуть parent к solution/execution/validation |

## Completion signal

Протокол завершён, когда parent сохранён как complex с approved decomposition plan, child registry entries и dependency edges созданы; либо issue оставлен simple с reason; либо работа остановлена с blocker. После создания children routing выбирается через [existing_issue_protocol.md](existing_issue_protocol.md).

## Связанные файлы

- [Catalog](../catalog.md)
- [Catalog JSONL](../catalog.jsonl)
- [Existing issue protocol](existing_issue_protocol.md)
- [Requirements protocol](requirements_protocol.md)
- [Solution / Contract / Output protocol](solution_contract_output_protocol.md)
- [Linked Issues protocol](linked_issues_protocol.md)
- [Issue Retention protocol](issue_retention_protocol.md)
- [Issue registry](../../Issues/issue_registry.md)
- [Issue registry JSONL](../../Issues/issue_registry.jsonl)
- [Dependency graph](../../Issues/dependency_graph.jsonl)
- [Service state](../../State/service_state.md)
- [Page registry](../../State/page_registry.jsonl)
- [Persistence protocol](../common/persistence_protocol.md)
