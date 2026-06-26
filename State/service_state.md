# Сервисное состояние

Parent: [README](../README.md)  
Owner issue: `EXEC-004`  
Источник истины: `State/service_state.md`  
Status: `repair_rework_candidate_pending_manual_reviewer`  
Updated: `2026-06-25T22:04:48+02:00`

## Назначение

Этот файл хранит верхнее состояние `Concept Builder Service Mode`. Его читают при старте обслуживания системы, перед изменением структуры репозитория и перед переходом к service-level `issue`.

## Текущий service scope

| Поле | Значение |
|---|---|
| Mode | `Service Mode` |
| Active scope | `root_service` |
| Active issue | `none` |
| Active phase | `current_main_truth_language_repair_rework` |
| Repository | `hackdestroyerpath/3_Concept_builder_ph3_wth_nts` |
| Production root | `/` |
| Default branch | `main` |
| Task branch | `agent/cbu-current-main-truth-language-repair-20260622` |
| Pull request | `#22` (`open`, `unmerged`) |
| Write status | `current_main_truth_language_repair_rework_candidate_pending_manual_reviewer` |
| GitHub metadata | `write_mode=branch_pr; base_main=194970c7c5ac37f2dbbfcd51256caaa46f67f8f9; reviewed_input_head=beb12ea6d012949b4480c84fc493203bb1f13a11` |
| Validation status | `pass_with_deferred_items`; rework candidate pending complete branch readback and manual reviewer recheck |
| Validation report | [service_validation_report.md](service_validation_report.md) |
| Blocking status | `pending_manual_reviewer` |
| Latest accepted implementation | `Stage 06 merged in PR #21` |
| Verified base main | `194970c7c5ac37f2dbbfcd51256caaa46f67f8f9` |
| Stage 05 | `05A accepted; 05B accepted; top-level accepted_merged_main` |
| Stage 06 | `accepted implementation merged in PR #21` |
| Stage 07 status | `repair_candidate_pending_manual_reviewer` |
| Base residual IDs | `RES-CLOSURE-001`, `RES-LANG-001` |
| Candidate residual IDs | `[]` only after complete branch validation/readback |
| Closure candidate | `true` only after complete branch validation/readback |
| Closure allowed | `false` |
| Next status | `pending_manual_reviewer_recheck_then_squash_merge_fresh_main_readback_and_separate_final_closure_transition` |

Stage 06 implementation from merged PR `#21` присутствует в `main`. Commit `194970c7c5ac37f2dbbfcd51256caaa46f67f8f9` добавил unsupported positive final-closure claim только в [service_validation_report.md](service_validation_report.md), тогда как README, service state, issue snapshot, page metadata и [persistence_log.jsonl](persistence_log.jsonl) не подтверждали отдельный разрешённый final transition.

Текущая ветка `agent/cbu-current-main-truth-language-repair-20260622` исправляет base residuals `RES-CLOSURE-001` и `RES-LANG-001`. Candidate residuals считаются пустыми, а `closure_candidate=true` — factual только после полного final-branch readback. Ветка не является merged state; `closure_allowed=false`, manual reviewer recheck, squash merge, fresh `main` readback и separate final-closure transition остаются обязательными gates. Приведённые ниже operational sections и accepted capabilities сохраняются без сокращения.

## Accepted Stage 04 capabilities

- issue continuation и duplicate prevention;
- requirements approval и reopen;
- разделение solution / contract / output;
- complex/linked readiness;
- retention lifecycle;
- `GOV-001`;
- optional attachment artifacts.

## Accepted Stage 05A capabilities

- explicit startup cases;
- five-file concept bootstrap;
- verified-only readiness и разделение lifecycle/readiness/integrity;
- local page/issue/dependency isolation;
- canonical dependency metadata seed;
- canonical service-escalation handoff;
- no runtime concept without user scope.

## Accepted Stage 05B capabilities

- read-only export precheck;
- explicit closed/WIP blocker и limitation matrix;
- deterministic naming и collision/idempotency policy;
- package-root local-open и root-escape gates;
- compact issue summary;
- synthetic non-production smoke strategy;
- common final-validation и human/machine catalog parity.

## Минимальная загрузка при старте `Service Mode`

