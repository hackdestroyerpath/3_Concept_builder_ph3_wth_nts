# Persistence protocol

Parent: [Protocol catalog](../catalog.md)
Status: `available`
Updated: `2026-06-05T11:45:45Z`

## Purpose

This protocol defines safe write behavior for Concept Builder production files.

## Rules

1. Read the current target file before update.
2. Define the write set before changing files.
3. Write primary artifact first.
4. Update related registry, state and persistence log.
5. Verify the written path by reading it back.
6. Do not claim GitHub commit until the connector returns a commit SHA or the ref is verified.

## Failure status

Use `blocked_on_persistence` when write is not confirmed. Use `package_draft_not_committed` when the state exists only as a local archive.

## Related

- [context_loading_protocol.md](context_loading_protocol.md)
- [final_validation_protocol.md](final_validation_protocol.md)
- [service_state.md](../../State/service_state.md)
- [persistence_log.jsonl](../../State/persistence_log.jsonl)
