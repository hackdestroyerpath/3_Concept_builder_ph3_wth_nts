# Протокол Requirements

Parent: [Каталог протоколов](../catalog.md)  
Owner issue: `EXEC-008` / `USER-005`  
Protocol ID: `service/requirements`  
Источник истины: `Protocols/service_protocols/requirements_protocol.md`  
Status: `available`  
Updated: `2026-06-18T01:29:00Z`

## Назначение

`Requirements` — обязательный checkpoint перед requalification, solution, contract, execution, validation и closure выбранного service issue. QA, чат и agent analysis могут быть источниками, но финальным источником истины перед solution является только сохранённый `Issues/<issue_id>/requirements.md`.

Требования создаются даже если QA была пропущена. Это предотвращает ситуацию, где выполнение начинается по устному контексту, а future agent не может проверить, почему work set вообще допустим. Какая удивительная идея: записывать требования до выполнения. Почти цивилизация.

## Когда использовать

| Состояние | Действие |
|---|---|
| `active: requirements` | создать, обновить или вынести requirements на review |
| QA завершён | перенести resolved answers в requirement IDs |
| QA пропущен | зафиксировать skip reason и residual risks |
| Пользователь меняет требования | обновить `requirements.md` и approval log |
| Materially important change найден после approval | reopen requirements и остановить downstream execution |
| Requirements approved | не менять молча; route к requalification/solution |

## Обязательные входы

| Вход | Источник |
|---|---|
| Выбранный issue | [../../Issues/issue_registry.jsonl](../../Issues/issue_registry.jsonl), registry summary и, если есть, `Issues/<issue_id>/state.md` |
| Reason | `Issues/<issue_id>/reason.md` или registry `reason_summary` |
| Input refs | registry `input_ref`, Inbox manifest или source material |
| QA trace | `Issues/<issue_id>/qa/`, QA summary или `qa_skipped_with_reason` |
| Dependency status | [../../Issues/dependency_graph.jsonl](../../Issues/dependency_graph.jsonl) |
| Existing focus packet | [existing_issue_protocol.md](existing_issue_protocol.md) |
| Persistence rules | [../common/persistence_protocol.md](../common/persistence_protocol.md) |
| Protocol catalog | [../catalog.md](../catalog.md) |

## Preconditions

1. Issue выбран и имеет `status = active`, `phase = requirements`, либо перевод в эту phase уже записан через [existing_issue_protocol.md](existing_issue_protocol.md).
2. Reason/source существуют. Если reason слабый или отсутствует, stop: `blocked_on_missing_source`.
3. QA завершён, явно пропущен с reason, либо определено, что QA не нужна для первого draft.
4. Blocking dependencies не мешают формулированию requirements. Stale dependency, влияющий на требования, блокирует approval.
5. Нет другого approved `requirements.md`, который агент собирается молча заменить.
6. Runtime issue folder создаётся только если сейчас реально пишется `requirements.md`; пустая папка не создаётся.

## Source-of-truth rule

Основной файл:

```text
Issues/<issue_id>/requirements.md
```

Дополнительный machine mirror `requirements.jsonl` допустим только как derived mirror. Markdown остаётся главным читаемым источником. Downstream `solution.md`, `contract.md`, `output/` и validation читают approved `requirements.md`, а не чатовые формулировки.

Если запись `requirements.md` не подтверждена, агент не заявляет, что requirements вынесены на review или approved.

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

## Stage 04 checklist

Requirements packet нельзя считать готовым к review, пока не выполнены все пункты:

| Gate | Pass condition |
|---|---|
| Source reason | каждое top-level требование связано с reason, input, QA, user decision или dependency |
| QA trace | QA summary указан или есть explicit skip reason с risk note |
| Requirement IDs | каждый requirement имеет стабильный `REQ-...`; ID не переиспользуется |
| Acceptance notes | каждое требование имеет проверяемое acceptance note |
| Non-goals | исключения и границы scope записаны явно |
| Approval log | любое утверждение, reopen, change/remove фиксируется строкой журнала |
| Reopen behavior | materially important change возвращает status к `reopened` / phase `requirements` |

