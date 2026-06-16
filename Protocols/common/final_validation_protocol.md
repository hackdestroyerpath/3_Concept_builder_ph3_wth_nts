# Протокол финальной проверки

Parent: [Каталог протоколов](../catalog.md)  
Owner issue: `EXEC-012`  
Источник истины: `Protocols/common/final_validation_protocol.md`  
Status: `available`  
Updated: `2026-06-05T11:45:45Z`

## Назначение

Этот протокол закрывает рабочие изменения только после проверки связности, состояния, issue-покрытия, нейтральности стиля и границы production-слоя. Без этой проверки репозиторий может стать несогласованным набором Markdown-фрагментов вместо рабочей сети файлов.

Протокол применяется в `Service Mode` и в `Execution Mode`. В root scope результат сохраняется в [State/service_validation_report.md](../../State/service_validation_report.md). В concrete concept scope аналогичный отчёт сохраняется в локальном `Concepts/<concept_slug>/State/validation_report.md` или в export-пакете, если это прямо задано локальным протоколом.

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

## Жёсткие gates

Проверка не может получить `pass`, если нарушено хотя бы одно условие:

1. root `README.md` или concept `README.md` отсутствует;
2. Markdown-ссылка ведёт в несуществующий рабочий файл;
3. дочерний Markdown-файл не имеет parent/back link;
4. рабочий Markdown-файл не достижим из entry map и не записан в page registry;
5. `State/page_registry.jsonl` расходится с фактическими файлами, links или backlinks;
6. JSONL-файл не парсится построчно;
7. dependency graph содержит цикл или ссылается на несуществующий issue;
8. blocking issue остаётся `open`, `approved`, `active` или `blocked` без documented blocker/deferred reason;
9. production tree содержит ТЗ, source notes, checkpoint methodology, validation сырьё разработки или другой dev-only материал;
10. readable Markdown не на русском языке, кроме технических путей, ID, статусов, JSONL-ключей и имён сервисов;
11. protocol catalog объявляет `available` протокол без существующего файла;
12. persistence log не содержит записи о проверяемой операции или честного `package_draft_not_committed` статуса;
13. metadata статусы одного и того же протокола расходятся между Markdown-шапкой, [catalog.jsonl](../catalog.jsonl) и [page_registry.jsonl](../../State/page_registry.jsonl);
14. production Markdown содержит разговорные, оценочные или саркастические фразы, не имеющие операционной роли;
15. production metadata ссылается на development-only материалы как на рабочие пути репозитория.

`pass_with_deferred_items` разрешён только когда deferred items явно неблокирующие, имеют owner, reason и next action.

## Процедура проверки

### 1. Freeze проверяемого scope

1. Определи root или concept scope.
2. Зафиксируй список файлов scope.
3. Отдельно зафиксируй dev-only материалы, которые не должны попасть в commit-ready package.
4. Не добавляй новые содержательные файлы во время проверки, кроме validation report, page registry update и persistence marker.

### 2. Link/backlink/orphan check

1. Собери все Markdown-файлы scope.
2. Извлеки относительные Markdown-ссылки, игнорируя code fences, web links, anchors и mailto.
3. Проверь существование target-файлов.
4. Построй фактические backlinks.
5. Сравни фактические backlinks с page registry.
6. Проверь, что каждый дочерний Markdown-файл содержит parent link в шапке.
7. Для каждого orphan-кандидата выбери одно действие: добавить ссылку, добавить registry entry или удалить лишний файл.

### 3. JSONL и registry check

1. Каждый `.jsonl` парсится как независимые JSON objects, по строке за раз.
2. `issue_registry.jsonl` содержит стабильные `issue_id`, `status`, `phase`, `dependency_refs`, `validation_path`, `next_action`.
3. `dependency_graph.jsonl` содержит metadata record и edge records.
4. Все `dependency_refs` у issue существуют в graph.
5. Blocking edges не создают cycles.
6. Зависимая задача не marked ready, если blocking dependency не satisfied/deferred/rejected/not_applicable.

### 4. Protocol catalog check

1. Все записи [catalog.jsonl](../catalog.jsonl) со статусом `available` ведут в существующие файлы.
2. [catalog.md](../catalog.md) содержит ту же операционную картину, что JSONL companion.
3. Planned protocol не исполняется как available.
4. Для новых protocol-файлов есть запись в page registry и parent link.

### 5. Language, style и boundary check

1. Читаемые Markdown-файлы проверяются на русский основной язык.
2. Английский допускается для путей, ID, статусов, JSONL keys, service names и устойчивых технических терминов.
3. Production Markdown должен быть операционным и нейтральным: без разговорных вставок, шуток, сарказма и оценочных метафор.
4. Production tree проверяется против верхних директорий: `README.md`, `State/`, `Instructions/`, `Protocols/`, `Issues/`, `Inbox/`, `Concepts/`.
5. Dev-only материалы остаются вне production tree.
6. Metadata может ссылаться на исходный handoff как на внешний источник происхождения, но не должна указывать development-only файлы как рабочие target paths.

### 6. Metadata, issue и contract coverage

1. Для protocol-файлов сверить `Status` в Markdown-шапке, `status` в [catalog.jsonl](../catalog.jsonl) и `status` в [page_registry.jsonl](../../State/page_registry.jsonl), если файл есть в обоих реестрах.
2. Все явно пользовательские issue и optimizer-detected issue получают один из статусов: `closed`, `deferred`, `rejected`, `archived` или documented backlog entry.
3. Для каждого deferred item есть reason и next action.
4. Контракт проверяется по пунктам: рабочая сеть, связи, no orphan, no dev materials, state/protocol/issue/concept sources of truth, русский язык, checkpoint continuity и финальный report.
5. Runtime requirements проверяются только для реально существующих runtime issue. Если runtime issue не создавались, статус requirements coverage — `not_applicable_for_bootstrap`.

### 7. Report и persistence

1. Сохрани validation report: [State/service_validation_report.md](../../State/service_validation_report.md) для root scope.
2. Обнови [State/page_registry.jsonl](../../State/page_registry.jsonl), если report или protocol добавлены.
3. Обнови [Issues/issue_registry.jsonl](../../Issues/issue_registry.jsonl), если проверка закрывает bootstrap issue.
4. Добавь запись в [State/persistence_log.jsonl](../../State/persistence_log.jsonl).
5. Только после этого отвечай пользователю финальным статусом.

## Формат результата

| Status | Когда ставить | Что сказать пользователю |
|---|---|---|
| `pass` | blockers и deferred items отсутствуют | пакет готов к commit |
| `pass_with_deferred_items` | blockers отсутствуют, но есть явно deferred non-blocking items | пакет готов к commit, deferred items перечислены |
| `blocked` | есть broken links, orphan, graph cycle, boundary breach, language failure или blocking issue | пакет не готов, нужен repair checkpoint |

## Связанные файлы

- [Root README](../../README.md)
- [Service state](../../State/service_state.md)
- [Execution index](../../State/execution_index.md)
- [Page registry](../../State/page_registry.jsonl)
- [Persistence protocol](persistence_protocol.md)
- [Persistence log](../../State/persistence_log.jsonl)
- [Protocol catalog](../catalog.md)
- [Issue registry](../../Issues/issue_registry.md)
- [Dependency graph](../../Issues/dependency_graph.jsonl)
- [Service validation report](../../State/service_validation_report.md)
