# Протокол Solution / Contract / Output

Parent: [Каталог протоколов](../catalog.md)  
Owner issue: `EXEC-009`  
Protocol ID: `service/solution_contract_output`  
Источник истины: `Protocols/service_protocols/solution_contract_output_protocol.md`  
Status: `available`  
Updated: `2026-06-05T11:45:45Z`

## Назначение

Этот протокол управляет участком `simple issue` после утверждения требований: подготовка `solution.md`, подготовка `contract.md`, утверждение пользователем, выполнение approved solution и сохранение `output/` package. Он не заменяет [requirements_protocol.md](requirements_protocol.md), не выполняет complex decomposition и не закрывает issue без проверки. Одного слова “готово” недостаточно: закрытие требует contract coverage и validation evidence.

Главный принцип: `solution.md` объясняет способ выполнения, `contract.md` задаёт проверяемый критерий готовности, а `output/` фиксирует то, что реально было сделано. Эти артефакты не дублируют друг друга и не соревнуются за источник истины.

## Когда использовать

| Состояние | Действие |
|---|---|
| `active: solution` | создать, обновить или вынести на review `solution.md` и `contract.md` |
| `requirements.md` approved и issue подтверждён как `simple` | подготовить solution/contract packet |
| Пользователь изменил proposed solution или contract | обновить соответствующий artifact и approval log |
| `solution.md` и `contract.md` approved | перейти к `execution` |
| `active: execution` | выполнить approved solution в разрешённом scope и сохранить `output/` |
| Output не покрывает contract | зафиксировать blocker и вернуть issue в `execution`, `solution` или `requirements` |

Если issue требует parent/child decomposition или независимых work packages, этот протокол не притворяется decomposer. Он должен остановиться и вернуть routing к [complex_issue_protocol.md](complex_issue_protocol.md). Если нужен внешний dependency без parent/child, routing идёт к [linked_issues_protocol.md](linked_issues_protocol.md).

## Обязательные входы

| Вход | Источник |
|---|---|
| Выбранный issue | [../../Issues/issue_registry.jsonl](../../Issues/issue_registry.jsonl) и, если есть, `Issues/<issue_id>/state.md` |
| Reason | `Issues/<issue_id>/reason.md` или registry `reason_summary` |
| Approved requirements | `Issues/<issue_id>/requirements.md`, созданный через [requirements_protocol.md](requirements_protocol.md) |
| QA summary | `Issues/<issue_id>/qa/qa_summary.md` или recorded skip reason |
| Dependency status | [../../Issues/dependency_graph.jsonl](../../Issues/dependency_graph.jsonl) и при необходимости [linked_issues_protocol.md](linked_issues_protocol.md) |
| Catalog route | [../catalog.md](../catalog.md) и [../catalog.jsonl](../catalog.jsonl) |
| Page registry | [../../State/page_registry.jsonl](../../State/page_registry.jsonl) |
| Persistence rules | [../common/persistence_protocol.md](../common/persistence_protocol.md) |
| Existing issue focus | [existing_issue_protocol.md](existing_issue_protocol.md) |

## Preconditions

1. Issue выбран через [existing_issue_protocol.md](existing_issue_protocol.md) или уже зафиксирован как active issue в [../../State/service_state.md](../../State/service_state.md).
2. `status = active`, `phase = solution` или `phase = execution`; для approved issue без phase сначала выполняется routing через existing issue protocol.
3. `requirements.md` существует, сохранён и имеет `Status: approved` или эквивалентный machine marker.
4. Issue подтверждён как `simple`, либо requalification не выявила необходимости в complex decomposition.
5. Blocking dependencies отсутствуют; stale dependency должен быть проверен до solution approval или execution.
6. Affected files и граница scope перечислены или могут быть перечислены в `solution.md`.
7. Persistence доступен. Если запись невозможна, агент не заявляет, что solution, contract или output стали источником истины.

## Источники истины phase

| Artifact | Роль | Когда создаётся |
|---|---|---|
| `Issues/<issue_id>/solution.md` | proposed способ выполнения и embedded plan | phase `solution` |
| `Issues/<issue_id>/contract.md` | проверяемый критерий готовности и evidence rules | phase `solution`, вместе с solution |
| `Issues/<issue_id>/output/report.md` | сводка фактически выполненной работы | phase `execution` |
| `Issues/<issue_id>/output/changed_files.md` | affected files и краткое описание изменений | phase `execution` |
| `Issues/<issue_id>/output/contract_coverage.md` | покрытие каждого пункта contract evidence | phase `execution` |
| `Issues/<issue_id>/output/attachments_manifest.jsonl` | индекс output attachments, если они есть | phase `execution`, optional but path reserved |

