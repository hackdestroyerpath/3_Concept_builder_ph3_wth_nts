# Solution contract output protocol

Parent: [Protocol catalog](../catalog.md)  
Protocol ID: `service/solution_contract_output`  
Status: `available`

## Purpose

Move an issue from approved requirements to solution, contract, output and validation.

## Rules

1. Do not execute work without approved requirements or a recorded skip reason.
2. Keep the plan inside `solution.md`; do not create a duplicate plan source.
3. Create a contract with acceptance checks before output work.
4. Change only files in the approved write set.
5. Save output summary and validation evidence.
6. Update registry, state and page registry after file changes.
7. Use [../common/persistence_protocol.md](../common/persistence_protocol.md) before claiming completion.

## Completion

The issue can close only after output, validation, registry update and persistence marker exist.

## Related

- [requirements_protocol.md](requirements_protocol.md)
- [question_answer_protocol.md](question_answer_protocol.md)
- [linked_issues_protocol.md](linked_issues_protocol.md)
- [../common/final_validation_protocol.md](../common/final_validation_protocol.md)
