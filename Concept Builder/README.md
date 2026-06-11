# Concept Builder

Status: `commit_ready_package`  
Updated: `2026-06-05T11:45:45Z`

## Назначение

`Concept Builder` — production-сеть файлов для двух режимов работы: `Service Mode` обслуживает структуру, state, protocols и issues; `Execution Mode` развивает конкретные концепции внутри [Concepts](Concepts/README.md).

## Старт агента

1. Открыть этот файл.
2. Для обслуживания системы открыть [State/service_state.md](State/service_state.md), [Protocols/catalog.md](Protocols/catalog.md), [Issues/issue_registry.md](Issues/issue_registry.md).
3. Для развития концепции открыть [State/execution_index.md](State/execution_index.md), [Concepts/README.md](Concepts/README.md), [Protocols/execution_protocols/README.md](Protocols/execution_protocols/README.md).
4. Перед записью применять [Protocols/common/persistence_protocol.md](Protocols/common/persistence_protocol.md).
5. Перед закрытием применять [Protocols/common/final_validation_protocol.md](Protocols/common/final_validation_protocol.md).

## Entry map

| Area | Path |
|---|---|
| Service state | [State/service_state.md](State/service_state.md) |
| Execution index | [State/execution_index.md](State/execution_index.md) |
| Page registry | [State/page_registry.jsonl](State/page_registry.jsonl) |
| Persistence log | [State/persistence_log.jsonl](State/persistence_log.jsonl) |
| Structural backlog | [State/structural_backlog.jsonl](State/structural_backlog.jsonl) |
| Service instructions | [Instructions/service_mode_project_instructions.md](Instructions/service_mode_project_instructions.md) |
| Execution instructions | [Instructions/execution_mode_project_instructions.md](Instructions/execution_mode_project_instructions.md) |
| Protocol catalog | [Protocols/catalog.md](Protocols/catalog.md) |
| Protocol catalog JSONL | [Protocols/catalog.jsonl](Protocols/catalog.jsonl) |
| Issues | [Issues/issue_registry.md](Issues/issue_registry.md) |
| Dependency graph | [Issues/dependency_graph.jsonl](Issues/dependency_graph.jsonl) |
| Inbox | [Inbox/README.md](Inbox/README.md) |
| Concepts | [Concepts/README.md](Concepts/README.md) |
| Concept template | [Concepts/_template/README.md](Concepts/_template/README.md) |

## Production areas

| Area | Purpose |
|---|---|
| `State/` | current state, registries, validation and persistence markers |
| `Instructions/` | project instruction source texts |
| `Protocols/` | operational protocols for service and execution modes |
| `Issues/` | service issue registry, graph, archive and tombstones |
| `Inbox/` | incoming material rules |
| `Concepts/` | concept layer entry and template |

## Rules

- Every Markdown file should have a parent/backlink or registry entry.
- Production tree must not contain source snapshots, checkpoint reports or development-only materials.
- GitHub write is done only when the connector returns a commit SHA or the ref is verified.
- Runtime concept folders and runtime issue folders are created only after explicit user intent or approved issue.
- New changes should go through issue lifecycle and persistence protocol.

## Next step

Open [State/service_state.md](State/service_state.md) for service work or [State/execution_index.md](State/execution_index.md) for concept work.
