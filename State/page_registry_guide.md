# Руководство по page registry

Parent: [page_registry.jsonl](page_registry.jsonl)  
Источник истины: [page_registry.jsonl](page_registry.jsonl)  
Readable companion: [navigation_map.md](navigation_map.md)  
Machine companion: [file_manifest.jsonl](file_manifest.jsonl)  
Status: `active`  
Updated: `2026-06-16T20:44:45Z`

## Назначение

Этот guide объясняет поля [page_registry.jsonl](page_registry.jsonl) и порядок проверки достижимости. Он не является вторым source of truth. При расхождении guide, navigation map или file manifest с [page_registry.jsonl](page_registry.jsonl) приоритет остаётся у registry.

## Поля registry

| Поле | Значение |
|---|---|
| `path` | Root-relative путь production-файла. Для Markdown это рабочий route; для JSONL/data это machine artifact. |
| `parent` | Родительский route или `null`, если файл является root/data source. Для Markdown parent должен совпадать с header-ссылкой или быть явно объяснён. |
| `links` | Список root-relative файлов, на которые ссылается страница или registry entry. |
| `backlinks` | Список файлов, которые ссылаются на этот `path`. Поле используется для orphan-check и review reachability. |
| `orphan` | `false`, если файл достижим из root/service/execution route или является допустимым data source. `true` требует repair или documented reason. |
| `entrypoint` | `true`, если файл может быть стартовой точкой маршрута для режима, слоя или раздела. |
| `kind` | Тип файла: `markdown`, `jsonl`, `data` или другой текущий machine kind repo. |
| `source_of_truth` | `true`, если файл является главным источником факта, состояния или registry для своей области. |

## Как добавлять новую Markdown-страницу

1. Проверить, что Stage/issue разрешает новый path.
2. Создать страницу с H1 и parent link в header.
3. Указать source-of-truth note или явно назвать companion role.
4. Добавить страницу в [page_registry.jsonl](page_registry.jsonl) с `path`, `parent`, `links`, `backlinks`, `entrypoint`, `orphan=false`, `role`, `status` и `updated_at`.
5. Если файл является production companion, добавить строку в [file_manifest.jsonl](file_manifest.jsonl).
6. Обновить [navigation_map.md](navigation_map.md), только если новый файл должен быть виден в human route.
7. Добавить transaction-like запись в [persistence_log.jsonl](persistence_log.jsonl).

## Как проверять достижимость

1. Начать от [../README.md](../README.md).
2. Выбрать service или execution route по текущему mode.
3. Проверить, что каждый target Markdown имеет parent link.
4. Проверить, что каждый `links` target существует в production tree или является разрешённым future placeholder внутри runtime issue/concept scope.
5. Проверить, что `backlinks` отражают фактические links изменённых файлов.
6. Распарсить JSONL-файлы построчно: [page_registry.jsonl](page_registry.jsonl), [file_manifest.jsonl](file_manifest.jsonl), [persistence_log.jsonl](persistence_log.jsonl), issue registry и dependency graph.
7. Зафиксировать результат в [service_validation_report.md](service_validation_report.md), если проверяется root service scope.

## Что не добавлять

Не добавлять в registry dev-only prompts, handoff/checkpoint материалы, Phase 2/P2R5 repair-history, donor repository layout wholesale, service scripts без approved issue и runtime concept folders без полного concept scope.
