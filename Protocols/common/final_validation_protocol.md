# Протокол финальной проверки

Parent: [Каталог протоколов](../catalog.md)  
Owner issue: `EXEC-012`  
Источник истины: `Protocols/common/final_validation_protocol.md`  
Status: `available`  
Updated: `2026-06-21T22:15:00Z`

## Назначение

Этот протокол разрешает переход только после проверки связности, состояния, issue-покрытия, dependency readiness, каталога, сохранения, операционной читаемости и границы production-слоя. Для concept export дополнительно проверяются precheck, package consistency, deterministic naming и package-root local-open evidence. Для final acceptance добавляются named evidence matrix, runtime contract coverage и North Star coverage.

Протокол применяется в `Service Mode` и `Execution Mode`. В root scope результат сохраняется в [State/service_validation_report.md](../../State/service_validation_report.md). В concrete concept scope аналогичный отчёт сохраняется в `Concepts/<concept_slug>/State/validation_report.md` или в export package, если это прямо задано локальным протоколом.

Canonical export semantics принадлежат protocol ID `execution/concept_export` по path `Protocols/execution_protocols/concept_export_protocol.md`. Этот файл задаёт validation gates, но не копирует export transaction целиком.

## Минимальные входы

| Вход | Root service scope | Concept scope |
|---|---|---|
| Entry point | [README](../../README.md) | `Concepts/<concept_slug>/README.md` |
| State | [State/service_state.md](../../State/service_state.md) или [State/execution_index.md](../../State/execution_index.md) | `Concepts/<concept_slug>/State/concept_state.md` |
| Page registry | [State/page_registry.jsonl](../../State/page_registry.jsonl) | `Concepts/<concept_slug>/State/page_registry.jsonl` |
| Protocol catalog | [Protocols/catalog.md](../catalog.md) и [catalog.jsonl](../catalog.jsonl) | root catalog плюс local concept protocol notes, если они есть |
| Issue registry | [Issues/issue_registry.jsonl](../../Issues/issue_registry.jsonl) | `Concepts/<concept_slug>/Issues/issue_registry.jsonl` |
| Dependency graph | [Issues/dependency_graph.jsonl](../../Issues/dependency_graph.jsonl) | `Concepts/<concept_slug>/Issues/dependency_graph.jsonl` |
| Persistence log | [State/persistence_log.jsonl](../../State/persistence_log.jsonl) | local persistence log, если он создан |
| Export evidence, когда применимо | `export_precheck`, manifest/page map/included set/local-open report | concept-local export artifacts и state export fields |
| Final-acceptance evidence, когда применимо | approved audit/task-state refs, live transfer-registry rows, accepted stage readbacks | scope-local equivalent или `not_applicable_with_reason` |

## Canonical status vocabulary

Final validation использует только:

```text
pass
pass_with_deferred_items
blocked
```

Export/report literals `pass_with_notes`, `fail` и raw dependency `satisfied` не являются canonical closure statuses. Старые значения явно нормализуются либо блокируют transition при недостаточном evidence.

`pass_with_deferred_items` допустим только когда deferred items неблокирующие, имеют owner, reason и next action. Для closed export дополнительно требуется user acceptance deferred items в closure scope.

## Named evidence matrix

Каждый validation report содержит machine-readable блок со следующей минимальной схемой:

```yaml
validation_scope: root|service|execution|concept|export
base_ref: string
checked_paths: []
changed_paths: []
readback_refs: []
failed_checks: []
registry_evidence: []
state_evidence: []
issue_event_or_persistence_evidence: []
link_backlink_orphan_evidence: []
dependency_evidence: []
catalog_evidence: []
contract_coverage_evidence: []
language_evidence: []
mode_dry_run_evidence: []
conflict_rollback_evidence: []
production_boundary_evidence: []
north_star_coverage_ref: string|null
open_risks: []
residual_ids: []
next_safe_step: string
```

Правила evidence:

