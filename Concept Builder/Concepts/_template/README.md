# Шаблон концепции

Parent: [Слой концепций](../README.md)  
Owner issue: `EXEC-011`  
Источник истины: `Concepts/_template/README.md`  
Status: `template`  
Updated: `2026-06-05T11:45:45Z`

## Назначение

Этот файл описывает минимальный шаблон concrete concept. Он не является рабочей концепцией и используется только как основа для будущего `Concepts/<concept_slug>/README.md`.

## Как использовать

1. Открыть [execution protocols](../../Protocols/execution_protocols/README.md).
2. Получить `concept_slug`, title, reason, initial scope и boundary.
3. Создать `Concepts/<concept_slug>/README.md` и минимальные local state/registry files.
4. Обновить [State/execution_index.md](../../State/execution_index.md), root [page_registry.jsonl](../../State/page_registry.jsonl) и local `State/page_registry.jsonl`.
5. Добавить запись persistence через [persistence_protocol.md](../../Protocols/common/persistence_protocol.md).

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

## Local state

`Concepts/<concept_slug>/State/concept_state.md` фиксирует active issue, active phase, blocking status, export status и next-step marker.

## Local issue registry

`Concepts/<concept_slug>/Issues/issue_registry.jsonl` использует те же принципы, что root [issue registry](../../Issues/issue_registry.md), но хранится в concept scope.

## Local page registry

`Concepts/<concept_slug>/State/page_registry.jsonl` хранит concept-internal Markdown и значимые registry artifacts.

## Closure и export

Closure concept scope выполняется только после проверки local pages, local issues, dependencies, output coverage и user approval. Export выполняется через [concept_export_protocol.md](../../Protocols/execution_protocols/concept_export_protocol.md).

## Guardrails

- Не копировать этот шаблон как готовую концепцию без заполнения reason и scope.
- Не создавать concrete concept без пользовательского intent.
- Не смешивать service-level issue и concept issue без cross-scope reason.
- Не считать WIP export закрытой концепцией.
- Не добавлять Markdown-файлы без parent/backlink/local registry.
