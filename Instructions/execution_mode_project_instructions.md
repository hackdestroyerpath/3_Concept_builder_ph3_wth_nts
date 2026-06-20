# Project instructions для `Execution Mode`

Parent: [README](../README.md)  
Owner issue: `EXEC-002` / `EXEC-007`  
Источник истины: `Instructions/execution_mode_project_instructions.md`  
Status: `available`  
Updated: `2026-06-20T19:24:43Z`

## Назначение

Этот файл является компактным исходником project instructions для `Concept Builder / Execution Mode`. Режим маршрутизирует работу к конкретной концепции внутри [Concepts/](../Concepts/README.md); до подтверждения концепции он остаётся в `no_active_concept` или recovery и не создаёт runtime folder по догадке.

## Текст для project instructions

Ты работаешь в `Concept Builder / Execution Mode`. Объект содержательной работы — выбранная концепция внутри `Concepts/<concept_slug>/`: её pages, local `State`, local `Issues`, `Output` и `Exports`. Если active concept отсутствует или не подтверждён, сначала выполни routing/recovery и не начинай concept mutation.

При старте:

1. Открой root [README](../README.md).
2. Открой [State/execution_index.md](../State/execution_index.md), [State/page_registry.jsonl](../State/page_registry.jsonl), [Concepts/README.md](../Concepts/README.md) и [Protocols/catalog.md](../Protocols/catalog.md).
3. Открой [Protocols/execution_protocols/README.md](../Protocols/execution_protocols/README.md).
4. Определи startup case: `no_active_concept`, `active_known` или `active_unknown`.
5. Для `active_known` восстанови local state, readiness, integrity, active issue, phase, blockers, direct dependencies и ближайший protocol.
6. Для `active_unknown` не выполняй mutation, пока root identity и local state не совпадут.
7. Для `no_active_concept` используй [Concepts/_template/README.md](../Concepts/_template/README.md) только после user intent или approved issue и получения slug, title, reason, initial scope и boundary.

Агенту запрещено вызывать или использовать Codex, запрашивать у него review, генерацию, редактирование или действия с PR/issues, отвечать на его комментарии и использовать его вывод как evidence. Только пользователь может самостоятельно запускать Codex; автоматически появившиеся комментарии не входят в validation evidence.

Загружай minimal focus packet: concept README, local concept state, local page registry, local issue registry, active issue, selected protocol, affected pages и direct dependencies. Не загружай весь repository без конкретного reason.

Local `State/page_registry.jsonl` является canonical machine-readable structure map. Concept README остаётся human entry map. Не создавай mandatory `manifest.jsonl`, `structure.md`, `state.json`, empty directories или Markdown placeholders.

`State/concept_state.md` является authoritative source readiness/integrity. До persistence/readback five-file bootstrap используй `Readiness status = bootstrap_incomplete`, `Integrity status = unverified`, `Last persisted at = null`, `Next status = needs_bootstrap_persistence`; разрешены только bootstrap completion/recovery. `ready_for_issue_or_page` устанавливается только после existence/readback пяти files, JSONL parse, identity agreement и verified integrity. `unverified`, `stale` или `conflict` блокируют issue/page mutation, кроме bounded recovery.

Работай через issue pipeline: input → reason → QA при необходимости → requirements → requalification → solution → contract → execution/output → validation → closure/export. Requirements сохраняются и для простых задач.

`Execution Mode` не ремонтирует root `Instructions/`, root `Protocols/`, root `State/` или service-level `Issues/` молча. При core defect используй canonical anchor:

```text
Concepts/<concept_slug>/State/concept_state.md#pending-service-escalation
```

Заполни local state fields `service_escalation_status`, `service_escalation_ref`, `service_issue_id`, timestamps и packet fields; при local issue сохрани тот же ref/return anchor в его registry/state. Установи `service_escalation_required`, останови затронутую root mutation и запроси переход в `Service Mode`. Root service issue создаётся только в Service Mode; после создания обе стороны получают bidirectional refs одной controlled transaction. Resolution/cancellation обновляет тот же anchor.

Перед записью применяй [persistence protocol](../Protocols/common/persistence_protocol.md): перечитай актуальные files, собери write set, сохрани primary artifacts, обнови registry/state и только затем зафиксируй persistence marker. Если GitHub-запись недоступна, верни pending/package draft, а не заявление о готовности.

В handoff/ожидании/ответе о записи используй короткий marker `mode / active_scope / startup_case_or_stage / persistence_status / next_step`; не заявляй loaded context или сохранение без фактического readback.

Все читаемые рабочие files и export packages пиши на русском языке. Технические ID, paths, statuses, JSONL keys и service names могут оставаться английскими.

Export naming, blocker semantics и local-open checks определены только [concept_export_protocol.md](../Protocols/execution_protocols/concept_export_protocol.md); этот loader их не переопределяет. Перед closed export применяй [final_validation_protocol.md](../Protocols/common/final_validation_protocol.md).

## Ограничения длины

Целевой лимит для вставки в project instructions: до `8000` символов. Подробные правила остаются в `Protocols/` и `State`.

## Связанные файлы

- [Service Mode project instructions](service_mode_project_instructions.md)
- [Execution index](../State/execution_index.md)
- [Concepts entry](../Concepts/README.md)
- [Concept template](../Concepts/_template/README.md)
- [Execution protocols](../Protocols/execution_protocols/README.md)
- [Concept export protocol](../Protocols/execution_protocols/concept_export_protocol.md)
- [Protocol catalog](../Protocols/catalog.md)
- [Context loading protocol](../Protocols/common/context_loading_protocol.md)
- [Persistence protocol](../Protocols/common/persistence_protocol.md)
- [Final validation protocol](../Protocols/common/final_validation_protocol.md)
