# Existing issue protocol

Parent: [Protocol catalog](../catalog.md)  
Protocol ID: `service/existing_issue`  
Status: `available`  
Updated: `2026-06-05T11:45:45Z`

## Purpose

Focus an existing issue, verify blockers, select the next phase protocol and prevent work outside the issue scope.

## When to use

Use this protocol when the user says `продолжай`, names an issue ID, asks to resume work, or asks to audit/update a known issue.

## Required reads

1. [../../State/service_state.md](../../State/service_state.md)
2. [../../Issues/issue_registry.md](../../Issues/issue_registry.md)
3. [../../Issues/issue_registry.jsonl](../../Issues/issue_registry.jsonl)
4. [../../Issues/dependency_graph.jsonl](../../Issues/dependency_graph.jsonl)
5. [../catalog.md](../catalog.md)

## Focus packet

The focus packet must contain:

| Field | Meaning |
|---|---|
| `issue_id` | selected issue |
| `status` | current issue status |
| `phase` | active lifecycle phase |
| `blockers` | blocking dependencies or none |
| `next_protocol` | selected protocol from catalog |
| `write_scope` | affected files only |

## Steps

1. Locate issue in the registry.
2. Check status and phase.
3. Read blocking dependencies.
4. Select next protocol: QA, requirements, solution/output, complex issue, linked issues or retention.
5. If the issue is closed, do not reopen it without user intent.
6. If a blocker exists, return blocker and do not execute downstream work.
7. If writing is needed, use [../common/persistence_protocol.md](../common/persistence_protocol.md).

## Completion

The protocol is complete when the next protocol or blocker is explicit and the agent has not loaded or modified unrelated scope.