`solution.md` и `contract.md` создаются парой. Отдельный `plan.md` не создаётся: план является разделом `solution.md`, чтобы не дублировать источники истины.

## Phase routing

| Текущее состояние | Разрешённое действие | Следующее состояние |
|---|---|---|
| approved requirements, simple issue | создать `solution.md` и `contract.md` в status `review` | ожидание user approval |
| user approves solution + contract | перевести оба artifact в `approved` | `active: execution` |
| user requests changes | обновить artifact, сохранить approval log | `active: solution` / review |
| approved solution + contract | выполнить work set | `active: execution` |
| output saved and contract covered | перевести к validation | `active: validation` |
| output incomplete | зафиксировать blocker | `active: execution` или return to `solution` / `requirements` |

Переход к execution без approved contract запрещён, кроме случая, где пользователь явно разрешил выполнение без отдельного обсуждения, а contract всё равно сохранён, проверяем и committed.

## Минимальная структура `solution.md`

```text
# Решение — <issue_id>

Parent: [state.md](state.md)
Status: draft | review | approved | reopened
Approved by: user | not_approved
Approved at: <timestamp or null>
Source issue: <issue_id>

## Исходные материалы
- Issue state: ...
- Reason: ...
- Requirements: ...
- QA summary: ...
- Related input: ...

## Карта покрытия требований
| REQ-ID | How solution covers it | Notes |
|---|---|---|
| REQ-001 | ... | ... |

## Предлагаемый подход
...

## План
1. ...
2. ...
3. ...

## Затронутые файлы и объекты
| Path / object | Change type | Owner requirement | Notes |
|---|---|---|---|

## Риски и ограничения
- ...

## Решения пользователя
- none / ...

## Журнал утверждений
| Time | Actor | Action | Notes |
|---|---|---|---|
```

### Quality gate для solution

`solution.md` годится для review только если:

1. каждое approved requirement имеет строку покрытия или explicit exclusion с user reason;
2. план достаточно конкретен для выполнения, но не описывает output как уже сделанный;
3. affected files перечислены настолько точно, насколько позволяет текущий scope;
4. known risks и non-goals не скрыты за обобщёнными формулировками;
5. materially important assumptions вынесены отдельно;
6. нет нового scope, который не прошёл requirements approval.

## Минимальная структура `contract.md`

```text
# Контракт — <issue_id>

Parent: [state.md](state.md)
Status: draft | review | approved | reopened
Approved by: user | not_approved
Approved at: <timestamp or null>
Source issue: <issue_id>

## Критерий готовности
- ...

## Проверки требований
| REQ-ID | Required evidence | Pass condition |
|---|---|---|
| REQ-001 | ... | ... |

## Ожидаемый результат
- report: required / optional
- attachments: required / optional
- changed files: ...

## Условия незакрытия
- ...

## Метод проверки
- manual check / file check / link check / protocol check / user confirmation

## Журнал утверждений
| Time | Actor | Action | Notes |
|---|---|---|---|
```

### Quality gate для contract

Contract нельзя вынести на утверждение, если он:

- не указывает evidence для каждого approved requirement;
- использует непроверяемые формулы вроде “сделать качественно”;
- не говорит, какие файлы или output artifacts должны появиться;
- не содержит conditions of non-closure;
- не различает обязательный output и optional attachments;
- позволяет закрыть issue без validation или user-approved exception.

## Утверждение solution и contract

Пользователь может ответить:

| Ответ | Действие агента |
|---|---|
| `утверждаю solution и contract` | перевести оба artifact в `approved`, обновить registry и phase `execution` |
| `изменить solution ...` | обновить `solution.md`, сохранить approval log, снова вынести на review |
| `изменить contract ...` | обновить `contract.md`, сохранить approval log, снова вынести на review |
| `выполнить без отдельного обсуждения` | разрешено только если contract уже сохранён, проверяем и user intent однозначен |
| `вернуться к requirements` | reopen requirements, зафиксировать reason и stop execution |
| `отложить` | перевести issue в `deferred` или `blocked` с reason |

Если пользователь утверждает только solution без contract, execution не начинается. Для запуска требуется approval обеих частей.