1. [../README.md](../README.md) — корневой вход и карта маршрутов.
2. [service_state.md](service_state.md) — текущий service-scope, validation status и next-step marker.
3. [page_registry.jsonl](page_registry.jsonl) — проверка существования страниц, parent, backlinks и orphan-status.
4. [../Protocols/catalog.md](../Protocols/catalog.md) — выбор самого локального протокола.
5. [../Protocols/service_protocols/service_start_protocol.md](../Protocols/service_protocols/service_start_protocol.md) — старт `Service Mode` и первичная навигация.
6. [../Protocols/service_protocols/existing_issue_protocol.md](../Protocols/service_protocols/existing_issue_protocol.md) — фокусировка на существующем issue.
7. [../Protocols/service_protocols/question_answer_protocol.md](../Protocols/service_protocols/question_answer_protocol.md) — QA перед requirements, если есть materially important unknowns.
8. [../Protocols/service_protocols/requirements_protocol.md](../Protocols/service_protocols/requirements_protocol.md) — draft/review/approval/reopen requirements.
9. [../Protocols/service_protocols/solution_contract_output_protocol.md](../Protocols/service_protocols/solution_contract_output_protocol.md) — solution/contract review, execution и output package.
10. [../Protocols/service_protocols/complex_issue_protocol.md](../Protocols/service_protocols/complex_issue_protocol.md) — decomposition и requalification.
11. [../Protocols/service_protocols/linked_issues_protocol.md](../Protocols/service_protocols/linked_issues_protocol.md) — dependency edges, stale и cycle handling.
12. [../Protocols/service_protocols/issue_retention_protocol.md](../Protocols/service_protocols/issue_retention_protocol.md) — archive, tombstone, deletion и Inbox cleanup lifecycle.
13. [../Protocols/execution_protocols/README.md](../Protocols/execution_protocols/README.md) — execution scope и concept template routing.
14. [../Protocols/execution_protocols/concept_export_protocol.md](../Protocols/execution_protocols/concept_export_protocol.md) — export policy для closed/WIP concept package.
15. [../Protocols/common/context_loading_protocol.md](../Protocols/common/context_loading_protocol.md) — правила минимальной загрузки и context lift.
16. [../Protocols/common/final_validation_protocol.md](../Protocols/common/final_validation_protocol.md) — проверка перед закрытием issue, commit package или export.
17. [../Issues/issue_registry.md](../Issues/issue_registry.md) и [../Issues/issue_registry.jsonl](../Issues/issue_registry.jsonl) — текущая issue-модель и bootstrap registry.
18. [../Issues/dependency_graph.jsonl](../Issues/dependency_graph.jsonl) — проверка blocking dependencies перед выбором issue.
19. [../Inbox/README.md](../Inbox/README.md) — читать только при intake нового issue или проверке Inbox lifecycle.

Агент не должен имитировать planned протоколы: если нужный протокол ещё не создан, он создаёт implementation step или фиксирует blocker.

## State-области верхнего уровня

| Файл | Роль | Когда читать |
|---|---|---|
| [service_state.md](service_state.md) | состояние обслуживания системы | старт `Service Mode`, структурные изменения, validation |
| [execution_index.md](execution_index.md) | индекс концепций и active execution-object | старт `Execution Mode`, выбор или создание концепции |
| [page_registry.jsonl](page_registry.jsonl) | реестр страниц и backlinks | создание, удаление, проверка или навигация |
| [persistence_log.jsonl](persistence_log.jsonl) | журнал transaction-like сохранения | перед ответом о сохранении, после записи |
| [structural_backlog.jsonl](structural_backlog.jsonl) | управляемый backlog структурных решений | при новом структурном вопросе или guard-проверке |
| [service_validation_report.md](service_validation_report.md) | финальная проверка service layer | перед commit/export или handoff |

## Доступные service protocols

| Протокол | Статус | Когда использовать |
|---|---|---|
| [../Protocols/service_protocols/service_start_protocol.md](../Protocols/service_protocols/service_start_protocol.md) | `available` | `пинг`, `старт`, service bootstrap, `продолжай` в service scope |
| [../Protocols/service_protocols/new_issue_protocol.md](../Protocols/service_protocols/new_issue_protocol.md) | `available` | создание нового service-level issue из input/attachments |
| [../Protocols/service_protocols/existing_issue_protocol.md](../Protocols/service_protocols/existing_issue_protocol.md) | `available` | выбор, продолжение или диагностика существующего issue |
| [../Protocols/service_protocols/question_answer_protocol.md](../Protocols/service_protocols/question_answer_protocol.md) | `available` | закрытие materially important unknowns перед requirements |
| [../Protocols/service_protocols/requirements_protocol.md](../Protocols/service_protocols/requirements_protocol.md) | `available` | requirements draft, review, approval и reopen |
| [../Protocols/service_protocols/solution_contract_output_protocol.md](../Protocols/service_protocols/solution_contract_output_protocol.md) | `available` | solution/contract review, execution и output package |
| [../Protocols/service_protocols/complex_issue_protocol.md](../Protocols/service_protocols/complex_issue_protocol.md) | `available` | parent/children decomposition и requalification |
| [../Protocols/service_protocols/linked_issues_protocol.md](../Protocols/service_protocols/linked_issues_protocol.md) | `available` | dependency edges, readiness, stale и cycle handling |
| [../Protocols/service_protocols/issue_retention_protocol.md](../Protocols/service_protocols/issue_retention_protocol.md) | `available` | archive, tombstone, deletion и Inbox cleanup lifecycle |

