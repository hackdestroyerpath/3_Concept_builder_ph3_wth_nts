# Протокол Solution / Contract / Output

Parent: [Каталог протоколов](../catalog.md)  
Owner issue: `EXEC-009`  
Protocol ID: `service/solution_contract_output`  
Источник истины: `Protocols/service_protocols/solution_contract_output_protocol.md`  
Status: `available`  
Updated: `2026-06-18T20:00:00Z`

## Назначение

Этот протокол управляет simple issue после approved requirements: подготовка `solution.md`, подготовка `contract.md`, approval, execution и сохранение `output/` package. Он не заменяет [requirements_protocol.md](requirements_protocol.md), [complex_issue_protocol.md](complex_issue_protocol.md), [linked_issues_protocol.md](linked_issues_protocol.md) или final validation.

Разделение источников истины строгое:

| Artifact | Роль |
|---|---|
| `solution.md` | выбранный подход, границы scope, план и affected files |
| `contract.md` | acceptance criteria, evidence rules и conditions of non-closure |
| `output/` | фактически выполненная работа, changed files и coverage |

`output/` не описывает план как будто он уже выполнен. `solution.md` не заменяет contract. `contract.md` не должен быть отчётом о выполнении.

## Когда использовать

| Состояние | Действие |
|---|---|
| `active: solution` | создать или обновить `solution.md` + `contract.md` в status `review` |
| `requirements.md` approved и issue simple | подготовить solution/contract packet |
| Пользователь изменил solution или contract | обновить artifact и approval log |
| `solution.md` + `contract.md` approved | перейти к `active: execution` |
| `active: execution` | выполнить approved work set и сохранить `output/` |
| Output не покрывает contract | сохранить gap и вернуть issue к execution/solution/requirements |

Если issue требует parent/child decomposition или независимых work packages, этот протокол останавливается и routes к [complex_issue_protocol.md](complex_issue_protocol.md). Если нужна внешняя dependency без parent/child, routing идёт к [linked_issues_protocol.md](linked_issues_protocol.md).

## Обязательные входы

| Вход | Источник |
|---|---|
| Выбранный issue | [../../Issues/issue_registry.jsonl](../../Issues/issue_registry.jsonl), registry summary, state если есть |
| Reason | `Issues/<issue_id>/reason.md` или registry `reason_summary` |
| Approved requirements | `Issues/<issue_id>/requirements.md`, созданный через [requirements_protocol.md](requirements_protocol.md) |
| QA summary / skip reason | QA artifacts или `qa_skipped_with_reason` из requirements |
| Dependency status | [../../Issues/dependency_graph.jsonl](../../Issues/dependency_graph.jsonl) |
| Existing focus | [existing_issue_protocol.md](existing_issue_protocol.md) |
| Catalog route | [../catalog.md](../catalog.md) и [../catalog.jsonl](../catalog.jsonl) |
| Persistence rules | [../common/persistence_protocol.md](../common/persistence_protocol.md) |
| Page registry | [../../State/page_registry.jsonl](../../State/page_registry.jsonl) |

## Preconditions

1. Issue выбран через existing issue protocol или уже зафиксирован в state.
2. `status = active`, `phase = solution` или `phase = execution`.
3. `requirements.md` существует, сохранён и approved.
4. Issue подтверждён как `simple`, либо requalification не выявила complex scope.
5. Dependency edges нормализованы по [linked_issues_protocol.md](linked_issues_protocol.md):
   - solution/contract draft допускается при `satisfied_for_draft`, только если отсутствующий artifact не нужен для текущего design step и risk записан;
   - approval, переход к execution, runtime execution и validation запрещены при active blocking readiness `blocked`, `stale`, `cycle_blocked`, `satisfied_for_draft` или `unsatisfied`;
   - generic legacy `status = satisfied` не считается `ready` без artifact/state evidence.
