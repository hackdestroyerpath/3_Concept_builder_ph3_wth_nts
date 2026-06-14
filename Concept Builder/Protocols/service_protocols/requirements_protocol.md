# Протокол Requirements

Parent: [Каталог протоколов](../catalog.md)  
Owner issue: `EXEC-008`  
Protocol ID: `service/requirements`  
Источник истины: `Protocols/service_protocols/requirements_protocol.md`  
Status: `available`  
Updated: `2026-06-05T09:52:50Z`

## Назначение

`Requirements` — обязательный checkpoint перед requalification, solution, contract, execution и closure выбранного issue. QA помогает уточнить требования, но не является финальным источником истины. Финальный источник истины перед solution — утверждённый `Issues/<issue_id>/requirements.md`.

Требования создаются даже если [Question Answer](question_answer_protocol.md) был пропущен. Это предотвращает расхождение между чатовым обсуждением и файловым источником истины.

Протокол применяется в `Service Mode`. Для будущего `Execution Mode` структура требований будет использовать concept-local issue paths после создания execution-слоя.

## Когда использовать

| Состояние | Действие |
|---|---|
| `active: requirements` | создать, обновить или вынести requirements на утверждение |
| `qa` завершён или пропущен | подготовить draft requirements |
| Пользователь изменил требования | обновить `requirements.md` и approval log |
| Materially important изменение найдено после approval | вернуть phase к `requirements` |
| Requirements уже approved | не менять молча; перейти к requalification/solution routing |

## Обязательные входы

| Вход | Источник |
|---|---|
| Выбранный issue | [../../Issues/issue_registry.jsonl](../../Issues/issue_registry.jsonl) и, если есть, `Issues/<issue_id>/state.md` |
| Reason | registry `reason_summary` или `Issues/<issue_id>/reason.md` |
| Input refs | `input_ref`, Inbox manifest или source material |
| QA trace | `Issues/<issue_id>/qa/` или `qa_skipped_with_reason` |
| Dependency status | [../../Issues/dependency_graph.jsonl](../../Issues/dependency_graph.jsonl) |
| Page registry | [../../State/page_registry.jsonl](../../State/page_registry.jsonl) |
| Persistence rules | [../common/persistence_protocol.md](../common/persistence_protocol.md) |
| Protocol catalog | [../catalog.md](../catalog.md) |

## Preconditions

1. Issue выбран и имеет `status = active`, `phase = requirements`, либо перевод в эту phase уже записан через [existing_issue_protocol.md](existing_issue_protocol.md).
2. QA либо завершён, либо явно пропущен с reason, либо не нужен для первого draft.
3. Blocking dependencies не мешают формулированию requirements.
4. Требования можно сохранить в runtime issue folder или честно зафиксировать невозможность записи.
5. Нет другого утверждённого `requirements.md`, который агент собирается молча заменить; история изменений должна фиксироваться в approval log.

## Источник истины

Основной файл:

```text
Issues/<issue_id>/requirements.md
```

Дополнительный machine mirror `requirements.jsonl` допустим только если дерево требований стало слишком большим для удобного Markdown. `requirements.md` остаётся главным читаемым источником истины.

## Минимальная структура `requirements.md`

```text
# Требования — <issue_id>

Parent: [state.md](state.md)
Status: draft | review | approved | reopened
Approved by: user | not_approved
Approved at: <timestamp or null>
Source issue: <issue_id>

## Исходные материалы
- Issue state: ...
- Reason: ...
- Input: ...
- QA summary: ...

## Дерево требований
| REQ-ID | Parent | Requirement | Source | Status | Acceptance note |
|---|---|---|---|---|---|
| REQ-001 | none | ... | reason/input/QA/user/agent_analysis | draft | ... |

## Исключения и нецели
- ...

## Допущения и риски
- ...

## Журнал утверждений
| Time | Actor | Action | Notes |
|---|---|---|---|
```

Каждое требование должно быть проверяемым. Непроверяемые формулировки вроде “сделать нормально” не являются requirement.

## Поля requirement

| Field | Значение |
|---|---|
| `REQ-ID` | стабильный ID, например `REQ-001` |
| `Parent` | parent requirement или `none` |
| `Requirement` | проверяемая формулировка |
| `Source` | `issue`, `reason`, `input`, `QA`, `user_assertion`, `agent_analysis`, `dependency` |
| `Status` | `draft`, `review`, `approved`, `removed`, `changed` |
| `Acceptance note` | как понять, что requirement учтён |

ID требования не переиспользуется после удаления. Removed requirement остаётся в журнале или таблице со status `removed`, если на него уже ссылались QA, solution или contract.

## Порядок создания draft

1. **Focus read**: прочитать issue entry, reason, input refs, QA summary и dependency status.
2. **Source map**: перечислить, откуда берётся каждое требование.
3. **Draft tree**: сформировать requirements tree с ID, source и acceptance note.
4. **Non-goals**: явно вынести исключения, чтобы scope не расширялся без approval.
5. **Risk assumptions**: записать допущения, которые не блокируют draft, но могут вернуть phase к requirements позже.
6. **Persistence**: сохранить `requirements.md`, обновить issue registry paths/status и [../../State/page_registry.jsonl](../../State/page_registry.jsonl), если файл создан.
7. **Review packet**: показать пользователю краткое дерево требований на утверждение.

