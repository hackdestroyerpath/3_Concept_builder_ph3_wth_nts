# Final validation protocol

Parent: [Protocol catalog](../catalog.md)  
Protocol ID: `common/final_validation`  
Status: `available`  
Updated: `2026-06-05T11:45:45Z`

## Purpose

Final validation confirms that a Concept Builder scope is internally consistent before closure, export, or commit-ready handoff. It checks structure, links, metadata, JSONL, issue coverage, dependency graph, production boundary and persistence status.

## Scope

This protocol applies to root `Concept Builder/` and to future concept scopes under `Concepts/<concept_slug>/`.

## Required checks

1. Required files exist.
2. Markdown links resolve or have documented external status.
3. Page registry paths match actual production files.
4. JSONL files parse line by line.
5. Dependency graph has known nodes and no blocking cycles.
6. Protocol catalog points only to existing available protocols.
7. Production files do not reference development-only source paths as working files.
8. Issue coverage is closed, rejected, deferred with reason, or explicitly blocked.
9. Service and execution scopes are not mixed without a cross-scope reason.
10. Persistence log states whether GitHub commit was actually performed.
11. Protocol metadata is consistent across Markdown, catalog JSONL and page registry.
12. Readable production Markdown uses neutral working style.
13. Root README and state files point to the same next-step model.

## Hard blockers

Validation is `blocked` if any required file is missing, JSONL does not parse, dependency graph has a blocking cycle, production tree contains development-only materials, or GitHub persistence is claimed without a verified commit.

## Result statuses

- `pass`
- `pass_with_deferred_items`
- `blocked`

`pass_with_deferred_items` is allowed only when the deferred item is non-blocking and has reason plus next action.

## Procedure

1. Freeze the checked scope.
2. Build a file list.
3. Check links and parent/backlink rules.
4. Parse JSONL files.
5. Check issue registry and dependency graph.
6. Check protocol catalog and metadata sync.
7. Check production/development boundary.
8. Save validation report.
9. Update persistence log.
10. Report only the status that is actually persisted.

## Output

Root validation result is written to [../../State/service_validation_report.md](../../State/service_validation_report.md). Concept validation result is written inside the concrete concept scope.

## Related

- [context_loading_protocol.md](context_loading_protocol.md)
- [persistence_protocol.md](persistence_protocol.md)
- [../catalog.md](../catalog.md)
- [../../Issues/issue_registry.md](../../Issues/issue_registry.md)
- [../../State/page_registry.jsonl](../../State/page_registry.jsonl)
- [../../State/persistence_log.jsonl](../../State/persistence_log.jsonl)
