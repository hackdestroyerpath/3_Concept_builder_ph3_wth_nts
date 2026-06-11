# Каталог протоколов

Parent: [README](../README.md)  
Источник истины: `Protocols/catalog.md`  
Machine companion: [catalog.jsonl](catalog.jsonl)  
Status: `available`  
Updated: `2026-06-05T11:45:45Z`

## Назначение

Каталог выбирает ближайший рабочий protocol по mode, trigger, active issue, phase и scope. Planned protocol нельзя исполнять как available.

## Common protocols

| ID | Status | Path | Когда использовать |
|---|---|---|---|
| `common/context_loading` | `available` | [context_loading_protocol.md](common/context_loading_protocol.md) | старт, focus packet, context lift |
| `common/persistence_transaction` | `available` | [persistence_protocol.md](common/persistence_protocol.md) | перед любой записью |
| `common/final_validation` | `available` | [final_validation_protocol.md](common/final_validation_protocol.md) | перед closure/export/package status |

## Service protocols

| ID | Status | Path | Когда использовать |
|---|---|---|---|
| `service/service_start` | `available` | [service_start_protocol.md](service_protocols/service_start_protocol.md) | старт Service Mode |
| `service/new_issue` | `available` | [new_issue_protocol.md](service_protocols/new_issue_protocol.md) | новый service issue или input intake |
| `service/existing_issue` | `available` | [existing_issue_protocol.md](service_protocols/existing_issue_protocol.md) | фокус на существующий issue |
| `service/question_answer` | `available` | [question_answer_protocol.md](service_protocols/question_answer_protocol.md) | materially important unknowns |
| `service/requirements` | `available` | [requirements_protocol.md](service_protocols/requirements_protocol.md) | requirements phase |
| `service/solution_contract_output` | `available` | [solution_contract_output_protocol.md](service_protocols/solution_contract_output_protocol.md) | solution, contract, output |
| `service/complex_issue` | `available` | [complex_issue_protocol.md](service_protocols/complex_issue_protocol.md) | decomposition |
| `service/linked_issues` | `available` | [linked_issues_protocol.md](service_protocols/linked_issues_protocol.md) | dependencies and graph repair |
| `service/issue_retention` | `available` | [issue_retention_protocol.md](service_protocols/issue_retention_protocol.md) | archive, tombstone, cleanup |

## Execution protocols

| ID | Status | Path | Когда использовать |
|---|---|---|---|
| `execution/index` | `available` | [README.md](execution_protocols/README.md) | вход в concept scope |
| `execution/concept_export` | `available` | [concept_export_protocol.md](execution_protocols/concept_export_protocol.md) | concept export / closure package |

## Routing rules

1. Start with [../README.md](../README.md) and active state.
2. Use the most local protocol that matches mode and phase.
3. For concept work, map service lifecycle paths into `Concepts/<concept_slug>/` scope.
4. If no available protocol matches, create or defer a service issue.
5. Before writing, apply [common/persistence_protocol.md](common/persistence_protocol.md).
6. Before closing, apply [common/final_validation_protocol.md](common/final_validation_protocol.md).

## Related

- [../State/service_state.md](../State/service_state.md)
- [../State/execution_index.md](../State/execution_index.md)
- [../Issues/issue_registry.md](../Issues/issue_registry.md)
- [../Concepts/README.md](../Concepts/README.md)
