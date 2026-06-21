# Протокол финальной проверки

Parent: [Каталог протоколов](../catalog.md)  
Owner issue: `EXEC-012`  
Источник истины: `Protocols/common/final_validation_protocol.md`  
Status: `available`  
Updated: `2026-06-21T03:03:27Z`

## Назначение

Этот протокол закрывает рабочие изменения только после проверки связности, состояния, issue-покрытия, dependency readiness, нейтральности стиля и границы production-слоя. Для concept export он дополнительно проверяет precheck, package consistency, deterministic naming и package-root local-open evidence.

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
| Export evidence, когда применимо | `export_precheck` result, manifest/page map/included set/local-open report | concept-local export artifacts и state export fields |

## Canonical status vocabulary

Final validation использует только:

```text
pass
pass_with_deferred_items
blocked
```

Export/report literals `pass_with_notes`, `fail` и raw dependency `satisfied` не являются canonical closure statuses. Старые значения должны быть явно нормализованы до одного из трёх статусов либо блокировать closure при недостаточном evidence.

`pass_with_deferred_items` допустим только когда deferred items неблокирующие, имеют owner, reason и next action. Для closed export дополнительно требуется user acceptance этих deferred items для closure scope.

## Жёсткие gates

Проверка не может получить `pass` или `pass_with_deferred_items`, если нарушено хотя бы одно условие:

1. root `README.md` или concept `README.md` отсутствует;
2. Markdown-ссылка ведёт в несуществующий рабочий файл;
3. дочерний Markdown-файл не имеет parent/back link;
4. рабочий Markdown-файл не достижим из entry map и не записан в page registry;
5. `State/page_registry.jsonl` расходится с фактическими файлами, links или backlinks;
6. JSONL-файл не парсится построчно или содержит duplicate keys;
7. dependency graph содержит цикл или ссылается на несуществующий issue;
8. blocking issue остаётся `open`, `approved`, `active` или `blocked` без documented blocker/deferred reason;
9. production tree содержит ТЗ, source notes, checkpoint methodology, validation сырьё разработки или другой dev-only материал;
10. readable Markdown не на русском языке, кроме технических путей, ID, статусов, JSONL-ключей и имён сервисов;
11. protocol catalog объявляет `available` протокол без существующего файла;
12. persistence log не содержит записи о проверяемой операции или честного `package_draft_not_committed` / pending статуса;
13. metadata статусы одного и того же протокола расходятся между Markdown-шапкой, [catalog.jsonl](../catalog.jsonl) и [page_registry.jsonl](../../State/page_registry.jsonl);
14. production Markdown содержит разговорные, оценочные или саркастические фразы, не имеющие операционной роли;
15. production metadata ссылается на development-only материалы как на рабочие paths репозитория;
16. агент вызывал или использовал Codex bot, запрашивал у него review, генерацию, редактирование либо действия с PR/issues, отвечал на его комментарии или использовал его вывод как evidence;
17. direct blocking dependency после нормализации имеет readiness `blocked`, `stale`, `cycle_blocked`, `satisfied_for_draft` или `unsatisfied`;
18. closed export не имеет сохранённого read-only `export_precheck` result для того же concept identity/state revision;
19. export type/finality marker не соответствует фактическому route либо WIP представлен как final;
20. manifest, page map и фактический included set расходятся;
21. package-root local-open result отсутствует либо closed export имеет local-open fail;
22. package root README отсутствует/не открывается или relative path выходит за package root;
23. deterministic export ID/archive/path не соответствует factual concept slug, export type, state revision и UTC timestamp;
24. export path collision не разрешён доказанным exact idempotent retry;
25. compact issue-summary policy нарушена: full issue folders включены без `context_loss_risk = true` и concrete inclusion reason;
26. concept state export fields обновлены до package persistence/readback либо ссылаются на несуществующий package;
27. registry содержит candidate/temp/export paths, которых фактически нет;
28. production tree содержит `Concepts/smoke`, tracked smoke fixture, generated test archive или другой production smoke artifact;
29. closed export имеет open blocking issue, dependency не normalized `ready`, required output missing, language fail или validation status вне canonical `pass|pass_with_deferred_items`;
30. WIP export с defects не содержит exact limitations, explicit `not_final` marker и user acceptance для `draft_with_notice`.

## GOV-001 — hard gate Codex

Только пользователь может самостоятельно запускать Codex. Это не даёт агенту права взаимодействовать с Codex или использовать его результат. Автоматически появившиеся комментарии Codex игнорируются: агент не отвечает на них, не включает их IDs или статусы в validation/persistence evidence и не считает их проверкой.

