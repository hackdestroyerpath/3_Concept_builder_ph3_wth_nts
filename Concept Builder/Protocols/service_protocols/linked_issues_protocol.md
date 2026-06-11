# Linked issues protocol

Parent: [Protocol catalog](../catalog.md)  
Protocol ID: `service/linked_issues`  
Status: `available`  
Updated: `2026-06-05T11:45:45Z`

## Purpose

Track relationships and dependencies between issues. This protocol keeps dependency edges explicit, prevents cycles and makes readiness decisions auditable.

## Edge types

| Type | Meaning |
|---|---|
| `blocking` | target must be complete before source can close |
| `related` | issues share context but do not block each other |
| `informational` | issue references another issue as background |

## Steps

1. Identify source issue and target issue.
2. Confirm both issues exist in [../../Issues/issue_registry.jsonl](../../Issues/issue_registry.jsonl).
3. Define edge type and reason.
4. Add or update edge in [../../Issues/dependency_graph.jsonl](../../Issues/dependency_graph.jsonl).
5. Check for dependency cycles.
6. Update issue readiness if a new blocker appears or is resolved.
7. Persist registry and graph changes.

## Readiness rule

An issue with open blocking dependencies is not ready for closure. A dependency may be satisfied, deferred, rejected or marked not applicable only with an explicit reason.

## Repair rule

If the graph references a missing issue or stale edge, stop downstream work and create a graph repair action.

## Related

- [complex_issue_protocol.md](complex_issue_protocol.md)
- [existing_issue_protocol.md](existing_issue_protocol.md)
- [../../Issues/dependency_graph.jsonl](../../Issues/dependency_graph.jsonl)
- [../../Issues/issue_registry.md](../../Issues/issue_registry.md)