Непроверяемые формулировки вроде `сделать нормально` не являются requirement. Они превращаются в вопрос, risk или blocker, потому что магия всё ещё не входит в dependency model.

## Поля requirement

| Field | Значение |
|---|---|
| `REQ-ID` | стабильный ID, например `REQ-001` |
| `Parent` | parent requirement или `none` |
| `Requirement` | проверяемая формулировка |
| `Source` | `issue`, `reason`, `input`, `QA`, `user_assertion`, `agent_analysis`, `dependency` |
| `Status` | `draft`, `review`, `approved`, `removed`, `changed` |
| `Acceptance note` | как понять, что requirement учтён |

Removed requirement остаётся в таблице или approval log, если на него уже ссылались QA, solution, contract, output или dependency edge.

## Порядок создания draft

1. **Focus read**: прочитать registry row, state если есть, reason, input refs, QA trace и dependency status.
2. **Source map**: перечислить source для каждого requirement.
3. **Draft tree**: сформировать tree с ID, parent, source и acceptance note.
4. **Non-goals**: вынести excluded scope, чтобы solution не расширился без approval.
5. **Risk assumptions**: записать допущения и stale dependency risks.
6. **Persistence**: сохранить `requirements.md`, обновить registry paths/status и [../../State/page_registry.jsonl](../../State/page_registry.jsonl), если создан Markdown-файл.
7. **Review packet**: показать пользователю краткое дерево требований и допустимые ответы.

## Утверждение требований

| Ответ пользователя | Действие агента |
|---|---|
| `утверждаю requirements` / `утвердить` | перевести `requirements.md` в `approved` |
| `добавить REQ ...` | добавить requirement и вернуть status к review |
| `изменить REQ-...` | пометить старую формулировку `changed`, записать новую версию |
| `убрать REQ-...` | пометить requirement `removed` с reason |
| `вопрос по REQ-...` | ответить локально или route к QA, если нужен materially important answer |

После approval:

1. `requirements.md` получает `Status: approved`;
2. registry `requirements_path` указывает на файл;
3. phase переходит к `requalification` или `solution` по [catalog](../catalog.md);
4. агент не меняет approved requirements молча;
5. materially important изменение возвращает issue к `requirements` и добавляет approval-log row.

## Обновление существующих requirements

Если `requirements.md` уже существует:

1. перечитать текущий файл, registry и state;
2. определить, меняется ли scope, acceptance note, non-goal или risk;
3. сохранить изменение в approval log;
4. не удалять IDs, если downstream artifacts уже ссылались на них;
5. если approved requirements меняются, reopen и запросить новое утверждение.

## Requalification handoff

После approved requirements агент проверяет тип issue:

| Условие | Действие |
|---|---|
| Один work package, один output, один contract | оставить `simple` и route к solution |
| Независимые outputs, children или dependency sequencing | предложить `complex` через [complex_issue_protocol.md](complex_issue_protocol.md) |
| Нужна внешняя dependency без parent/child | route к [linked_issues_protocol.md](linked_issues_protocol.md) |
| Тип меняется | запросить user approval, не менять молча |

Этот протокол не выполняет solution/contract/output. Он доводит issue до requirements review/approval или честного blocker.

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

Если запись не подтверждена, агент возвращает `blocked_on_persistence` или package draft с write set и не заявляет committed review state.

## Failure behavior

| Сбой | Статус | Действие |
|---|---|---|
| QA содержит unresolved blocking unknown | `blocked_on_qa_unknown` | вернуться к [question_answer_protocol.md](question_answer_protocol.md) |
| Нет reason/source | `blocked_on_missing_source` | вернуться к intake/reason repair |
| Dependency stale влияет на requirements | `blocked_on_stale_dependency` | обновить dependency readiness и получить required artifact |
| Requirements conflict | `needs_requirements_decision` | показать конфликт и варианты merge/split/defer |
| User approval ambiguous | `needs_confirmation` | сохранить draft/review, запросить точное решение |
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
