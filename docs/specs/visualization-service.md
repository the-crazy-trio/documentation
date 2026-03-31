# Спецификация Visualization Service

## Назначение

`Visualization Service` отвечает за подготовку данных для графиков и безопасный вызов sandbox renderer для генерации PNG-артефактов.

Компонент нужен для того, чтобы:
- отделить подготовку chart dataset от рендеринга;
- не допускать произвольный plotting code;
- обеспечивать единый contract для графиков и таблиц;
- переиспользовать chart artifacts между запросами.

## Входы

- `ChartSpec`
- dataset reference или маленький inline dataset
- request metadata
- user scope metadata

Точная схема `ChartSpec` описана в `docs/specs/chart-specification-schema.md`.

## Выходы

- artifact metadata (`artifact_id`, `mime_type`, `width`, `height`, `created_at`)
- optional dataset metadata
- validation warnings, если visualization была деградирована

Точный output schema описан в `docs/specs/tool-schemas.md`.

## Внутренний pipeline

1. Resolve dataset
- получить dataset из `AnalyticsSnapshot`, market candles или derived table;
- проверить, что user scope соблюден.

2. Validate `ChartSpec`
- проверить тип графика, поля осей, series и renderer limits;
- убедиться, что dataset содержит нужные поля.

3. Prepare rendering payload
- нормализовать подписи;
- ограничить размер dataset;
- применить fallback для больших series.

4. Call sandbox renderer
- передать только typed payload;
- дождаться PNG artifact metadata.

5. Persist artifact refs
- сохранить artifact metadata и ссылки для последующего использования.

## Разрешенные возможности

Разрешено:
- line/bar/stacked_bar/pie/area/table_snapshot;
- small summary tables;
- threshold annotations и другие allowlisted annotations.

Запрещено:
- arbitrary code execution;
- network access в renderer;
- доступ к произвольной файловой системе;
- неподдерживаемые типы графиков.

## Failure modes

- `CHART_SPEC_INVALID`
- `CHART_TYPE_UNSUPPORTED`
- `CHART_DATASET_NOT_FOUND`
- `CHART_FIELD_MISSING`
- `CHART_SERIES_LIMIT_EXCEEDED`
- `CHART_SIZE_LIMIT_EXCEEDED`
- `RENDER_TIMEOUT`
- `RENDER_FAILED`

## Fallback policy

- если dataset слишком большой, сервис должен пытаться сократить объем до допустимого summary;
- если график нельзя безопасно построить, сервис должен вернуть ошибку, а не деградировать в произвольную картинку;
- если renderer unavailable, оркестратор должен вернуть текстовый partial answer без артефакта.

## Storage policy

- `ChartSpec` хранится как metadata;
- dataset хранится отдельно при необходимости повторного построения;
- PNG хранится как artifact;
- артефакты могут иметь TTL и быть удаляемыми без потери основных аналитических данных.

## Метрики качества и надежности

- render success rate;
- render latency;
- invalid chart spec rate;
- artifact reuse rate;
- dataset truncation rate.

## Зависимости

- `docs/specs/chart-specification-schema.md`
- `docs/specs/tool-schemas.md`
- `docs/specs/frontend-response-contract.md`
- `docs/specs/analytics-snapshot-schema.md`
