# Реестр issue

Parent: [README](../README.md)  
Owner issue: `EXEC-005`  
Источник истины: `Issues/issue_registry.md`  
Machine companion: [issue_registry.jsonl](issue_registry.jsonl)  
Dependency graph: [dependency_graph.jsonl](/dependency_graph.jsonl)  
Status: `validated_pending_current_main_repair`  
Updated: `2026-06-25T22:04:48+02:00`

## Назначение

Этот файл — человекочитаемый вход в root issue-модель `Concept Builder`. Machine source — `issue_registry.jsonl`, dependency edges — `dependency_graph.jsonl`. Runtime issue folders не создаются заранее; они появляются только когда реальный protocol требует phase artifacts.

## Status policy

| Status | Значение |
|---|---|
| `closed` | issue реализован и покрыт финальной проверкой |
| `deferred` | issue отложен с owner, reason и next action |
| `closed_as_continuous_guard` | разовое issue закрыто как постоянный guard |
| `blocked` | нужен repair step |

## Coverage snapshot

| ID | Class | Status | Dependency ready | Coverage / decision | Основной output |
|---|---|---|---|---|---|
| `EXEC-001` | `implementation` | `closed` | `not_applicable` | Закрыто accepted root validation; новые изменения требуют approved issue. | `README.md` |
| `EXEC-002` | `implementation` | `closed` | `ready` | Project instruction loaders приняты. | `Instructions/service_mode_project_instructions.md` |
| `EXEC-003` | `implementation` | `closed` | `ready` | Catalog и routing приняты. | `Protocols/catalog.md` |
| `EXEC-004` | `implementation` | `closed` | `ready` | State/persistence model принята. | `State/service_state.md` |
| `EXEC-005` | `implementation` | `closed` | `ready` | Root issue model принята. | `Issues/issue_registry.md` |
| `EXEC-006` | `implementation` | `closed` | `ready` | New issue intake принят. | `Protocols/service_protocols/service_start_protocol.md` |
| `EXEC-007` | `implementation` | `closed` | `ready` | Existing issue route принят. | `Protocols/service_protocols/existing_issue_protocol.md` |
| `EXEC-008` | `implementation` | `closed` | `ready` | QA/requirements workflow принят. | `Protocols/service_protocols/question_answer_protocol.md` |
| `EXEC-009` | `implementation` | `closed` | `ready` | Solution/contract/output workflow принят. | `Protocols/service_protocols/solution_contract_output_protocol.md` |
| `EXEC-010` | `implementation` | `closed` | `ready` | Complex/linked workflow принят. | `Protocols/service_protocols/complex_issue_protocol.md` |
| `EXEC-011` | `implementation` | `closed` | `ready` | Execution/concept/export layer принята. | `Protocols/execution_protocols/README.md` |
| `EXEC-012` | `implementation` | `closed` | `ready` | Stage 06 implementation принята через merged PR `#21`; current-main residual repair и отдельный final closure transition ещё ожидаются. | `Protocols/common/final_validation_protocol.md` |
| `USER-001` | `user-noted` | `deferred` | `ready` | Nonblocking: scripts только по отдельному cost/benefit issue. | `Issues/CB-SVC-001-script-assessment/` |
| `USER-002` | `user-noted` | `closed` | `ready` | State layout покрыт. | `State/service_state.md` |
| `USER-003` | `user-noted` | `closed` | `ready` | Execution Mode покрыт. | `Protocols/execution_protocols/README.md` |
| `USER-004` | `user-noted` | `closed` | `ready` | Add-during-approval покрыт. | `Protocols/service_protocols/new_issue_protocol.md` |
| `USER-005` | `user-noted` | `closed` | `ready` | Requirements/QA storage покрыт. | `Protocols/service_protocols/requirements_protocol.md` |
| `USER-006` | `user-noted` | `closed` | `ready` | Retention lifecycle покрыт. | `Protocols/service_protocols/issue_retention_protocol.md` |
| `USER-007` | `user-noted` | `closed` | `ready` | Export policy покрыта. | `Protocols/execution_protocols/concept_export_protocol.md` |
| `OPT-001` | `optimizer-detected` | `closed_as_continuous_guard` | `not_applicable` | Production/development boundary. | `State/service_validation_report.md` |
| `OPT-002` | `optimizer-detected` | `closed_as_continuous_guard` | `not_applicable` | Operational language/style guard. | `Protocols/common/final_validation_protocol.md` |
| `OPT-003` | `optimizer-detected` | `closed_as_continuous_guard` | `ready` | Context budget/local focus. | `State/service_state.md` |
| `OPT-004` | `optimizer-detected` | `closed_as_continuous_guard` | `ready` | Link/backlink/orphan guard. | `State/page_registry.jsonl` |

