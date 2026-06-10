# Final validation protocol

Parent: [Protocol catalog](../catalog.md)
Status: `available`
Updated: `2026-06-05T11:45:45Z`

## Purpose

Final validation confirms that a Concept Builder scope is internally consistent before closure, export, or commit-ready handoff.

## Checks

1. Required files exist.
2. Markdown links resolve.
3. Page registry paths match actual files.
4. JSONL files parse line by line.
5. Dependency graph has known nodes and no blocking cycles.
6. Protocol catalog points only to existing available protocols.
7. Production files do not reference development-only source paths as working files.
8. Issue coverage is closed, rejected, deferred with reason, or explicitly blocked.
9. Service and execution scopes are not mixed without a cross-scope reason.
10. Persistence log states whether GitHub commit was actually performed.

## Result statuses

- `pass`
- `pass_with_deferred_items`
- `blocked`

## Output

Root service validation is written to [../../State/service_validation_report.md](../../State/service_validation_report.md). Concept validation is written inside the concrete concept scope.

## Related

- [context_loading_protocol.md](context_loading_protocol.md)
- [persistence_protocol.md](persistence_protocol.md)
- [../catalog.md](../catalog.md)
- [../../Issues/issue_registry.md](../../Issues/issue_registry.md)