## Утверждение требований

Пользователь может:

| Ответ | Действие агента |
|---|---|
| `утверждаю requirements` / `утвердить` | перевести `requirements.md` в `approved` |
| `добавить REQ ...` | добавить requirement и снова вынести на review |
| `изменить REQ-...` | пометить старую формулировку `changed`, записать новую версию |
| `убрать REQ-...` | пометить requirement `removed` с reason |
| `вопрос по REQ-...` | ответить локально или вернуть phase к QA, если нужен новый materially important answer |

После утверждения:

1. `requirements.md` получает `Status: approved`;
2. registry поле `requirements_path` указывает на файл;
3. issue phase переходит к `requalification` или `solution` по [catalog](../catalog.md);
4. агент не меняет requirements молча;
5. materially important изменение возвращает phase к `requirements` и добавляет запись в approval log.

## Правило пропуска QA

Если [question_answer_protocol.md](question_answer_protocol.md) не запускался, в `requirements.md` обязательно фиксируется:

```text
QA: skipped
Skip reason: <почему вопросов не требовалось>
Risk: <какие допущения остаются, если есть>
```

Пропуск без reason запрещён. Иначе contract coverage теряет проверяемое основание для отсутствующих QA-ответов.

## Обновление уже существующих требований

Если `requirements.md` существует:

1. перечитать текущий файл и issue state;
2. определить, меняется ли scope, acceptance note, non-goals или risk;
3. сохранить изменение в approval log;
4. не удалять старые requirement IDs, если на них ссылаются downstream artifacts;
5. если approved requirements меняются, вернуть phase к `requirements` и запросить новое утверждение.

## Requalification handoff

После approved requirements агент проверяет тип issue:

| Условие | Действие |
|---|---|
| Requirements можно выполнить одним work package | оставить `type = simple` и перейти к solution |
| Requirements требуют независимых children, разные outputs или dependency sequencing | предложить requalification в `complex` через [complex_issue_protocol.md](complex_issue_protocol.md) |
| Тип меняется | запросить утверждение пользователя, не менять молча |
| Нужна dependency-связь без parent/child | route к [linked_issues_protocol.md](linked_issues_protocol.md) |

На этом шаге `solution/contract/output` не выполняются. Этот протокол доводит issue только до approved requirements и next routing к [complex_issue_protocol.md](complex_issue_protocol.md), [linked_issues_protocol.md](linked_issues_protocol.md) или [solution_contract_output_protocol.md](solution_contract_output_protocol.md), если issue остаётся simple.

## Формат ответа на review

```text
Сохранено: Issues/<issue_id>/requirements.md.
Status: review.
Краткое дерево требований:
| ID | Requirement | Acceptance note |
|---|---|---|
| REQ-001 | ... | ... |

Доступные ответы: утвердить / добавить / изменить / убрать / вопрос по REQ-ID.
```

Если запись не подтверждена, агент возвращает `blocked_on_persistence` или `package_draft_not_committed` с write set и не заявляет, что requirements вынесены на утверждение.

## Failure behavior

| Сбой | Статус | Действие |
|---|---|---|
| QA содержит unresolved blocking unknown | `blocked_on_qa_unknown` | вернуться к [question_answer_protocol.md](question_answer_protocol.md) |
| Нет reason/source | `blocked_on_missing_source` | вернуться к intake/reason repair |
| Dependency stale влияет на requirements | `blocked_on_stale_dependency` | обновить dependency readiness и запросить required artifact |
| Requirements conflict | `needs_requirements_decision` | показать конфликт и варианты merge/split/defer |
| User approval ambiguous | `needs_confirmation` | сохранить draft, запросить точное решение |
| Нельзя записать файл | `blocked_on_persistence` | не считать requirements источником истины |

## Completion signal

Протокол завершён, когда `requirements.md` сохранён и вынесен на review, утверждён пользователем, либо работа остановлена с честным blocker. При approval следующий routing — requalification или solution по [catalog](../catalog.md); для simple issue следующий phase-протокол — [solution_contract_output_protocol.md](solution_contract_output_protocol.md).

## Связанные файлы

- [Catalog](../catalog.md)
- [Question Answer protocol](question_answer_protocol.md)
- [Existing issue protocol](existing_issue_protocol.md)
- [Solution / Contract / Output protocol](solution_contract_output_protocol.md)
- [Complex Issue protocol](complex_issue_protocol.md)
- [Linked Issues protocol](linked_issues_protocol.md)
- [New issue protocol](new_issue_protocol.md)
- [Service state](../../State/service_state.md)
- [Issue registry](../../Issues/issue_registry.md)
- [Issue registry JSONL](../../Issues/issue_registry.jsonl)
- [Dependency graph](../../Issues/dependency_graph.jsonl)
- [Inbox rules](../../Inbox/README.md)
- [Page registry](../../State/page_registry.jsonl)
- [Persistence protocol](../common/persistence_protocol.md)
