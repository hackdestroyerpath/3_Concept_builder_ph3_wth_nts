# New issue protocol

Parent: [Protocol catalog](../catalog.md)  
Protocol ID: `service/new_issue`  
Status: `available`  
Updated: `2026-06-05T11:45:45Z`

## Purpose

Create a new service-level issue from user input, uploaded material, or a confirmed service task. The protocol preserves traceability from input to reason, requirements, output and validation.

## When to use

Use this protocol when the user provides new work that is not already represented by an active issue. Do not create a duplicate issue if an existing issue already owns the same scope.

## Required checks

1. Read [../../README.md](../../README.md).
2. Read [../../State/service_state.md](../../State/service_state.md).
3. Read [../../Issues/issue_registry.md](../../Issues/issue_registry.md) and [../../Issues/issue_registry.jsonl](../../Issues/issue_registry.jsonl).
4. Check [../../Issues/dependency_graph.jsonl](../../Issues/dependency_graph.jsonl).
5. If the user provided raw material or attachments, follow [../../Inbox/README.md](../../Inbox/README.md).
6. Select the smallest lifecycle phase that fits the request.

## Issue creation fields

Every new issue needs:

| Field | Meaning |
|---|---|
| `issue_id` | stable service issue ID |
| `title` | short readable title |
| `class` | `service`, `implementation`, `repair`, `audit`, `user-noted` |
| `status` | initial status |
| `phase` | current lifecycle phase |
| `reason` | why the issue exists |
| `scope` | files or areas affected |
| `owner` | service or concept scope |
| `next_action` | next protocol or blocker |

## Process

1. Capture input reference or create an Inbox packet.
2. Assign issue ID.
3. Write reason and scope.
4. Decide whether QA is required.
5. Add issue row to JSONL registry.
6. Add human-readable summary to issue registry.
7. Add dependency edges only when they are real.
8. Update service state next-step marker.
9. Persist changes with [../common/persistence_protocol.md](../common/persistence_protocol.md).

## Add-during-approval branch

If new material arrives during approval or review, do not silently append it to an approved scope. Record the new input and reopen the smallest affected phase: QA, requirements, solution or contract.

## Completion

The protocol is complete when a new issue is registered, dependencies are known, next action is explicit and persistence status is honest.
