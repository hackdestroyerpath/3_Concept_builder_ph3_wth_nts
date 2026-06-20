# Протокол Requirements

Parent: [Каталог протоколов](../catalog.md)  
Owner issue: `EXEC-008` / `USER-005`  
Protocol ID: `service/requirements`  
Источник истины: `Protocols/service_protocols/requirements_protocol.md`  
Status: `available`  
Updated: `2026-06-19T13:20:00Z`

## Назначение

`Requirements` — обязательный checkpoint перед requalification, solution, contract, execution, validation и closure выбранного service issue. Финальным источником истины перед solution является сохранённый `Issues/<issue_id>/requirements.md`; чат, QA и agent analysis выступают только источниками требований.

Требования сохраняются и при пропущенном QA, чтобы scope и критерии оставались проверяемыми вне чата.

## Обязательные входы

| Вход | Источник |
|---|---|
| Выбранный issue | [../../Issues/issue_registry.jsonl](../../Issues/issue_registry.jsonl) и существующий `Issues/<issue_id>/state.md` |
| Reason | `Issues/<issue_id>/reason.md` или registry `reason_summary` |
| Input refs | registry `input_ref`, Inbox manifest или source material |
| QA trace | `Issues/<issue_id>/qa/`, QA summary или `qa_skipped_with_reason` |
| Dependency status | [../../Issues/dependency_graph.jsonl](../../Issues/dependency_graph.jsonl) |
| Existing focus | [existing_issue_protocol.md](existing_issue_protocol.md) |
| Persistence rules | [../common/persistence_protocol.md](../common/persistence_protocol.md) |

## Preconditions

1. Issue выбран и имеет `status = active`, `phase = requirements`, либо такой переход уже сохранён.
2. Reason/source существует; иначе применяется `blocked_on_missing_source`.
3. QA завершён, пропущен с reason или признан ненужным для первого draft.
4. Stale dependency, влияющий на требования, блокирует approval.
5. Approved requirements не заменяются без reopen и нового решения пользователя.
6. Runtime folder создаётся только при фактической записи phase artifact; пустая папка запрещена.

## Source of truth

```text
Issues/<issue_id>/requirements.md
```

`requirements.jsonl` допустим только как derived mirror. Downstream artifacts используют approved Markdown, а не чатовые формулировки. Неподтверждённая запись не считается review или approval state.

## Минимальная структура

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
- QA trace: completed | skipped | not_required
- QA skip reason: ...

## Дерево требований
| REQ-ID | Parent | Requirement | Source | Status | Acceptance note |
|---|---|---|---|---|---|
| REQ-001 | none | ... | reason/input/QA/user/agent_analysis/dependency | draft | ... |

## Исключения и нецели
- ...

## Допущения и риски
- ...

## Журнал утверждений
| Time | Actor | Action | Notes |
|---|---|---|---|
```

## Проверка готовности requirements packet

| Gate | Pass condition |
|---|---|
| Source reason | каждое top-level требование связано с reason, input, QA, user decision или dependency |
| QA trace | указан QA summary либо explicit skip reason с risk note |
| Requirement IDs | каждый requirement имеет стабильный `REQ-...`; ID не переиспользуется |
| Acceptance notes | каждое требование имеет проверяемый acceptance note |
| Non-goals | границы scope записаны явно |
| Approval log | approval, reopen, change и removal фиксируются в журнале |
| Reopen behavior | materially important change возвращает status к `reopened` и phase к `requirements` |

Непроверяемая формулировка превращается в вопрос, risk или blocker, а не в requirement.

## Создание и обновление

1. Читаются registry row, state, reason, input refs, QA trace и dependency status.
2. Для каждого requirement фиксируются source, stable ID и acceptance note.
3. Non-goals и assumptions отделяются от обязательных требований.
4. `requirements.md` сохраняется раньше registry/page-registry mirrors.
5. После подтверждённой записи packet получает status `review`.

При изменении существующего файла сохраняется approval history. Использованные downstream IDs не удаляются: прежняя формулировка получает `changed` или `removed`, а новая версия фиксируется отдельно.

## Approval и reopen

Approval переводит файл в `Status: approved`, заполняет actor/time, сохраняет `requirements_path` и направляет issue к requalification либо solution. Тип issue не меняется молча.

Materially important изменение approved scope, acceptance note, non-goal или risk:

- переводит файл в `reopened`;
- возвращает issue в phase `requirements`;
- останавливает downstream execution;
- добавляет строку approval log;
- требует нового approval.

## Requalification handoff

| Условие | Routing |
|---|---|
| Один work package, один output и один contract | simple issue → [solution_contract_output_protocol.md](solution_contract_output_protocol.md) |
| Независимые outputs, children или sequencing | предложение complex через [complex_issue_protocol.md](complex_issue_protocol.md) |
| Внешняя dependency без parent/child | [linked_issues_protocol.md](linked_issues_protocol.md) |

## Failure behavior

| Сбой | Статус | Действие |
|---|---|---|
| Unresolved blocking QA unknown | `blocked_on_qa_unknown` | вернуться к [question_answer_protocol.md](question_answer_protocol.md) |
| Нет reason/source | `blocked_on_missing_source` | repair intake/reason |
| Dependency stale | `blocked_on_stale_dependency` | обновить readiness и required artifact |
| Requirements conflict | `needs_requirements_decision` | merge/split/defer decision |
| Approval ambiguous | `needs_confirmation` | сохранить review state без approval |
| Persistence недоступен | `blocked_on_persistence` | не объявлять requirements source of truth |

## Completion signal

Протокол завершён после подтверждённой записи review/approved state либо после сохранения честного blocker. После approval применяется requalification или [Solution / Contract / Output protocol](solution_contract_output_protocol.md).

## Связанные файлы

- [Catalog](../catalog.md)
- [Question Answer protocol](question_answer_protocol.md)
- [Existing issue protocol](existing_issue_protocol.md)
- [Solution / Contract / Output protocol](solution_contract_output_protocol.md)
- [Complex Issue protocol](complex_issue_protocol.md)
- [Linked Issues protocol](linked_issues_protocol.md)
- [Service state](../../State/service_state.md)
- [Issue registry](../../Issues/issue_registry.md)
- [Issue registry JSONL](../../Issues/issue_registry.jsonl)
- [Dependency graph](../../Issues/dependency_graph.jsonl)
- [Inbox rules](../../Inbox/README.md)
- [Page registry](../../State/page_registry.jsonl)
- [Persistence protocol](../common/persistence_protocol.md)
