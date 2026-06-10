# Execution protocols

Parent: [Protocol catalog](../catalog.md)
Status: `available`

## Purpose

Entry point for Concept Builder execution mode.

## Rules

1. Work inside a concrete `Concepts/<concept_slug>/` scope.
2. Use [../../State/execution_index.md](../../State/execution_index.md) to find active concept.
3. Use service lifecycle protocols through concept path mapping.
4. Escalate root-system defects to Service Mode.
5. Export only through [concept_export_protocol.md](concept_export_protocol.md).

## Path mapping

Root `Issues/` maps to `Concepts/<concept_slug>/Issues/`. Root `State/` maps to `Concepts/<concept_slug>/State/`.

## Related

- [../../Concepts/README.md](../../Concepts/README.md)
- [../../Concepts/_template/README.md](../../Concepts/_template/README.md)
- [concept_export_protocol.md](concept_export_protocol.md)
