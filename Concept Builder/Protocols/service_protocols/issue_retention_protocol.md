# Issue retention protocol

Parent: [Protocol catalog](../catalog.md)  
Protocol ID: `service/issue_retention`  
Status: `available`  
Updated: `2026-06-05T11:45:45Z`

## Purpose

Define archive, tombstone, cleanup and deletion rules for service issues, Inbox packets and historical records.

## When to use

Use this protocol when an issue is closed, rejected, archived, compacted to tombstone, or when Inbox material can be cleaned up.

## Rules

1. Do not delete historical issue records without a retention decision.
2. Closed or rejected issues may move to archive.
3. Archive records may later become tombstones when full context is no longer needed.
4. Temporary or nonhistorical input may be removed only with reason.
5. Do not reuse issue IDs after tombstone or deletion.
6. Update registry, dependency graph, page registry and persistence log after retention work.

## Archive

Archive keeps enough information to understand the issue, decision, output references and validation result. Use [../../Issues/_archive/README.md](../../Issues/_archive/README.md) as the entry point.

## Tombstone

Tombstone keeps compact history after cleanup. Use [../../Issues/_tombstones/README.md](../../Issues/_tombstones/README.md) as the entry point.

## Inbox cleanup

Inbox cleanup must preserve input ID, linked issue, cleanup reason and retention status. Follow [../../Inbox/README.md](../../Inbox/README.md).

## Completion

Retention work is complete only after the registry, graph, page registry and persistence log are consistent.