Если агент нарушил GOV-001, текущая проверка получает `blocked`, а все изменения после точки нарушения требуют отдельного ручного аудита. Codex-derived evidence удаляется из production report/log до повторной проверки.

## Dependency normalization для closure и export

Canonical source для dependency normalization — [Linked Issues protocol](../service_protocols/linked_issues_protocol.md). Этот протокол не дублирует lifecycle table, а использует нормализованный результат и его evidence.

Для execution, validation, closed export и closure действует gate:

- только normalized `ready` разрешает переход;
- `blocked`, `stale`, `cycle_blocked`, `satisfied_for_draft` и `unsatisfied` блокируют переход;
- raw legacy `status = satisfied` не равен `ready` без required artifact/state evidence;
- observed readiness и evidence каждого direct blocking edge фиксируются в validation report;
- WIP может описать non-ready dependency только как exact limitation и никогда не превращает её в closure-ready edge.

## Export-specific validation contract

### Read-only precheck

Для `closed_concept` validation проверяет, что `export_precheck`:

- относится к тому же `concept_slug`, requested export type и `state_revision`;
- создан до package mutation;
- содержит integrity/readiness/output/page-registry/issue/dependency/language/validation fields;
- содержит `blockers`, `limitations`, `allowed_result` и `next_step`;
- не создал directory/archive/registry/state/persistence mutation;
- дал `allowed_result = closed_concept`.

Для WIP precheck должен дать `allowed_result = work_in_progress`, concept state должен быть readable, integrity — `verified`, а limitations должны совпадать с package evidence. `not_available` validation в precheck блокирует closed export.

### Package truthfulness

Проверяются одновременно:

1. export type и finality;
2. deterministic ID `<concept_slug>__<closed|wip>__r<state_revision>__<YYYYMMDDTHHMMSSZ>`;
3. archive name `<export_id>.zip` и concept-local export path;
4. отсутствие random suffix и predicted commit SHA;
5. manifest/page map/included/excluded lists;
6. compact `issue_сводка.md` и evidence для исключений;
7. collision/idempotency readback;
8. local-open result и exact broken targets;
9. post-persistence concept-state export fields;
10. truthful root/local registry and execution-index deltas.

### Package-root local-open

Validation report должен фиксировать:

- declared entrypoint и факт открытия root `README.md`;
- checked files;
- missing targets и case mismatches;
- relative Markdown links вне code fences;
- exclusion external URL, `mailto:` и pure anchors из file-target resolution;
- root-escape targets;
- manifest/content mismatches.

Missing/unopenable root README или root escape блокируют любой export. Любой другой local-open fail блокирует closed export. WIP допускает fail только по `draft_with_notice` с exact limitations и user acceptance.

### Synthetic smoke evidence

Smoke evidence проверяется как non-production strategy. Минимальный набор сценариев:

1. closed happy path;
2. blocking issue/dependency с blocked closed и описанным WIP route;
3. broken relative package link с local-open fail;
4. deterministic naming, collision и exact idempotent retry.

Evidence может происходить из in-memory map или temporary untracked directory вне production tree. Никакой smoke file не регистрируется как production page.

## Процедура проверки

### 1. Freeze проверяемого scope

1. Определи root, concept или export scope.
2. Зафиксируй список файлов scope.
3. Отдельно зафиксируй dev-only и temporary materials, которые не должны попасть в commit/package.
4. Для export зафиксируй concept identity, state revision, requested type, candidate/persisted ID и package root.
5. Не добавляй новые содержательные файлы во время проверки, кроме validation report, page registry update и persistence marker, разрешённых transaction plan.

### 2. Link/backlink/orphan check

1. Собери все Markdown-файлы scope.
2. Извлеки relative Markdown links, игнорируя code fences, web links, anchors и `mailto:`.
3. Проверь существование target-файлов и exact case.
4. Построй фактические backlinks.
5. Сравни фактические backlinks с page registry.
6. Проверь, что каждый дочерний Markdown-файл содержит parent link в шапке.
7. Для каждого orphan-кандидата выбери одно действие: добавить link, добавить registry entry или удалить лишний файл по отдельному разрешённому write set.
8. Для export повтори resolution относительно package root и запрети root escape.

### 3. JSONL, registry и dependency readiness check

1. Каждый `.jsonl` парсится как независимые JSON objects, по строке за раз, с duplicate-key detection.
2. `issue_registry.jsonl` содержит стабильные `issue_id`, `status`, `phase`, `dependency_refs`, `validation_path`, `next_action`.
3. `dependency_graph.jsonl` содержит metadata record и edge records.
4. Все `dependency_refs` у issue существуют в graph.
5. Blocking edges не создают cycles.
6. Нормализуй каждый direct blocking edge строго по [Linked Issues protocol](../service_protocols/linked_issues_protocol.md).
7. Разрешай execution, validation, closed export и closure только при normalized `ready`.
8. Graph/registry mismatch или evidence conflict делает validation `blocked` до repair.

