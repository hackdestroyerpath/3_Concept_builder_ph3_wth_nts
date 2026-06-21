# Каталог протоколов

Parent: [README](../README.md)  
Owner issue: `EXEC-003`  
Источник истины: `Protocols/catalog.md`  
Machine companion: [catalog.jsonl](catalog.jsonl)  
Status: `available`  
Updated: `2026-06-21T22:15:00Z`

## Назначение

Каталог является источником истины выбора протокола. Агент определяет `mode`, `state`, `phase`, active `issue` и trigger, затем загружает самый локальный available protocol. Planned/deferred route не исполняется как available.

## Правило выбора

1. Прочитать root [README](../README.md) и relevant state: [Service state](../State/service_state.md) либо [Execution index](../State/execution_index.md).
2. Проверить [page registry](../State/page_registry.jsonl), чтобы не создать Markdown-сироту.
3. Для issue scope проверить machine registry `Issues/issue_registry.jsonl` и [dependency graph](../Issues/dependency_graph.jsonl).
4. Найти trigger в таблице.
5. Открыть только available protocol и прямые зависимости.
6. Deferred/planned protocol не имитировать: сохранить owner, reason и next action.
7. При нескольких route выбрать самый локальный; context lift требует reason.

## Доступные протоколы

| Protocol ID | Файл | Mode | Когда использовать | Completion signal |
|---|---|---|---|---|
| `common/context_loading` | [common/context_loading_protocol.md](common/context_loading_protocol.md) | `common` | старт/restore/focus packet | минимальный context packet и next-step marker |
| `common/persistence_transaction` | [common/persistence_protocol.md](common/persistence_protocol.md) | `common` | production write или package draft | commit marker или truthful pending/blocked status |
| `common/final_validation` | [common/final_validation_protocol.md](common/final_validation_protocol.md) | `common` | issue/export closure, pre-commit, final acceptance, North Star self-check | named evidence report, contract coverage и residual decision |
| `service/service_start` | [service_protocols/service_start_protocol.md](service_protocols/service_start_protocol.md) | `service` | старт Service Mode | next protocol или blocker |
| `service/new_issue` | [service_protocols/new_issue_protocol.md](service_protocols/new_issue_protocol.md) | `service` | intake нового issue | input/registry artifacts или blocker |
| `service/existing_issue` | [service_protocols/existing_issue_protocol.md](service_protocols/existing_issue_protocol.md) | `service` | выбор/продолжение issue | focus packet и blockers |
| `service/question_answer` | [service_protocols/question_answer_protocol.md](service_protocols/question_answer_protocol.md) | `service/execution` | materially important unknowns | QA trace или skip/blocker |
| `service/requirements` | [service_protocols/requirements_protocol.md](service_protocols/requirements_protocol.md) | `service/execution` | requirements draft/review/approval | review/approved/blocker |
| `service/solution_contract_output` | [service_protocols/solution_contract_output_protocol.md](service_protocols/solution_contract_output_protocol.md) | `service/execution` | solution/contract/output | saved artifacts или blocker |
| `service/complex_issue` | [service_protocols/complex_issue_protocol.md](service_protocols/complex_issue_protocol.md) | `service/execution` | requalification/decomposition | child entries или blocker |
| `service/linked_issues` | [service_protocols/linked_issues_protocol.md](service_protocols/linked_issues_protocol.md) | `service/execution` | dependency edges/readiness | edge state или rejection reason |
| `service/issue_retention` | [service_protocols/issue_retention_protocol.md](service_protocols/issue_retention_protocol.md) | `service/execution` | archive/tombstone/delete/Inbox cleanup | retention decision |
| `execution/index` | [execution_protocols/README.md](execution_protocols/README.md) | `execution` | startup/select/create/continue/recovery/escalation | startup case, focus, readiness/recovery marker |
| `execution/concept_export` | [execution_protocols/concept_export_protocol.md](execution_protocols/concept_export_protocol.md) | `execution` | precheck/WIP/closed export/local-open diagnosis | read-only precheck, persisted package или exact blocker |

## Selection matrix