## Current bounded residual repair

`RES-CLOSURE-001` и `RES-LANG-001` являются current-main repair residuals, а не новыми North Star source IDs и не synthetic runtime issue. Их branch candidate находится в `agent/cbu-current-main-truth-language-repair-20260622` и остаётся `pending_manual_reviewer`.

```yaml
base_main: 194970c7c5ac37f2dbbfcd51256caaa46f67f8f9
base_residual_ids: [RES-CLOSURE-001, RES-LANG-001]
candidate_residual_ids: []
stage_07_status: repair_candidate_pending_manual_reviewer
closure_candidate: true
closure_allowed: false
```

Candidate fields применимы только после exact branch readback. Merge и final closure не выполнены.

## Deferred items

| ID | Owner | Reason | Next action |
|---|---|---|---|
| `USER-001` | Service Mode | Scripts не входят в production manifest без usage signal и cost/benefit decision. | Создать отдельный approved issue после реального service usage signal. |

## Continuous guards

| ID | Guard | Owner / trigger | Canonical enforcement paths | Failure action |
|---|---|---|---|---|
| `OPT-001` | Production/development boundary: task-state, prompt, handoff/archive и methodology не являются runtime production. | Repository boundary; перед любой production mutation и final acceptance. | `README.md`; `State/page_registry.jsonl`; `State/service_validation_report.md`; `Protocols/common/final_validation_protocol.md` | `blocked`; исключить dev-only paths, перечитать scope и повторить boundary audit. |
| `OPT-002` | Readable agent-oriented language/style: основной operational text на русском; technical tokens разрешены. | Language policy; при изменении readable Markdown. | `Protocols/common/final_validation_protocol.md` | `blocked` только при operational unreadability, semantic contradiction или explicit contract violation; cosmetic sweep не запускать. |
| `OPT-003` | Context budget/local focus: entry → focused → expanded только с reason; low confidence ведёт в recovery. | State / context loading; startup, restore, context lift. | `State/service_state.md`; `Protocols/common/context_loading_protocol.md` | Остановить affected execution, загрузить минимальный missing evidence и зафиксировать recovery marker. |
| `OPT-004` | Links/backlinks/orphans: actual exact-case relative links должны совпадать с registry. | Navigation; create/update/delete Markdown или final acceptance. | `State/page_registry.jsonl`; `State/service_validation_report.md` | `blocked`; repair content/registry/backlinks одной bounded transaction, затем повторить readback. |

## Dependency readiness

Graph metadata фиксирует `node_count=23`, `edge_count=39`, `cycle_check=pass`. Raw historical edge status `satisfied` нормализуется в `ready` только вместе с closed source issue, required artifact/state evidence и accepted validation ref; одна строка `satisfied` без evidence не разрешает transition. Deferred `USER-001` не блокирует current production manifest.

## Связанные файлы

- [Root README](../README.md)
- [Service state](../State/service_state.md)
- [Service instructions](../Instructions/service_mode_project_instructions.md)
- Service validation report: `State/service_validation_report.md`
- Structural backlog: `State/structural_backlog.jsonl`
- [Protocol catalog](../Protocols/catalog.md)
- [Issue archive](_archive/README.md)
- [Issue tombstones](_tombstones/README.md)