### 4. Protocol catalog check

1. Все записи [catalog.jsonl](../catalog.jsonl) со status `available` ведут в существующие files.
2. [catalog.md](../catalog.md) содержит ту же operational picture, что JSONL companion.
3. Planned protocol не исполняется как available.
4. Для новых protocol-файлов есть page-registry entry и parent link.
5. Для `execution/concept_export` human/machine rows одинаково описывают triggers, verified inputs, precheck/package outputs, blockers и completion signal.

### 5. Language, style, governance и boundary check

1. Readable Markdown проверяется на русский основной язык.
2. Английский допускается для paths, ID, statuses, JSONL keys, service names и устойчивых technical terms.
3. Production Markdown должен быть operational и neutral: без разговорных вставок, шуток, сарказма и оценочных метафор.
4. Production tree проверяется против верхних directories: `README.md`, `State/`, `Instructions/`, `Protocols/`, `Issues/`, `Inbox/`, `Concepts/`.
5. Dev-only и smoke temporary materials остаются вне production tree.
6. Metadata может ссылаться на внешний handoff как provenance, но не указывает development-only files как working target paths.
7. Execution history и production evidence проверяются на GOV-001; автоматически появившиеся Codex comments не учитываются.
8. Для export подтверждается отсутствие production `Concepts/smoke`, tracked fixture и generated test archive.

### 6. Metadata, issue и contract coverage

1. Для protocol-файлов сверить `Status` в Markdown-шапке, `status` в [catalog.jsonl](../catalog.jsonl) и `status` в [page_registry.jsonl](../../State/page_registry.jsonl), если файл есть в обоих реестрах.
2. Все явно пользовательские issue и optimizer-detected issue получают один из statuses: `closed`, `deferred`, `rejected`, `archived` или documented backlog entry.
3. Для каждого deferred item есть owner, reason и next action.
4. Контракт проверяется по пунктам: рабочая сеть, links, no orphan, no dev materials, state/protocol/issue/concept sources of truth, русский язык, checkpoint continuity и final report.
5. Runtime requirements проверяются только для реально существующих runtime issue. Если runtime issue не создавались, status requirements coverage — `not_applicable_for_bootstrap`.
6. Для закрываемого issue validation report перечисляет normalized direct dependency readiness и evidence для каждого blocking edge.
7. Для export validation report перечисляет precheck fields, blocker/limitation outcome, naming evidence, local-open result, issue-summary policy и state-update ordering.

### 7. Export gate check, когда применимо

1. Сверить precheck и post-assembly state revision; revision drift требует нового precheck.
2. Сверить requested type, allowed result и persisted type.
3. Проверить closed/WIP matrix без silent downgrade.
4. Проверить manifest/page map/included set/excluded refs.
5. Проверить deterministic name и collision/idempotency evidence.
6. Проверить package-root local-open и hard blockers.
7. Проверить compact issue summary и `context_loss_risk` exceptions.
8. Проверить explicit WIP `not_final` marker, limitations и user acceptance.
9. Проверить concept state export fields только после persistence/readback.
10. Проверить synthetic smoke evidence без production artifacts.

### 8. Report и persistence

1. Сохрани validation report: [State/service_validation_report.md](../../State/service_validation_report.md) для root scope либо concept/export-local report для concept scope.
2. Обнови [State/page_registry.jsonl](../../State/page_registry.jsonl), только если report/protocol добавлены либо фактические link/backlink sets изменились.
3. Обнови [Issues/issue_registry.jsonl](../../Issues/issue_registry.jsonl), если проверка закрывает runtime issue.
4. Добавь factual entry в [State/persistence_log.jsonl](../../State/persistence_log.jsonl) либо local persistence log.
5. Если normalized edge state становится новым operational source of truth, обнови graph и registry mirrors одной controlled transaction до closure.
6. Для export не записывай candidate package path как persisted до readback.
7. Только после этого отвечай пользователю final status.

## Формат результата

| Status | Когда ставить | Что сказать пользователю |
|---|---|---|
| `pass` | blockers и deferred items отсутствуют | package/change готов к следующему разрешённому transition |
| `pass_with_deferred_items` | blockers отсутствуют, deferred items явно неблокирующие и документированы | package/change готов; deferred items перечислены |
| `blocked` | broken links, orphan, graph/dependency failure, export gate failure, GOV-001, boundary, language или blocking issue | package/change не готов; нужен repair checkpoint |

Для WIP export даже `pass` означает только корректность WIP package, а не closure концепции.

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
