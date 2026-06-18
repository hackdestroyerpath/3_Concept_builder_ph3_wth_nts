# Протокол работы с существующим issue

Parent: [Каталог протоколов](../catalog.md)  
Owner issue: `EXEC-007`  
Protocol ID: `service/existing_issue`  
Источник истины: `Protocols/service_protocols/existing_issue_protocol.md`  
Status: `available`  
Updated: `2026-06-18T16:42:00Z`

## Назначение

Протокол выбирает ровно один существующий service-level `issue`, сверяет registry, state, dependency graph и focus, затем формирует минимальный packet для следующей phase. Он не создаёт новый issue и не начинает параллельную работу, если nonterminal issue уже покрывает запрос.

## Runtime scaffold

Runtime folder создаётся только когда lifecycle требует phase artifacts. Пустые folders запрещены.

```text
Issues/<issue_id>/
├── state.md
├── reason.md
├── requirements.md
├── solution.md
├── contract.md
└── output/
    ├── report.md
    ├── changed_files.md
    ├── contract_coverage.md
    └── attachments_manifest.jsonl
```

QA artifacts создаются только после фактического QA или explicit QA-trace requirement. Bootstrap issue могут оставаться registry-only, если registry содержит sufficient reason, target paths и next action.

## Обязательные входы

| Вход | Назначение |
|---|---|
| [../../State/service_state.md](../../State/service_state.md) | active issue, phase, pending action, return anchor |
| [../../Issues/issue_registry.md](../../Issues/issue_registry.md) | human lifecycle snapshot |
| [../../Issues/issue_registry.jsonl](../../Issues/issue_registry.jsonl) | exact selected row |
| [../../Issues/dependency_graph.jsonl](../../Issues/dependency_graph.jsonl) | blockers, stale state, cycles |
| [../../State/page_registry.jsonl](../../State/page_registry.jsonl) | path and parent integrity |
| [../catalog.md](../catalog.md) | next protocol status |
| [../common/context_loading_protocol.md](../common/context_loading_protocol.md) | focus boundary |
| [../common/persistence_protocol.md](../common/persistence_protocol.md) | transaction guard |

Runtime files читаются только если registry указывает на них и файл существует.

## Preconditions

1. Registry и dependency graph parse clean.
2. Selected `issue_id` существует, иначе возвращается shortlist.
3. Selected row содержит status, phase, scope, targets, dependency readiness, next action и source/reason evidence.
4. Runtime state согласован с registry либо registry-only bootstrap явно разрешён.
5. Legacy edge readiness нормализован по [linked_issues_protocol.md](linked_issues_protocol.md).
6. `blocked`, `stale`, `cycle_blocked`, `satisfied_for_draft` и `unsatisfied` запрещают execution, validation и closure.
7. Несогласованные registry/state/focus facts дают `focus_evidence_conflict`.

## Duplicate-prevention gate

Перед созданием нового issue или сменой focus:

| Проверка | Pass condition |
|---|---|
| Active state | active issue отсутствует/совпадает; для switch есть explicit decision и return anchor |
| Registry overlap | среди других rows (`issue_id != selected issue_id`) нет nonterminal issue со materially overlapping scope/targets |
| Focus packet | switch не теряет pending user action |
| Dependency graph | существующая edge уже не выражает тот же dependency/overlap |
| Return anchor | известна точка возврата |

Material overlap существует при совпадающем или ancestor/descendant scope вместе с общим, вложенным либо тем же изменяемым target object. Exact-ID continuation не блокируется собственной registry row.

Fail gate ведёт к continuation, merge/split/link/defer decision, а не к автоматическому duplicate issue.

## Порядок выполнения

### 1. Resolve reference

| Ситуация | Действие |
|---|---|
| Exact ID | выбрать одну row |
| Partial ID/title | вернуть shortlist |
| `продолжай` | взять active issue и сверить registry |
| Несколько matches | не выбирать автоматически |
| Active issue отличается | показать switch/link/defer options |

Shortlist содержит только ID, title, status, phase, dependency readiness и next action.

### 2. Lifecycle routing

| Status / phase | Действие | Next protocol |
|---|---|---|
| `creating` | intake/repair | `service/new_issue` или blocker |
| `proposed` | reason decision | planned decision update |
| `needs_discussion` | scope/reason discussion | planned decision update |
| `approved` без phase | применить initial-phase table | concrete phase protocol |
| `active: qa` | QA continuation | `service/question_answer` |
| `active: requirements` | requirements continuation | `service/requirements` |
| `active: requalification` | simple/complex check | `service/complex_issue` |
| `active: solution` | solution/contract review | `service/solution_contract_output` |
| `active: execution` | approved execution | `service/solution_contract_output` |
| `active: validation` | final validation | `common/final_validation` |
| `closed` / `rejected` | read-only/retention | `service/issue_retention` |
| `archived` / `tombstone` / `deleted` | read-only | none/retention |
| `blocked` | show unblock condition | source phase/repair |
| `deferred` | require revive reason | revive decision |

Status/phase mismatch не исправляется молча.

### 3. Dependency readiness

