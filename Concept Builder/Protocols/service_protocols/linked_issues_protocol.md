# Linked issues protocol

Parent: [Protocol catalog](../catalog.md)
Status: `available`

## Purpose

Track relationships and dependencies between issues.

## Steps

1. Identify source issue and target issue.
2. Define edge type: blocking, related or informational.
3. Add edge to [../../Issues/dependency_graph.jsonl](../../Issues/dependency_graph.jsonl).
4. Check for cycles before marking dependent issue ready.
5. Update [../../Issues/issue_registry.md](../../Issues/issue_registry.md).

## Output

A dependency edge with reason and readiness impact.