1. `checked_paths` и `changed_paths` являются разными множествами; скан или read-only audit не превращается в claim об изменении файла.
2. `pass` и `pass_with_deferred_items` невозможны, если обязательная категория пуста.
3. `not_applicable` допустим только как объект с exact scope, reason и evidence basis.
4. `failed_checks: []` допустим только после заполнения всех обязательных категорий.
5. Слова `passed`, `synced`, `closed`, `ready`, `OK` и self-reported booleans сами по себе не являются evidence.
6. Evidence называет concrete path, ref, blob/commit, registry row, persistence transaction или воспроизводимый dry-run result.
7. Branch result остаётся branch-scoped до merge и fresh `main` readback.

## Жёсткие gates

Проверка не может получить `pass` или `pass_with_deferred_items`, если нарушено хотя бы одно условие:

1. root `README.md` или concept `README.md` отсутствует;
2. Markdown-ссылка ведёт в несуществующий рабочий файл;
3. дочерний Markdown-файл не имеет parent/back link;
4. рабочий Markdown-файл не достижим из entry map и не записан в page registry;
5. `State/page_registry.jsonl` расходится с actual files, links или backlinks;
6. JSONL не парсится построчно или содержит duplicate keys;
7. dependency graph содержит цикл или ссылается на несуществующий issue;
8. blocking issue остаётся `open`, `approved`, `active` или `blocked` без documented blocker/deferred reason;
9. production tree содержит task-state, source notes, checkpoint methodology, prompt, handoff/archive или другой active dev-only material;
10. readable Markdown операционно нечитаем либо противоречит explicit file contract;
11. protocol catalog объявляет `available` протокол без существующего файла;
12. persistence log не содержит factual записи операции или честного pending/package-draft статуса;
13. metadata статусы протокола расходятся между Markdown, catalog JSONL и page registry, когда запись присутствует;
14. production Markdown содержит разговорные или оценочные фразы без операционной роли;
15. active metadata ссылается на development-only material как на working path;
16. агент взаимодействовал с Codex или использовал Codex output как evidence;
17. direct blocking dependency после нормализации имеет readiness `blocked`, `stale`, `cycle_blocked`, `satisfied_for_draft` или `unsatisfied`;
18. closed export не имеет read-only `export_precheck` для той же identity/revision;
19. export type/finality marker не соответствует route либо WIP представлен как final;
20. manifest, page map и фактический included set расходятся;
21. package-root local-open result отсутствует либо closed export имеет local-open fail;
22. package root README отсутствует/не открывается или relative path выходит за package root;
23. deterministic export ID/archive/path не соответствует factual identity/revision/time;
24. export collision не разрешён доказанным exact idempotent retry;
25. compact issue-summary policy нарушена без `context_loss_risk = true` и concrete reason;
26. concept state export fields обновлены до package persistence/readback либо указывают на отсутствующий package;
27. registry содержит candidate/temp/export paths, которых фактически нет;
28. production tree содержит `Concepts/smoke`, tracked smoke fixture или generated test archive;
29. closed export имеет open blocker, non-ready dependency, missing output, language fail или noncanonical validation status;
30. WIP export с defects не содержит exact limitations, `not_final` и acceptance для `draft_with_notice`;
31. обязательная evidence category отсутствует либо заменена общим status label;
32. `not_applicable` не имеет exact reason/scope/evidence;
33. checked/scanned paths смешаны с changed paths;
34. runtime issue closure не покрывает обязательный contract set и не имеет approved exception;
35. North Star registry имеет missing/duplicate ID или total не равен source registry;
36. final-acceptance candidate имеет P0/P1 residual;
37. активное boundary breach маскируется как исторический debris;
38. Stage 07 запущен, merge выполнен или `closure_allowed=true` заявлен без отдельного разрешённого transition.

## GOV-001 — hard gate Codex

Только пользователь может самостоятельно запускать Codex. Это не даёт агенту права взаимодействовать с Codex или использовать его результат. Автоматически появившиеся комментарии Codex игнорируются: агент не отвечает, не включает IDs/statuses в validation/persistence evidence и не считает их проверкой.

Нарушение GOV-001 даёт `blocked`; все изменения после точки нарушения требуют ручного аудита, а Codex-derived evidence удаляется до повторной проверки.

## Dependency normalization для closure и export

Canonical source — [Linked Issues protocol](../service_protocols/linked_issues_protocol.md). Для execution, validation, closed export и closure:

