# Solution contract output protocol

Parent: [Protocol catalog](../catalog.md)  
Protocol ID: `service/solution_contract_output`  
Status: `available`  
Updated: `2026-06-05T11:45:45Z`

## Purpose

Move an issue from approved requirements to solution, contract, output and validation. This protocol keeps implementation work tied to approved requirements and prevents output from drifting into unreviewed scope.

## When to use

Use this protocol after requirements are approved, or after requirements are explicitly skipped with a recorded reason. Do not use it while blocking dependencies, unanswered material questions, or approval gaps remain.

## Required inputs

1. Active issue from [../../Issues/issue_registry.md](../../Issues/issue_registry.md).
2. Approved requirements or a documented skip reason.
3. QA trace if the issue required questions.
4. Dependency status from [../../Issues/dependency_graph.jsonl](../../Issues/dependency_graph.jsonl).
5. Affected files and expected output paths.

## Solution phase

The solution states how the requirements will be satisfied. The plan belongs inside the solution, not in a duplicate source file. A solution must name the target files, scope boundaries, validation approach, and risks.

## Contract phase

The contract turns the solution into acceptance checks. Each requirement should map to a check, a file, or an explicit not-applicable reason. If the contract cannot be checked, the issue is not ready for execution.

## Execution phase

1. Change only files in the approved write set.
2. Keep service and concept scopes separate.
3. Do not create decorative files without operational role.
4. Update registry, state and page registry when files change.
5. Save output summary and validation evidence.
6. Use [../common/persistence_protocol.md](../common/persistence_protocol.md) before claiming completion.

## Output package

The output package should include changed files, output summary, validation result, unresolved blockers if any, and next action.

## Completion

The issue can close only after output, validation, registry update, state update and persistence marker exist.

## Related

- [requirements_protocol.md](requirements_protocol.md)
- [question_answer_protocol.md](question_answer_protocol.md)
- [linked_issues_protocol.md](linked_issues_protocol.md)
- [../common/final_validation_protocol.md](../common/final_validation_protocol.md)