Direct edges нормализуются chronology-aware. Для execution/validation/closure каждый active blocking edge должен быть `ready`. Generic legacy `satisfied` требует artifact/state evidence. Historical draft-only marker повышается до `ready` только более поздним committed final validation/artifact coverage.

QA, requirements и explicitly scoped bootstrap draft могут продолжаться при `satisfied_for_draft`, если отсутствующий artifact не нужен текущему draft step.

### 4. Initial phase для `approved` issue без phase

Rows проверяются сверху вниз; первая подтверждённая row выигрывает. Fallback rows содержат отрицательные условия, поэтому QA-first routing не считается конфликтом.

| Priority | Evidence | Phase | Next protocol |
|---|---|---|---|
| 1 | reason/source weak или missing | `qa` | `service/question_answer` |
| 2 | reason sufficient, но blocking material unknowns мешают requirements | `qa` | `service/question_answer` |
| 3 | blocking unknowns отсутствуют; requirements missing/not approved | `requirements` | `service/requirements` |
| 4 | requirements approved; simple/complex boundary не подтверждена | `requalification` | `service/complex_issue` |
| 5 | requirements approved; issue simple; solution/contract missing/not approved | `solution` | `service/solution_contract_output` |
| 6 | solution+contract approved; dependencies ready; output missing/incomplete | `execution` | `service/solution_contract_output` |
| 7 | output saved; contract pass; dependencies ready; validation missing/not pass | `validation` | `common/final_validation` |
| 8 | final validation pass уже существует | terminal repair | status должен быть `closed`, иначе conflict |

`focus_evidence_conflict` применяется только при противоречивых facts внутри выбранной priority, conflicting artifact versions или registry/state mismatch. Later fallback rows сами по себе conflict не создают.

Planned next protocol не исполняется как available; возвращается `next_protocol_planned`.

### 5. Focus packet

| Поле | Значение |
|---|---|
| `issue_id`, `title` | selected registry identity |
| `mode`, `scope_path` | Service Mode и scope |
| `status`, `phase` | reconstructed lifecycle |
| `dependency_ready` | normalized direct-edge result |
| `dependency_evidence` | edge IDs и short evidence |
| `reason_summary` | concise reason |
| `loaded_files` | реально прочитанные files |
| `missing_expected_files` | missing paths с reason |
| `return_anchor` | previous focus/state/user command |
| `next_protocol` | available/planned/blocked ID |
| `next_action` | одно действие |

Registry-only bootstrap folder не считается missing до phase, которая реально требует runtime artifact.

### 6. Persistence

Write требуется при active transition, phase change, focus switch, blocker/inconsistency, dependency-readiness update или return-anchor change.

Обычный write set:

1. [service_state.md](../../State/service_state.md);
2. [issue_registry.jsonl](../../Issues/issue_registry.jsonl);
3. [issue_registry.md](../../Issues/issue_registry.md);
4. [dependency_graph.jsonl](../../Issues/dependency_graph.jsonl), если edges/readiness менялись;
5. [persistence_log.jsonl](../../State/persistence_log.jsonl).

Без подтверждённой записи active transition не заявляется.

## Failure behavior

| Сбой | Статус | Действие |
|---|---|---|
| ID отсутствует | `issue_not_found` | shortlist/new issue route |
| Multiple candidates | `ambiguous_issue_reference` | shortlist |
| Другой issue materially покрывает запрос | `duplicate_issue_prevented` | continue/merge/split/link/defer |
| Registry parse error | `blocked_on_registry_parse` | repair write set |
| Registry/state/graph conflict | `focus_evidence_conflict` | repair before phase work |
| Cycle | `blocked_on_dependency_cycle` | no active/execution transition |
| Draft-only runtime dependency | `blocked_on_draft_only_dependency` | draft only или full evidence |
| Final/draft evidence conflict | `blocked_on_dependency_evidence_conflict` | check chronology/coverage |
| Unsatisfied/ambiguous dependency | `blocked_on_dependency` | show edge and unblock condition |
| Required runtime artifact missing | `missing_issue_artifact` | bootstrap exception or repair |
| Next protocol planned | `next_protocol_planned` | stop after focus packet |
| Persistence unavailable | `blocked_on_persistence` | no claimed transition |

## Completion signal

Протокол завершён, когда выбран один issue и concrete next protocol, выдан shortlist, либо сохранён честный blocker с write set и next action.

## Связанные файлы

- [Catalog](../catalog.md)
- [Context loading protocol](../common/context_loading_protocol.md)
- [Persistence protocol](../common/persistence_protocol.md)
- [Service start protocol](service_start_protocol.md)
- [New issue protocol](new_issue_protocol.md)
- [Question Answer protocol](question_answer_protocol.md)
- [Requirements protocol](requirements_protocol.md)
- [Solution / Contract / Output protocol](solution_contract_output_protocol.md)
- [Complex Issue protocol](complex_issue_protocol.md)
- [Linked Issues protocol](linked_issues_protocol.md)
- [Issue Retention protocol](issue_retention_protocol.md)
- [Service state](../../State/service_state.md)
- [Issue registry](../../Issues/issue_registry.md)
- [Issue registry JSONL](../../Issues/issue_registry.jsonl)
- [Dependency graph](../../Issues/dependency_graph.jsonl)
- [Page registry](../../State/page_registry.jsonl)