- только normalized `ready` разрешает transition;
- `blocked`, `stale`, `cycle_blocked`, `satisfied_for_draft`, `unsatisfied` блокируют transition;
- raw legacy `status = satisfied` не равен `ready` без required artifact/state evidence;
- observed readiness и evidence каждого direct blocking edge фиксируются в report;
- WIP может описать non-ready dependency только как exact limitation.

## Runtime issue contract coverage

Для runtime issue report перечисляет factual status и evidence для:

```text
reason/input
requirements
QA или explicit skip reason
requalification
solution
contract
output/report
contract coverage
validation
```

Approved exception имеет схему:

```yaml
missing_artifact: string
reason: string
owner: string
risk: string
next_action: string
approval_ref: string
```

Bootstrap registry-only issue может использовать `not_applicable_for_bootstrap` только когда runtime issue folder не требовалась, machine registry содержит factual lifecycle row, а accepted root validation report покрывает соответствующий artifact. Отсутствие runtime folder не превращается в silent pass.

## Final North Star acceptance

Final-acceptance procedure читает approved audit/task-state и live transfer registry как external evidence, но не копирует эти development materials в production. Report обязан:

1. зафиксировать leader repository и frozen base ref;
2. проверить accepted Stage 01–05 evidence против current `main`, а не только исторического текста;
3. оценить `Service Mode` и `Execution Mode` отдельно;
4. сопоставить каждый source registry ID ровно один раз с `accepted_done`, `accepted_deferred`, `rejected_with_reason` или `residual`;
5. для каждой строки указать priority, action/decision, target/evidence, disposition и reason;
6. для deferred/residual указать owner и next action;
7. считать `DO_NOT_TRANSFER` решением `rejected_with_reason`, а не gap;
8. блокировать candidate при missing/duplicate ID и любом P0/P1 residual;
9. вывести counts, `residual_ids`, `stage_07_candidate`, `closure_candidate`, `closure_allowed`.

Допустимый итоговый блок:

```yaml
north_star_registry_total: 74
accepted_done: integer
accepted_deferred: integer
rejected_with_reason: integer
residual: integer
residual_ids: []
stage_07_candidate: skip_no_residuals|required
closure_candidate: true|false
closure_allowed: false
```

`residual_ids=[]` разрешает только branch-scoped `stage_07_candidate=skip_no_residuals` и `closure_candidate=true`. Stage 07 не запускается. `closure_allowed` в этом transition всегда `false`.

## Production boundary и language scope

Blocking boundary breach: task-state/audit/prompt/handoff/archive/methodology в production, новый незарегистрированный runtime path, artifact вне approved scope или active metadata на dev-only target. Historical debris является неблокирующим только когда находится вне active production, не referenced runtime sources и не связан с текущей архитектурой; unrelated cleanup не запускается.

Readable operational Markdown остаётся преимущественно русским. English допустим для paths, IDs, keys, statuses, service names и stable technical terms. Language блокирует только operational unreadability, semantic contradiction или explicit contract violation. Cosmetic repository-wide rewrite не является final-acceptance requirement.

## Export-specific validation contract

### Read-only precheck

Для `closed_concept` validation проверяет тот же `concept_slug`, requested type и `state_revision`; precheck создан до mutation, содержит integrity/readiness/output/registry/issue/dependency/language/validation fields, blockers/limitations/allowed_result/next_step, не создал mutation и дал `allowed_result = closed_concept`.

Для WIP precheck должен дать `allowed_result = work_in_progress`, readable verified state и limitations, совпадающие с package evidence. `not_available` блокирует closed export.

### Package truthfulness

Проверяются: export type/finality; deterministic ID `<concept_slug>__<closed|wip>__r<state_revision>__<YYYYMMDDTHHMMSSZ>`; archive/path; отсутствие random suffix/predicted SHA; manifest/page map/included/excluded; compact `issue_сводка.md`; collision/idempotency; local-open; post-persistence state fields; factual registry/index deltas.

### Package-root local-open

Report фиксирует entrypoint/open result, checked files, missing targets, case mismatches, relative Markdown links вне code fences, exclusions для web/mailto/anchor, root escapes и manifest/content mismatches. Missing root README или root escape блокируют оба export types. Другой fail блокирует closed export; WIP допускает fail только с exact limitations и acceptance.