## Доступные common/execution resources

| Ресурс | Статус | Роль |
|---|---|---|
| [../Protocols/common/context_loading_protocol.md](../Protocols/common/context_loading_protocol.md) | `available` | context budget и context lift |
| [../Protocols/common/persistence_protocol.md](../Protocols/common/persistence_protocol.md) | `available` | transaction-like persistence |
| [../Protocols/common/final_validation_protocol.md](../Protocols/common/final_validation_protocol.md) | `available` | pre-commit/export validation |
| [../Concepts/README.md](../Concepts/README.md) | `available-empty-layer` | вход в слой концепций |
| [../Concepts/_template/README.md](../Concepts/_template/README.md) | `template` | шаблон concrete concept |
| [../Protocols/execution_protocols/README.md](../Protocols/execution_protocols/README.md) | `available` | execution routing, concept scope и path mapping |
| [../Protocols/execution_protocols/concept_export_protocol.md](../Protocols/execution_protocols/concept_export_protocol.md) | `available` | closed/WIP export package |

## Issue sources

| Файл | Роль | Статус |
|---|---|---|
| [../Issues/issue_registry.md](../Issues/issue_registry.md) | человекочитаемый lifecycle, schema и bootstrap snapshot | `available` |
| [../Issues/issue_registry.jsonl](../Issues/issue_registry.jsonl) | машинный реестр implementation/user/optimizer issue | `available` |
| [../Issues/dependency_graph.jsonl](../Issues/dependency_graph.jsonl) | dependency edges и cycle status | `available` |
| [../Issues/_archive/README.md](../Issues/_archive/README.md) | entry point архива закрытых/отклонённых issue | `available-empty-entrypoint` |
| [../Issues/_tombstones/README.md](../Issues/_tombstones/README.md) | entry point tombstone cleanup | `available-empty-entrypoint` |
| [../Inbox/README.md](../Inbox/README.md) | entry point input/attachments и traceability | `available-empty-entrypoint` |

## Context budget

| Уровень | Разрешённый пакет | Статус |
|---|---|---|
| `entry` | root `README.md`, этот state, page registry, protocol catalog | default |
| `focused` | active issue state, reason, ближайший протокол | после materialization runtime issue-папки или registry-only bootstrap issue |
| `expanded` | parent/linked/dependency summaries и affected files | только с reason |
| `full_scope` | весь service scope | только для финальной проверки или крупного refactor |
| `repository_wide` | широкий обход репозитория | запрещён по умолчанию |

Расширение контекста регулируется [../Protocols/common/context_loading_protocol.md](../Protocols/common/context_loading_protocol.md). Перед расширением агент фиксирует, какой факт нельзя проверить в текущем пакете и какой дополнительный файл нужен. После решения агент сворачивает детали обратно в state.

## Текущие bootstrap issue

| ID | Статус | Смысл |
|---|---|---|
| `EXEC-001` … `EXEC-012` | `closed` | рабочая сеть, State, instructions, protocols, issue model, execution layer, final validation и remediation pass созданы и проверены |
| `USER-001` | `deferred` | служебные скрипты не созданы без отдельного approved issue и cost/benefit decision |
| `USER-002` … `USER-007` | `closed` | пользовательские структурные вопросы покрыты реализацией и validation report |
| `OPT-001` … `OPT-004` | `closed_as_continuous_guard` | optimizer guards покрыты final validation report |

## Persistence guard

Перед ответом, который утверждает изменение состояния, агент обязан применить [../Protocols/common/persistence_protocol.md](../Protocols/common/persistence_protocol.md): перечитать state/registry, составить write set, записать artifacts, обновить индексы и добавить запись в [persistence_log.jsonl](persistence_log.jsonl). Если запись не выполнена, ответ должен быть `pending` или `blocked`, а не заявлением о сохранённом результате.

## Blockers

Блокирующий gate: `pending_manual_reviewer`. Reviewer blockers `CBU-RW-HISTORY-001` и `CBU-RW-BOUND-001` считаются закрытыми только после complete branch readback и повторного manual review.

Неблокирующие deferred items:

- `USER-001`: оценка служебных скриптов вынесена в будущий approved issue; это не блокирует accepted implementation или текущий repair candidate.
- Concrete concept folders не созданы: пользователь не задавал concept slug и initial scope.

<a id="next-step-marker"></a>

## Next-step marker

Next status: `pending_manual_reviewer_recheck_then_squash_merge_fresh_main_readback_and_separate_final_closure_transition`.

Следующий gate: manual reviewer проверяет exact final branch head, сохранение historical evidence, bounded deltas и семь blob SHA. Merge, post-merge `main` readback и положительный final-closure transition находятся вне этой executor-задачи.
