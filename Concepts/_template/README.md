# Шаблон концепции

Parent: [Слой концепций](../README.md)  
Owner issue: `EXEC-011`  
Источник истины: `Concepts/_template/README.md`  
Status: `template`  
Updated: `2026-06-16T16:48:11Z`

## Назначение

Этот файл описывает минимальный шаблон concrete concept. Он не является рабочей концепцией и не должен получать issue, output или export как будто это живой объект. Шаблон задаёт минимальную структуру для будущей сети файлов конкретной концепции.

## Как использовать

1. Открыть [execution protocols](../../Protocols/execution_protocols/README.md).
2. Получить от пользователя или approved issue: `concept_slug`, title, reason, initial scope, boundary.
3. Создать `Concepts/<concept_slug>/README.md` и минимальные local state/registry файлы.
4. Обновить [State/execution_index.md](../../State/execution_index.md), root [page_registry.jsonl](../../State/page_registry.jsonl) и local `State/page_registry.jsonl`.
5. Добавить запись persistence через [persistence_protocol.md](../../Protocols/common/persistence_protocol.md).
6. Не создавать пустые decorative Markdown-файлы в `Pages/`, `Output/` или `Exports/` без реального artifact.

## Минимальные metadata concept README

```text
# <Название концепции>

Parent: [Concepts](../README.md)
Concept slug: <concept_slug>
Owner mode: `Execution Mode`
Источник истины: `Concepts/<concept_slug>/README.md`
Status: `draft`
Updated: <timestamp>

## Назначение

<Коротко: что это за концепция и зачем она существует.>

## Scope

### Входит
- ...

### Не входит
- ...

## Operational model

<Как концепция должна работать как система.>

## Process

<Что и как происходит.>

## Pages

| Page | Role | Status |
|---|---|---|

## Issues

Local registry: `Issues/issue_registry.jsonl`.

## Output

<Ссылки появляются после реального output.>

## Exports

<Ссылки появляются после export package.>

## Next-step marker

Next status: `needs_first_issue_or_page`.
```

## Минимальный bootstrap concrete concept

```text
Concepts/<concept_slug>/
├── README.md
├── State/
│   ├── concept_state.md
│   └── page_registry.jsonl
├── Issues/
│   ├── issue_registry.jsonl
│   ├── dependency_graph.jsonl
│   ├── _archive/README.md
│   └── _tombstones/README.md
├── Pages/
├── Output/
└── Exports/
```

`Pages/`, `Output/` и `Exports/` допускаются как директории без Markdown entrypoint до первого содержательного файла. Если создаётся Markdown entrypoint, он должен попасть в local page registry и иметь parent link.

## `State/concept_state.md` schema

```text
# State концепции — <concept_slug>

Parent: [Concept README](../README.md)
Источник истины: `Concepts/<concept_slug>/State/concept_state.md`
Status: `draft | active | ready_for_closure_review | closed | exported | archived`
Updated: <timestamp>

## Current focus

| Field | Value |
|---|---|
| Active issue | none |
| Active phase | not_started |
| Blocking status | none |
| Export status | none |

## Context budget

| Level | Files |
|---|---|
| entry | README, concept_state, local page registry, local issue registry |
| focused | active issue, selected protocol, affected pages |
| expanded | linked issue and dependency summaries |

## Next-step marker

Next status: `needs_first_issue_or_page`.
```

## Local issue registry

`Concepts/<concept_slug>/Issues/issue_registry.jsonl` использует те же поля, что root [issue registry](../../Issues/issue_registry.md), но `scope_type = execution`, `scope_path = Concepts/<concept_slug>/`, а ID имеет формат `<concept_slug>-ISS-0001`.

## Local dependency graph

`Concepts/<concept_slug>/Issues/dependency_graph.jsonl` хранит только зависимости внутри concept scope. Root dependency graph не должен хранить concept-internal edges без cross-scope reason. Если dependency связывает concept issue с service issue, это cross-scope edge и требует явного reason в обоих registry или service-level decision.

## Local page registry

`Concepts/<concept_slug>/State/page_registry.jsonl` хранит concept-internal Markdown и значимые registry artifacts. Root [page_registry.jsonl](../../State/page_registry.jsonl) обязан знать как минимум concept entrypoint и local registry files, если они созданы в production tree.

Минимальная строка local page registry:

```json
{"path":"README.md","kind":"markdown","parent":null,"owner":"concept","source_of_truth":true,"entrypoint":true,"links":[],"backlinks":[],"orphan":false,"status":"draft"}
```

## Issue workflow внутри концепции

1. Создать reason для concept issue.
2. Провести QA только если unknowns влияют на requirements.
3. Сохранить `requirements.md`.
4. Проверить simple/complex requalification.
5. Создать solution и contract.
6. Выполнить approved work.
7. Сохранить output package.
8. Провести validation.
9. Закрыть issue или вернуть в нужную phase.

## Closure и export

Closure concept scope выполняется только после проверки local pages, local issues, dependencies, output coverage и user approval. Export выполняется через [concept_export_protocol.md](../../Protocols/execution_protocols/concept_export_protocol.md). Если есть open issue, export обязан быть `work_in_progress`.

## Guardrails

- Не копировать этот шаблон как готовую концепцию без заполнения reason и scope.
- Не создавать concrete concept без пользовательского intent.
- Не смешивать service-level issue и concept issue без cross-scope reason.
- Не считать WIP export закрытой концепцией.
- Не добавлять Markdown-файлы без parent/backlink/local registry.
