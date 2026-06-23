# Сервисное состояние

Parent: [README](../README.md)  
Owner issue: `EXEC-004`  
Источник истины: `State/service_state.md`  
Status: `repair_candidate_pending_manual_reviewer`  
Updated: `2026-06-22T21:56:58+02:00`

## Назначение

Этот файл хранит authoritative верхнее состояние `Concept Builder Service Mode`. Он различает factual состояние base `main`, текущий bounded branch candidate и eventual post-merge closure transition.

## Текущий service scope

| Поле | Значение |
|---|---|
| Mode | `Service Mode` |
| Active scope | `root_service` |
| Active issue | `none` |
| Active phase | `current_main_truth_language_repair` |
| Repository | `hackdestroyerpath/3_Concept_builder_ph3_wth_nts` |
| Production root | `/` |
| Default branch | `main` |
| Task branch | `agent/cbu-current-main-truth-language-repair-20260622` |
| Write status | `current_main_truth_language_repair_candidate_pending_manual_reviewer` |
| Validation status | `pass_with_deferred_items` for accepted implementation; repair candidate pending final branch readback/manual review |
| Validation report | [service_validation_report.md](service_validation_report.md) |
| Blocking status | `pending_manual_reviewer` |
| Latest accepted implementation | `Stage 06 merged in PR #21` |
| Verified base main | `194970c7c5ac37f2dbbfcd51256caaa46f67f8f9` |
| Stage 07 status | `repair_candidate_pending_manual_reviewer` |
| Base residual IDs | `RES-CLOSURE-001`, `RES-LANG-001` |
| Candidate residual IDs | `[]` after complete branch validation/readback |
| Closure candidate | `true` only for complete branch readback |
| Closure allowed | `false` |
| Next state | `pending_manual_review_squash_merge_fresh_main_readback_and_final_closure_transition` |

## Canonical candidate fields

```yaml
latest_accepted_implementation: Stage 06 merged in PR #21
verified_base_main: 194970c7c5ac37f2dbbfcd51256caaa46f67f8f9
stage_07_status: repair_candidate_pending_manual_reviewer
base_residual_ids: [RES-CLOSURE-001, RES-LANG-001]
candidate_residual_ids: []
closure_candidate: true
closure_allowed: false
next_state: pending_manual_review_squash_merge_fresh_main_readback_and_final_closure_transition
```

`candidate_residual_ids=[]` и `closure_candidate=true` становятся factual только после complete branch validation/readback.

## Base-main truth и candidate boundary

PR `#21` принят и его Stage 06 implementation присутствует в `main`. Commit `194970c7c5ac37f2dbbfcd51256caaa46f67f8f9` добавил unsupported positive final-closure claim только в validation report, не синхронизировав README, service state, issue snapshot, page metadata и persistence evidence.

Текущая ветка исправляет эти два residuals:

| ID | Base-main defect | Candidate result |
|---|---|---|
| `RES-CLOSURE-001` | Cross-file closure truth расходится; factual final-transition persistence отсутствует. | Семь truth files согласованы на `closure_candidate=true`, `closure_allowed=false`; exact branch readback обязателен. |
| `RES-LANG-001` | `Inbox/README.md` содержит conversational/evaluative sentence. | Формулировка заменена нейтральным operational rule; full production Markdown scan обязателен. |

Branch candidate не является merged state. Manual review, squash merge, fresh `main` readback и separate final closure transition остаются внешними gates.

## Accepted implementation capabilities

### Service Mode

- new/existing issue routing и duplicate prevention;
- QA, requirements approval/reopen;
- solution / contract / output separation;
- complex/linked readiness и cycle/stale handling;
- retention, archive, tombstone и Inbox cleanup;
- transaction-like persistence и bounded recovery;
- `GOV-001`.

### Execution Mode

- `no_active_concept`, `active_known`, `active_unknown`;
- five-file concept bootstrap;
- verified-only readiness и lifecycle/readiness/integrity separation;
- concept-local page/issue/dependency isolation;
- canonical service escalation;
- read-only export precheck, closed/WIP blocker matrix, deterministic naming, collision/idempotency и package-root local-open.

## Минимальная загрузка при старте

1. [../README.md](../README.md)
2. [service_state.md](service_state.md)
3. [page_registry.jsonl](page_registry.jsonl)
4. [../Protocols/catalog.md](../Protocols/catalog.md)
5. [../Protocols/service_protocols/service_start_protocol.md](../Protocols/service_protocols/service_start_protocol.md)
6. [../Issues/issue_registry.md](../Issues/issue_registry.md) и `Issues/issue_registry.jsonl`
7. [../Issues/dependency_graph.jsonl](../Issues/dependency_graph.jsonl)
8. [../Protocols/common/persistence_protocol.md](../Protocols/common/persistence_protocol.md)
9. [../Protocols/common/final_validation_protocol.md](../Protocols/common/final_validation_protocol.md)

## Context budget

| Уровень | Разрешённый пакет | Статус |
|---|---|---|
| `entry` | README, active state, page registry, catalog | default |
| `focused` | selected issue/concept, nearest protocol, affected files | after selection |
| `expanded` | direct parent/dependency summaries | only with reason |
| `full_scope` | весь service или concept scope | final validation / approved refactor |
| `repository_wide` | широкий scan | forbidden by default; allowed only for explicit validation need |

## Deferred и blockers

- `USER-001` остаётся deferred/nonblocking: service scripts требуют usage signal и отдельный approved cost/benefit issue.
- Concrete concept folders не созданы без user concept scope.
- Current blocking gate: manual reviewer acceptance текущего exact seven-file branch candidate.

<a id="next-step-marker"></a>

## Next-step marker

Next status: `pending_manual_review_squash_merge_fresh_main_readback_and_final_closure_transition`.

Следующий шаг не меняет `closure_allowed`: reviewer проверяет exact branch head, seven-path scope, blobs и validation evidence. Merge и post-merge closure находятся вне этой задачи.