### Synthetic smoke evidence

Smoke является non-production strategy: closed happy path; blocking issue/dependency; broken relative link; deterministic naming/collision/exact idempotent retry. Evidence происходит из in-memory map или temporary untracked scope. Production smoke artifact запрещён.

## Процедура проверки

### 1. Freeze scope

Зафиксировать scope, checked paths, changed paths, excluded dev-only/temporary materials, base ref и expected transition. Во время validation не добавлять content files вне approved transaction.

### 2. Link/backlink/orphan check

Собрать Markdown, извлечь exact-case relative links вне code fences, проверить targets, parent links, factual backlinks, page registry и orphan state. Для export повторить относительно package root и запретить root escape.

### 3. JSONL, registry и dependency check

Парсить каждую строку с duplicate-key detection; проверить stable issue fields, graph metadata/edges, refs, cycles и normalized readiness. Mismatch даёт `blocked`.

### 4. Protocol catalog check

Все `available` rows [catalog.jsonl](../catalog.jsonl) ведут в existing files; human [catalog](../catalog.md) содержит ту же operational picture; planned/deferred route не исполняется; новые protocol paths имеют registry/parent; final-validation human/machine rows совпадают по inputs, outputs, blockers и completion signal.

### 5. Language, governance и boundary check

Проверить operational readability, neutral style, production top-level boundary, absence dev/smoke artifacts и GOV-001. Historical debris классифицировать отдельно от active breach.

### 6. Metadata, issue и contract coverage

Сверить protocol status metadata. Каждое user/optimizer issue получает closed/deferred/rejected/archived/backlog status. Deferred item имеет owner/reason/next action. Runtime issue получает полный contract coverage либо approved exception; bootstrap-only coverage использует только explicit `not_applicable_for_bootstrap`.

### 7. Mode dry-runs

Root final acceptance содержит воспроизводимые read-only cases: Service Mode без active issue; existing registry-only issue; Execution Mode `no_active_concept`; `active_unknown` recovery; export precheck без concept; persistence stale-SHA/conflict/partial-write recovery. Для каждого указываются input refs, expected route, mutation status и result.

### 8. Export gate, когда применимо

Сверить precheck/revision/type, manifest/page map/included set, naming/collision, local-open, compact issue summary, WIP marker/acceptance, state-update ordering и smoke evidence.

### 9. North Star coverage, когда применимо

Прочитать external approved sources, подтвердить source total, построить unique disposition matrix, проверить counts/P0/P1 residuals, вывести Stage 07 decision и сохранить `closure_allowed=false`.

### 10. Report и persistence

Сохранить report в root [service validation report](../../State/service_validation_report.md) либо concept/export-local path. Обновить page registry только при factual link/backlink/metadata delta. Runtime issue registry обновлять только если closure действительно меняет lifecycle. Добавить factual row в [persistence log](../../State/persistence_log.jsonl) последней file mutation. Branch result не утверждает `main` verification.

## Формат результата

| Status | Когда ставить | Сообщение |
|---|---|---|
| `pass` | blockers/deferred отсутствуют, все evidence categories populated | change готов к следующему разрешённому transition |
| `pass_with_deferred_items` | blockers отсутствуют, deferred документированы | change готов; deferred перечислены |
| `blocked` | evidence, links, JSONL, graph, boundary, language, governance, contract/export/North-Star gate failed | change не готов; нужен repair checkpoint |

WIP `pass` означает только корректность WIP package. Branch-scoped final-acceptance candidate означает только готовность к manual review, не merge и не closure.

## Связанные файлы

- [Root README](../../README.md)
- [Service state](../../State/service_state.md)
- [Execution index](../../State/execution_index.md)
- [Page registry](../../State/page_registry.jsonl)
- [Persistence protocol](persistence_protocol.md)
- [Persistence log](../../State/persistence_log.jsonl)
- [Protocol catalog](../catalog.md)
- [Linked Issues protocol](../service_protocols/linked_issues_protocol.md)
- [Issue registry](../../Issues/issue_registry.md)
- [Dependency graph](../../Issues/dependency_graph.jsonl)
- [Service validation report](../../State/service_validation_report.md)
