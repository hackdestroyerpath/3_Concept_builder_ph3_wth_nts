# Requirements protocol

Parent: [Protocol catalog](../catalog.md)  
Protocol ID: `service/requirements`  
Status: `available`  
Updated: `2026-06-05T11:45:45Z`

## Purpose

Define and approve requirements before solution, contract or output work. Requirements are the stable source of truth for what must be built, changed, validated or intentionally excluded.

## When to use

Use this protocol after issue reason is clear and QA is complete or explicitly skipped. Do not start solution work if requirements are missing, ambiguous or materially changed by new input.

## Required inputs

1. Active issue from [../../Issues/issue_registry.md](../../Issues/issue_registry.md).
2. QA trace or skip reason from [question_answer_protocol.md](question_answer_protocol.md).
3. Relevant source files and affected paths.
4. Dependency status from [../../Issues/dependency_graph.jsonl](../../Issues/dependency_graph.jsonl).

## Requirement fields

| Field | Meaning |
|---|---|
| `id` | stable requirement ID |
| `statement` | what must be true |
| `source_ref` | source input, issue or file |
| `acceptance_check` | how to verify it |
| `scope` | included files or behavior |
| `status` | draft, approved, changed, rejected |

## Steps

1. Read reason, QA and affected files.
2. Separate in-scope from out-of-scope requirements.
3. Write requirements as checkable statements.
4. Mark assumptions explicitly.
5. Ask for approval when requirements define user-facing behavior or repository structure.
6. If new input changes requirements, reopen this phase.
7. Persist requirements and update issue state.

## Output

The result is an approved requirements set or a blocked/deferred state with reason and next action.

## Related

- [question_answer_protocol.md](question_answer_protocol.md)
- [solution_contract_output_protocol.md](solution_contract_output_protocol.md)
- [existing_issue_protocol.md](existing_issue_protocol.md)
- [../common/persistence_protocol.md](../common/persistence_protocol.md)