| Trigger / состояние | Mode | Protocol ID | Статус | Обязательные входы | Выходы | Следующий state / phase |
|---|---|---|---|---|---|---|
| Service start: `пинг`, `старт`, `1` | `service` | `service/service_start` | `available` | README, service state, catalog, issue registry | loaded status/navigation | new/existing issue route |
| Execution start/select/create/continue/recovery | `execution` | `execution/index` | `available` | README, execution index, registry, Concepts entry/template, optional local state | startup/focus/bootstrap/recovery/escalation | concept work/bootstrap/blocker |
| Context restore/focus packet | `service/execution` | `common/context_loading` | `available` | entry, state, registry, catalog | loaded status/focus | selected local protocol |
| Любая production-запись | `common` | `common/persistence_transaction` | `available` | latest state, targets, write set, ref | changes/readback/log | committed или pending |
| Новый service issue | `service` | `service/new_issue` | `available` | input, attachments, Inbox rules | input + registry/issue artifacts | proposed/blocker |
| Existing issue focus | `service` | `service/existing_issue` | `available` | registry, graph, optional artifacts | shortlist/focus packet | focused/blocker |
| Unknowns before requirements | `service/execution` | `service/question_answer` | `available` | issue, reason, input | QA/skip/blocker | requirements |
| Requirements work | `service/execution` | `service/requirements` | `available` | issue/input/QA/assertions | requirements/review | requalification/solution |
| Requalification/decomposition | `service/execution` | `service/complex_issue` | `available` | approved requirements, graph | decision/children | solution/child focus |
| Solution/contract/output | `service/execution` | `service/solution_contract_output` | `available` | approved requirements/solution/contract, affected paths | artifacts/coverage | validation/blocker |
| Dependency edge/readiness | `service/execution` | `service/linked_issues` | `available` | source/target registry rows, graph, evidence | normalized readiness | ready/blocked/stale/cycle |
| Retention/cleanup | `service/execution` | `service/issue_retention` | `available` | registry, graph, archive/tombstone/Inbox | decision + updates | retained/blocked |
| Export precheck/WIP/closed package | `execution` | `execution/concept_export` | `available` | optional selected concept for precheck; verified local state for mutation | precheck/package/blocker | exported/precheck/blocked |
| Issue/export closure, pre-commit или final acceptance | `service/execution` | `common/final_validation` | `available` | state, registries, graph, catalog, affected files, export evidence if applicable, approved audit/transfer registry for North Star scope | named evidence matrix, contract coverage, 74-ID dispositions, residual/Stage 07 decision | review-ready/repair; `closure_allowed=false` for branch candidate |
| Оценка service script | `service` | `service/script_evaluation` | `deferred` | structural backlog, usage signal, cost/benefit owner | decision record | approved issue или remain deferred |

### Export routing contract

- `export_precheck` read-only и может вернуть `blocked_no_concept` без mutation;
- package mutation требует selected concept, readable revision, verified integrity и parseable local registries;
- closed route блокируют identity/readiness/output/registry/dependency/validation/language/package/local-open/collision/persistence failures;
- WIP всегда `not_final`; portable defects требуют exact limitations, `draft_with_notice` и acceptance;
- completion: precheck без mutation, persisted/read-back package либо exact blocker.

### Final-acceptance routing contract

- final acceptance использует `common/final_validation`, а не новый protocol;
- evidence matrix разделяет checked и changed paths и не принимает status labels как proof;
- runtime issue closure требует reason/input, requirements, QA/skip, requalification, solution, contract, output/report, contract coverage и validation либо approved exception;
- North Star scope map-ит все 74 IDs ровно один раз;
- P0/P1 residual блокирует candidate; P2/P3 defer требует owner/reason/next action;
- `DO_NOT_TRANSFER` фиксируется как `rejected_with_reason`;
- branch candidate может вывести `stage_07_candidate=skip_no_residuals`, но не запускает Stage 07, не мержит PR и сохраняет `closure_allowed=false`.

## Связанные реестры issue и state

| Ресурс | Файл/путь | Когда читать |
|---|---|---|
| Человекочитаемый issue registry | [Issues/issue_registry.md](../Issues/issue_registry.md) | lifecycle/status/guards |
| Machine issue registry | `Issues/issue_registry.jsonl` | filtering/dependency refs/next action |
| Dependency graph | [Issues/dependency_graph.jsonl](../Issues/dependency_graph.jsonl) | blockers/cycles/readiness |
| Concepts entry | [Concepts/README.md](../Concepts/README.md) | execution start/create/select |
| Concept template | [Concepts/_template/README.md](../Concepts/_template/README.md) | concept creation |
| Inbox rules | `Inbox/README.md` | intake/cleanup |
| Issue archive | `Issues/_archive/README.md` | archive decision |
| Issue tombstones | `Issues/_tombstones/README.md` | cleanup decision |
| Service validation report | `State/service_validation_report.md` | commit/export/final acceptance |

## Deferred protocol candidates

| Protocol ID | Owner | Reason | Next action |
|---|---|---|---|
| `service/script_evaluation` | Service Mode / `USER-001` | scripts требуют usage signal и cost/benefit | отдельный approved issue после реального signal |
| `common/issue_decision_update` | future service issue | выделять только при доказанной тяжести current flow | оставить deferred |
| `common/response_packet` | future service issue | выделять только при реальном drift response format | оставить deferred |

Project instruction sources: `Instructions/service_mode_project_instructions.md`, `Instructions/execution_mode_project_instructions.md`.

## Обновление каталога

Новый protocol требует существующего file, human row, machine row, parent, factual page-registry update и persistence marker. Перед closure выполняется [final validation protocol](common/final_validation_protocol.md). Planned/deferred path не становится available без этих evidence.
