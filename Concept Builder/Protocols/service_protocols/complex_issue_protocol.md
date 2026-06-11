# Complex issue protocol

Parent: [Protocol catalog](../catalog.md)  
Protocol ID: `service/complex_issue`  
Status: `available`  
Updated: `2026-06-05T11:45:45Z`

## Purpose

Split work when one service issue is too large for one safe cycle. The protocol keeps parent outcome, child outcomes and dependency edges explicit.

## When to use

Use this protocol when an issue has several separable deliverables, unclear ownership, or different phases that should not be approved together.

## Steps

1. Read the active issue in [../../Issues/issue_registry.md](../../Issues/issue_registry.md).
2. Define the parent result.
3. List proposed child issues.
4. For each child, define scope, blocker status and expected output.
5. Add dependency edges in [../../Issues/dependency_graph.jsonl](../../Issues/dependency_graph.jsonl).
6. Do not close the parent while blocking children remain open.
7. Persist registry, graph and state changes.

## Output

A parent issue with traceable child issues and no dependency cycle.

## Related

- [linked_issues_protocol.md](linked_issues_protocol.md)
- [requirements_protocol.md](requirements_protocol.md)
- [solution_contract_output_protocol.md](solution_contract_output_protocol.md)