## Execution rules

Перед execution агент обязан:

1. перечитать approved `solution.md`, approved `contract.md`, [../../Issues/issue_registry.jsonl](../../Issues/issue_registry.jsonl), [../../Issues/dependency_graph.jsonl](../../Issues/dependency_graph.jsonl) и relevant target files;
2. проверить, что affected files находятся в approved scope;
3. сформировать work set и write set;
4. выполнить только действия, покрытые solution и contract;
5. если нужен новый scope, остановиться и создать linked issue/backlog entry вместо молчаливого расширения;
6. сохранить output package до ответа пользователю.

## Минимальная структура `output/`

```text
Issues/<issue_id>/output/
├── report.md
├── changed_files.md
├── contract_coverage.md
├── attachments_manifest.jsonl
└── attachments/
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

Каждая строка является одним JSON object:

```json
{"path":"Issues/<issue_id>/output/attachments/<file>","purpose":"...","source":"...","related_requirement":"REQ-...","related_contract_check":"...","status":"kept|generated|external_ref"}
```

Пустой manifest допустим как файл с нулём строк, если attachments не нужны, но если output ссылается на attachment, запись в manifest обязательна.

## Registry и State updates

При создании или изменении artifacts агент обновляет:

1. `solution_path`, `contract_path`, `output_path`, `phase`, `status`, `updated_at` в [../../Issues/issue_registry.jsonl](../../Issues/issue_registry.jsonl);
2. человекочитаемый [../../Issues/issue_registry.md](../../Issues/issue_registry.md), если меняется snapshot;
3. `Issues/<issue_id>/state.md`, если runtime issue folder существует;
4. [../../State/page_registry.jsonl](../../State/page_registry.jsonl), если созданы Markdown-файлы;
5. [../../State/persistence_log.jsonl](../../State/persistence_log.jsonl) после primary artifacts и registry updates;
6. [../../Issues/dependency_graph.jsonl](../../Issues/dependency_graph.jsonl), если output создаёт required artifact или делает зависимые issue stale/ready.

Bootstrap implementation issue без runtime folder могут быть отмечены в registry как `implemented_pending_final_validation`, но runtime artifacts не создаются без фактического runtime-перехода.

## Failure behavior

| Сбой | Статус | Действие |
|---|---|---|
| Нет approved requirements | `blocked_on_requirements` | вернуться к [requirements_protocol.md](requirements_protocol.md) |
| Issue оказался complex | `needs_complex_requalification` | остановить execution и route к [complex_issue_protocol.md](complex_issue_protocol.md) |
| Contract непроверяем | `contract_quality_fail` | переписать contract до review |
| User approval ambiguous | `needs_confirmation` | сохранить draft/review, запросить точное решение |
| Affected files вне scope | `scope_violation_blocked` | вернуть к requirements или создать linked issue через [linked_issues_protocol.md](linked_issues_protocol.md) |
| Dependency stale | `blocked_on_stale_dependency` | обновить dependency readiness и получить required artifact |
| Execution не покрывает contract | `output_contract_gap` | сохранить gap, не закрывать issue, выбрать return phase |
| Нельзя сохранить artifacts | `blocked_on_persistence` | не заявлять review/output как сохранённый источник истины |

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

Если запись не подтверждена, агент возвращает `blocked_on_persistence` или `package_draft_not_committed` с write set.

## Формат ответа после execution output

```text
Сохранено: Issues/<issue_id>/output/.
Phase: validation.
Contract coverage: pass | fail | blocked.
Changed files:
- ...
Следующий шаг: final validation / fix / return to solution / return to requirements.
```

Этот ответ не должен копировать весь `output/report.md`; он показывает только decision packet. Полная версия сохраняется в файлах output package.

## Completion signal

Протокол завершён, когда выполнено одно из условий:

1. `solution.md` и `contract.md` сохранены в status `review` или `approved`;
2. approved solution/contract выполнены, `output/` сохранён, registry/state обновлены и phase переведена к `validation`;
3. работа остановлена с честным blocker, write set и next required action.

После успешного output следующий routing — `common/final_validation` по [catalog](../catalog.md). Если final validation protocol ещё `planned`, агент не закрывает issue, а фиксирует blocker `next_protocol_planned`. После закрытия issue дальнейший archive/tombstone cleanup идёт только через [issue_retention_protocol.md](issue_retention_protocol.md).

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
