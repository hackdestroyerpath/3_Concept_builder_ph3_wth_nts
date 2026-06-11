# Реестр issue

Parent: [README](../README.md)  
Owner issue: `EXEC-005`  
Источник истины: `Issues/issue_registry.md`  
Machine companion: [issue_registry.jsonl](issue_registry.jsonl)  
Dependency graph: [dependency_graph.jsonl](dependency_graph.jsonl)  
Status: `validated`  
Updated: `2026-06-05T11:45:45Z`

## Назначение

Этот файл — человекочитаемый вход в root issue-модель `Concept Builder`. Машиночитаемый источник — [issue_registry.jsonl](issue_registry.jsonl), dependency edges — [dependency_graph.jsonl](dependency_graph.jsonl). Runtime issue-папки не создаются заранее; они появляются только когда реальный protocol требует `reason.md`, `requirements.md`, `solution.md`, `contract.md`, `output/` или QA artifacts.

## Status policy

| Status | Значение |
|---|---|
| `closed` | issue реализован и покрыт финальной проверкой |
| `deferred` | issue явно отложен с reason и next action |
| `closed_as_continuous_guard` | разовое issue закрыто как постоянный guard в protocol/state/report |
| `blocked` | нельзя закрывать без repair step |

## Coverage snapshot

| ID | Class | Status | Dependency ready | Coverage / decision | Основной output |
|---|---|---|---|---|---|
| `EXEC-001` | `implementation` | `closed` | `not_applicable` | Закрыто финальной проверкой; дальнейшие изменения выполнять через новый approved issue. | [README.md](../README.md) |
| `EXEC-002` | `implementation` | `closed` | `ready` | Закрыто финальной проверкой; дальнейшие изменения выполнять через новый approved issue. | [Instructions/service_mode_project_instructions.md](../Instructions/service_mode_project_instructions.md) |
| `EXEC-003` | `implementation` | `closed` | `ready` | Закрыто финальной проверкой; дальнейшие изменения выполнять через новый approved issue. | [Protocols/catalog.md](../Protocols/catalog.md) |
| `EXEC-004` | `implementation` | `closed` | `ready` | Закрыто финальной проверкой; дальнейшие изменения выполнять через новый approved issue. | [State/service_state.md](../State/service_state.md) |
| `EXEC-005` | `implementation` | `closed` | `ready` | Закрыто финальной проверкой; дальнейшие изменения выполнять через новый approved issue. | [Issues/issue_registry.md](issue_registry.md) |
| `EXEC-006` | `implementation` | `closed` | `ready` | Закрыто финальной проверкой; дальнейшие изменения выполнять через новый approved issue. | [Protocols/service_protocols/service_start_protocol.md](../Protocols/service_protocols/service_start_protocol.md) |
| `EXEC-007` | `implementation` | `closed` | `ready` | Закрыто финальной проверкой; дальнейшие изменения выполнять через новый approved issue. | [Protocols/service_protocols/existing_issue_protocol.md](../Protocols/service_protocols/existing_issue_protocol.md) |
| `EXEC-008` | `implementation` | `closed` | `ready` | Закрыто финальной проверкой; дальнейшие изменения выполнять через новый approved issue. | [Protocols/service_protocols/question_answer_protocol.md](../Protocols/service_protocols/question_answer_protocol.md) |
| `EXEC-009` | `implementation` | `closed` | `ready` | Закрыто финальной проверкой; дальнейшие изменения выполнять через новый approved issue. | [Protocols/service_protocols/solution_contract_output_protocol.md](../Protocols/service_protocols/solution_contract_output_protocol.md) |
| `EXEC-010` | `implementation` | `closed` | `ready` | Закрыто финальной проверкой; дальнейшие изменения выполнять через новый approved issue. | [Protocols/service_protocols/complex_issue_protocol.md](../Protocols/service_protocols/complex_issue_protocol.md) |
| `EXEC-011` | `implementation` | `closed` | `ready` | Закрыто финальной проверкой; дальнейшие изменения выполнять через новый approved issue. | [Protocols/execution_protocols/README.md](../Protocols/execution_protocols/README.md) |
| `EXEC-012` | `implementation` | `closed` | `ready` | Закрыто финальной проверкой; дальнейшие изменения выполнять через новый approved issue. | [Protocols/common/final_validation_protocol.md](../Protocols/common/final_validation_protocol.md) |
| `USER-001` | `user-noted` | `deferred` | `ready` | Non-blocking backlog: оценить служебные скрипты отдельным approved issue после запуска Service Mode. | `Issues/CB-SVC-001-script-assessment/` |
| `USER-002` | `user-noted` | `closed` | `ready` | Закрыто финальной проверкой; дальнейшие изменения выполнять через новый approved issue. | [State/service_state.md](../State/service_state.md) |
| `USER-003` | `user-noted` | `closed` | `ready` | Закрыто финальной проверкой; дальнейшие изменения выполнять через новый approved issue. | [Protocols/execution_protocols/README.md](../Protocols/execution_protocols/README.md) |
| `USER-004` | `user-noted` | `closed` | `ready` | Закрыто финальной проверкой; дальнейшие изменения выполнять через новый approved issue. | [Protocols/service_protocols/new_issue_protocol.md](../Protocols/service_protocols/new_issue_protocol.md) |
| `USER-005` | `user-noted` | `closed` | `ready` | Закрыто финальной проверкой; дальнейшие изменения выполнять через новый approved issue. | [Protocols/service_protocols/requirements_protocol.md](../Protocols/service_protocols/requirements_protocol.md) |
| `USER-006` | `user-noted` | `closed` | `ready` | Закрыто финальной проверкой; дальнейшие изменения выполнять через новый approved issue. | [Protocols/service_protocols/issue_retention_protocol.md](../Protocols/service_protocols/issue_retention_protocol.md) |
| `USER-007` | `user-noted` | `closed` | `ready` | Закрыто финальной проверкой; дальнейшие изменения выполнять через новый approved issue. | [Protocols/execution_protocols/concept_export_protocol.md](../Protocols/execution_protocols/concept_export_protocol.md) |
| `OPT-001` | `optimizer-detected` | `closed_as_continuous_guard` | `not_applicable` | Закрыто проверкой границы production-слоя; дальнейшие изменения выполнять через новый approved issue. | [State/service_validation_report.md](../State/service_validation_report.md) |
| `OPT-002` | `optimizer-detected` | `closed_as_continuous_guard` | `not_applicable` | Закрыто финальной проверкой; дальнейшие изменения выполнять через новый approved issue. | [Protocols/common/final_validation_protocol.md](../Protocols/common/final_validation_protocol.md) |
| `OPT-003` | `optimizer-detected` | `closed_as_continuous_guard` | `ready` | Закрыто финальной проверкой; дальнейшие изменения выполнять через новый approved issue. | [State/service_state.md](../State/service_state.md) |
| `OPT-004` | `optimizer-detected` | `closed_as_continuous_guard` | `ready` | Закрыто финальной проверкой; дальнейшие изменения выполнять через новый approved issue. | [State/page_registry.jsonl](../State/page_registry.jsonl) |

## Deferred items

| ID | Reason | Next action |
|---|---|---|
| `USER-001` | Отложено с reason: скрипты не входят в production manifest и не создаются без отдельного owner/benefit/cost decision. | Non-blocking backlog: оценить служебные скрипты отдельным approved issue после запуска Service Mode. |

## Continuous guards

| ID | Guard | Где закреплено |
|---|---|---|

## Dependency readiness

Все blocking edges в [dependency_graph.jsonl](dependency_graph.jsonl) имеют status `satisfied`; cycle check: `pass`. Deferred `USER-001` не блокирует финальный package, потому что script layer не входит в обязательный production manifest.

## Связанные файлы

- [Root README](../README.md)
- [Service state](../State/service_state.md)
- [Service validation report](../State/service_validation_report.md)
- [Structural backlog](../State/structural_backlog.jsonl)
- [Protocol catalog](../Protocols/catalog.md)
- [Issue archive](_archive/README.md)
- [Issue tombstones](_tombstones/README.md)