6. Affected files и scope перечислены или могут быть перечислены в `solution.md`.
7. Persistence доступен. Если запись невозможна, agent не заявляет, что artifacts стали источником истины.

## Runtime scaffold

Для runtime issue этот протокол использует только необходимые files:

```text
Issues/<issue_id>/
├── solution.md
├── contract.md
└── output/
    ├── report.md
    ├── changed_files.md
    ├── contract_coverage.md
    └── attachments_manifest.jsonl
```

Папка `output/attachments/` создаётся только если есть реальные attachments. Пустые runtime/output folders не создаются заранее.

## Phase routing

| Текущее состояние | Разрешённое действие | Следующее состояние |
|---|---|---|
| approved requirements, simple issue | создать `solution.md` и `contract.md` в status `review` | ожидание user approval |
| user approves solution + contract и dependencies normalized `ready` | перевести оба artifact в `approved` | `active: execution` |
| user requests changes | обновить artifact и approval log | `active: solution` / review |
| approved solution + contract, dependencies `ready` | выполнить work set | `active: execution` |
| output saved and contract covered | route к validation | `active: validation` |
| output incomplete | сохранить gap и blocker | `active: execution` или return phase |

## Approval gate

Execution без approved `solution.md` и approved `contract.md` запрещён. Approval также не снимает dependency blocker: transition к execution выполняется только после normalized graph check.

Обычный approval выполняется ответом пользователя `утверждаю solution и contract` или эквивалентным однозначным решением по обоим artifacts.

Узкий shortcut `выполнить без отдельного обсуждения` допустим только как атомарное approval обеих частей, если одновременно выполнены все условия:

1. `solution.md` и `contract.md` уже сохранены и прошли собственные quality gates;
2. user intent однозначно относится к текущему `issue_id`, exact work set и обоим artifacts;
3. approval log обоих files фиксирует explicit shortcut decision и timestamp;
4. оба files получают `Status: approved`, а registry/state переходят к `active: execution` одной persistence transaction;
5. branch/commit readback подтверждает artifacts и transition до начала execution;
6. все active blocking dependencies normalized to `ready`, required artifacts проверены и scope не расширился после user decision.

Одного committed `contract.md`, approval только solution или общего сообщения `делай` без привязки к review packet недостаточно. При ambiguity или draft-only dependency agent сохраняет review state и возвращает blocker.

## Минимальная структура `solution.md`

```text
# Решение — <issue_id>

Parent: [state.md](state.md)
Status: draft | review | approved | reopened
Approved by: user | not_approved
Approved at: <timestamp or null>
Source issue: <issue_id>
Requirements: requirements.md

## Исходные материалы
- Issue state: ...
- Reason: ...
- Requirements: ...
- QA summary / skip reason: ...
- Dependency status: ...

## Карта покрытия требований
| REQ-ID | How solution covers it | Notes |
|---|---|---|

## Предлагаемый подход
...

## План
1. ...

## Затронутые файлы и объекты
| Path / object | Change type | Owner requirement | Notes |
|---|---|---|---|

## Риски, ограничения и нецели
- ...

## Журнал утверждений
| Time | Actor | Action | Notes |
|---|---|---|---|
```

### Quality gate для solution

`solution.md` годится для review только если:

1. каждое approved requirement покрыто или исключено с user-approved reason;
2. план конкретен для execution, но не описывает output как уже готовый;
3. affected files перечислены настолько точно, насколько позволяет scope;
4. known risks, assumptions и non-goals записаны явно;
5. новый scope не появляется без requirements approval;
6. normalized dependencies, draft-only boundaries и stale risks отражены в плане.

## Минимальная структура `contract.md`

