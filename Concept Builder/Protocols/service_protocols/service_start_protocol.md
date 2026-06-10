# Service start protocol

Parent: [Protocol catalog](../catalog.md)
Status: `available`
Updated: `2026-06-05T11:45:45Z`

## Purpose

Starts Concept Builder Service Mode and selects the next protocol.

## Steps

1. Read [../../README.md](../../README.md).
2. Read [../../State/service_state.md](../../State/service_state.md).
3. Read [../catalog.md](../catalog.md).
4. Check [../../Issues/issue_registry.md](../../Issues/issue_registry.md).
5. Select the smallest protocol matching the user trigger.
6. Apply [../common/context_loading_protocol.md](../common/context_loading_protocol.md).
7. Before writing, apply [../common/persistence_protocol.md](../common/persistence_protocol.md).

## Output

A focus packet with mode, active issue, phase, selected protocol, blockers and next action.
