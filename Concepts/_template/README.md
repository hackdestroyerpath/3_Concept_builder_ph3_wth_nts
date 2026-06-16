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
```