```text
# Контракт — <issue_id>

Parent: [state.md](state.md)
Status: draft | review | approved | reopened
Approved by: user | not_approved
Approved at: <timestamp or null>
Source issue: <issue_id>
Requirements: requirements.md
Solution: solution.md

## Критерий готовности
- ...

## Проверки требований
| REQ-ID | Required evidence | Pass condition |
|---|---|---|

## Contract checks
| Check ID | Required evidence | Pass condition | Non-closure condition |
|---|---|---|---|

## Ожидаемый output
- report.md: required
- changed_files.md: required
- contract_coverage.md: required
- attachments_manifest.jsonl: required, may be empty

## Метод проверки
- manual check / file check / link check / protocol check / user confirmation

## Журнал утверждений
| Time | Actor | Action | Notes |
|---|---|---|---|
```

### Quality gate для contract

Contract нельзя вынести на review, если он:

- не указывает evidence для каждого approved requirement;
- использует непроверяемые критерии;
- не говорит, какие output artifacts должны появиться;
- не содержит conditions of non-closure;
- не различает required output и optional attachments;
- позволяет закрыть issue без validation или user-approved exception.

## Execution rules

Перед execution агент обязан:

1. перечитать approved `solution.md`, approved `contract.md`, registry, dependency graph и relevant target files;
2. нормализовать direct blocking edges и остановиться при `blocked`, `stale`, `cycle_blocked`, `satisfied_for_draft`, `unsatisfied` или ambiguous legacy `satisfied` без required evidence;
3. проверить, что affected files находятся в approved scope;
4. сформировать work set и write set;
5. выполнить только действия, покрытые solution и contract;
6. если нужен новый scope, остановиться и создать linked issue/backlog entry вместо молчаливого расширения;
7. сохранить output package до ответа пользователю;
8. обновить registry/state/persistence markers после artifacts.

## Минимальная структура `output/`

```text
Issues/<issue_id>/output/
├── report.md
├── changed_files.md
├── contract_coverage.md
└── attachments_manifest.jsonl
```

### `output/report.md`

```text
# Отчёт output — <issue_id>

Parent: [../state.md](../state.md)
Status: draft | saved | blocked
Source issue: <issue_id>
Solution: ../solution.md
Contract: ../contract.md

## Сводка выполненной работы
...

## Выполненные шаги solution
| Step | Status | Evidence |
|---|---|---|

## Requirements coverage summary
| REQ-ID | Covered | Evidence | Notes |
|---|---|---|---|

## Contract coverage summary
| Check ID | Status | Evidence | Notes |
|---|---|---|---|

## Изменённые или созданные файлы
- ...

## Attachments
- none / ...

## Отклонения от solution
- none / ...

## Remaining notes / blockers
- ...

## Recommended next action
...
```

### `output/changed_files.md`

```text
# Изменённые файлы — <issue_id>

Parent: [report.md](report.md)

| Path | Change type | Owner issue | Requirement | Notes |
|---|---|---|---|---|
```

### `output/contract_coverage.md`

```text
# Покрытие contract — <issue_id>

Parent: [report.md](report.md)

| Contract check | Pass / Fail / N/A | Evidence | Action if fail |
|---|---|---|---|
```

### `output/attachments_manifest.jsonl`

Каждая строка содержит один JSON-объект:

```json
{"path":"Issues/<issue_id>/output/attachments/<file>","purpose":"...","source":"...","related_requirement":"REQ-...","related_contract_check":"...","status":"kept|generated|external_ref"}
```

Если attachments не нужны, manifest может быть пустым. Если output ссылается на attachment, соответствующая строка manifest обязательна.

## Registry и State updates

При создании или изменении artifacts агент обновляет:

1. `solution_path`, `contract_path`, `output_path`, `phase`, `status`, `updated_at` в [../../Issues/issue_registry.jsonl](../../Issues/issue_registry.jsonl);
2. [../../Issues/issue_registry.md](../../Issues/issue_registry.md), если меняется human snapshot;
3. `Issues/<issue_id>/state.md`, если runtime issue folder существует;
4. [../../State/page_registry.jsonl](../../State/page_registry.jsonl), если созданы Markdown-файлы;
5. [../../State/persistence_log.jsonl](../../State/persistence_log.jsonl) после primary artifacts и registry updates;
6. [../../Issues/dependency_graph.jsonl](../../Issues/dependency_graph.jsonl), если output создаёт required artifact, makes dependency ready или делает dependents stale.

