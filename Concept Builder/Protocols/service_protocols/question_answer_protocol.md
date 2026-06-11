# Question answer protocol

Parent: [Protocol catalog](../catalog.md)  
Protocol ID: `service/question_answer`  
Status: `available`  
Updated: `2026-06-05T11:45:45Z`

## Purpose

Capture only materially important unknowns before requirements, solution or output work. The protocol prevents speculative work while avoiding unnecessary questionnaires.

## When to ask

Ask a question only when the answer can change at least one of these items:

| Area | Example impact |
|---|---|
| Scope | include or exclude a file, feature or artifact |
| Requirement | change required behavior or acceptance criteria |
| Contract | change what counts as done |
| Dependency | add, remove or reprioritize a blocker |
| Output | change final format, location or validation method |

If the answer will not change the work, do not ask. Record a skip reason and continue.

## Steps

1. Read the active issue and current phase.
2. List unknowns that affect requirements, solution, contract, dependencies or output.
3. Remove questions that are merely stylistic or already answered by source files.
4. Ask the smallest useful set of questions.
5. Save answers or skip reason in the issue record.
6. If an answer changes scope, reopen the smallest affected phase.
7. Persist changes through [../common/persistence_protocol.md](../common/persistence_protocol.md).

## Output

The result is one of:

- `qa_completed`: answers are recorded and next phase can proceed;
- `qa_skipped`: no material unknowns remain;
- `qa_blocked`: user answer is required before safe progress;
- `phase_reopened`: new answer changed scope and previous approval is no longer enough.

## Related

- [requirements_protocol.md](requirements_protocol.md)
- [solution_contract_output_protocol.md](solution_contract_output_protocol.md)
- [existing_issue_protocol.md](existing_issue_protocol.md)
- [../../Issues/issue_registry.md](../../Issues/issue_registry.md)