Bootstrap implementation issue без runtime folder могут оставаться registry-only. Runtime artifacts не создаются без фактического runtime-перехода.

## Failure behavior

| Сбой | Статус | Действие |
|---|---|---|
| Нет approved requirements | `blocked_on_requirements` | вернуться к [requirements_protocol.md](requirements_protocol.md) |
| Issue оказался complex | `needs_complex_requalification` | route к [complex_issue_protocol.md](complex_issue_protocol.md) |
| Contract непроверяем | `contract_quality_fail` | переписать contract до review |
| User approval ambiguous или покрывает только один artifact | `needs_confirmation` | сохранить draft/review, запросить точное решение по solution и contract |
| Dependency draft-only | `blocked_on_draft_only_dependency` | разрешить только scoped draft; получить full readiness перед approval/execution |
| Legacy dependency evidence ambiguous | `blocked_on_dependency_evidence` | проверить required artifact/source validation |
| Affected files вне scope | `scope_violation_blocked` | вернуть к requirements или создать linked issue через [linked_issues_protocol.md](linked_issues_protocol.md) |
| Dependency stale | `blocked_on_stale_dependency` | обновить readiness и получить required artifact |
| Execution не покрывает contract | `output_contract_gap` | сохранить gap, не закрывать issue |
| Нельзя сохранить artifacts | `blocked_on_persistence` | не заявлять review/output как committed source of truth |

## Формат ответа на solution/contract review

```text
Сохранено: Issues/<issue_id>/solution.md + Issues/<issue_id>/contract.md.
Status: review.
Краткая сводка solution:
- ...
Ключевые contract checks:
| Check | Evidence required |
|---|---|

Доступные ответы: утвердить solution и contract / изменить solution / изменить contract / вернуться к requirements / отложить.
```

## Формат ответа после execution output

```text
Сохранено: Issues/<issue_id>/output/.
Phase: validation.
Contract coverage: pass | fail | blocked.
Changed files:
- ...
Следующий шаг: final validation / fix / return to solution / return to requirements.
```

Ответ не копирует весь `output/report.md`; он показывает decision packet. Полная версия сохраняется в output files.

## Completion signal

Протокол завершён, когда `solution.md` и `contract.md` сохранены в review/approved; либо approved solution/contract выполнены, `output/` сохранён, registry/state обновлены и phase переведена к validation; либо работа остановлена с честным blocker, write set и next required action.

После successful output следующий routing — `common/final_validation` по [catalog](../catalog.md). После closure archive/tombstone cleanup идёт только через [issue_retention_protocol.md](issue_retention_protocol.md).

## Связанные файлы

- [Catalog](../catalog.md)
- [Catalog JSONL](../catalog.jsonl)
- [Complex Issue protocol](complex_issue_protocol.md)
- [Linked Issues protocol](linked_issues_protocol.md)
- [Requirements protocol](requirements_protocol.md)
- [Question Answer protocol](question_answer_protocol.md)
- [Existing issue protocol](existing_issue_protocol.md)
- [New Issue protocol](new_issue_protocol.md)
- [Service start protocol](service_start_protocol.md)
- [Service state](../../State/service_state.md)
- [Issue Retention protocol](issue_retention_protocol.md)
- [Issue registry](../../Issues/issue_registry.md)
- [Issue registry JSONL](../../Issues/issue_registry.jsonl)
- [Dependency graph](../../Issues/dependency_graph.jsonl)
- [Page registry](../../State/page_registry.jsonl)
- [Persistence protocol](../common/persistence_protocol.md)
