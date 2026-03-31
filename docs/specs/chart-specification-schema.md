# Схема спецификации графиков

## Назначение

Этот документ фиксирует канонический контракт `ChartSpec`, который используется между:
- orchestrator;
- analytics layer;
- visualization service;
- sandbox renderer;
- frontend artifact rendering.

Цель схемы — исключить произвольную генерацию графиков и сделать визуализацию полностью управляемой через typed contract.

## Статус документа

- Версия: `0.1.0`
- Статус: `working-draft`
- Владелец: `favead`
- Последнее обновление: `2026-03-31`

## Принципы

- пользователь не передает произвольный код рендера;
- LLM не генерирует низкоуровневый plotting code;
- backend строит только `ChartSpec` + dataset reference;
- renderer работает по allowlist типов графиков и опций;
- все подписи, оси и легенды должны быть безопасны для UI и логов.

## Канонический объект `ChartSpec`

```json
{
  "chart_spec_version": "0.1.0",
  "chart_id": "chart_123",
  "chart_type": "line",
  "title": "Динамика стоимости актива",
  "subtitle": "За последние 30 дней",
  "description": "График построен по дневным свечам T-Invest API",
  "dataset_ref": {
    "dataset_id": "dataset_123",
    "source": "analytics_snapshot",
    "snapshot_id": "asnap_123"
  },
  "x_axis": {
    "field": "date",
    "label": "Дата",
    "type": "datetime"
  },
  "y_axis": {
    "label": "Цена",
    "type": "numeric",
    "currency": "RUB"
  },
  "series": [
    {
      "field": "close",
      "label": "Цена закрытия",
      "color": "#2563eb",
      "aggregation": "none"
    }
  ],
  "legend": {
    "enabled": true,
    "position": "bottom"
  },
  "display": {
    "width": 1280,
    "height": 720,
    "theme": "light",
    "show_grid": true,
    "show_tooltips": false
  },
  "annotations": [],
  "validation": {
    "is_valid": true,
    "warnings": [],
    "errors": []
  }
}
```

## Поддерживаемые типы графиков для PoC

- `line`
- `bar`
- `stacked_bar`
- `pie`
- `area`
- `table_snapshot`

Неподдерживаемые типы для PoC:
- heatmap
- candlestick с пользовательским scripting layer
- arbitrary dashboard layouts
- network graphs

## Источники данных

`dataset_ref.source` может принимать значения:
- `analytics_snapshot`
- `derived_dataset`
- `market_candles`
- `portfolio_table`

Правила:
- большие series не должны встраиваться прямо в `ChartSpec`;
- `ChartSpec` должен ссылаться на dataset, а не дублировать его;
- допускается inline dataset только для очень маленьких summary-chart сценариев.

## Поля осей

### `x_axis.type`

Допустимые значения:
- `datetime`
- `category`
- `numeric`

### `y_axis.type`

Допустимые значения:
- `numeric`
- `percent`
- `money`

## Поля `series`

Каждая серия должна содержать:
- `field`
- `label`
- `aggregation`

Допустимые `aggregation`:
- `none`
- `sum`
- `avg`
- `max`
- `min`

## Annotation contract

```json
[
  {
    "type": "threshold_line",
    "value": 20.0,
    "label": "Лимит стратегии",
    "color": "#dc2626"
  }
]
```

Поддерживаемые аннотации:
- `threshold_line`
- `note`
- `highlight_range`

## Ограничения renderer

- максимальное число точек в одной серии: `5000`;
- максимальное число серий: `10`;
- максимальный размер изображения: `1600x1200`;
- запрещены HTML/JS-вставки в `title`, `subtitle`, `label`;
- все пользовательские подписи проходят sanitization.

## Правила валидации

`ChartSpec` считается невалидным, если:
- `chart_type` не входит в allowlist;
- dataset не содержит указанных `field`;
- число серий превышает лимит;
- размер графика превышает лимит;
- тип осей не соответствует типам данных.

## Ошибки

Рекомендуемые коды ошибок:
- `CHART_SPEC_INVALID`
- `CHART_TYPE_UNSUPPORTED`
- `CHART_DATASET_NOT_FOUND`
- `CHART_FIELD_MISSING`
- `CHART_SERIES_LIMIT_EXCEEDED`
- `CHART_SIZE_LIMIT_EXCEEDED`

## Готовые шаблоны для PoC

### 1. Price history line chart

Используется для:
- динамики цены актива;
- сравнения 1-2 временных рядов.

### 2. Portfolio allocation bar chart

Используется для:
- распределения по asset class;
- распределения по валютам;
- распределения по секторам.

### 3. Top positions pie chart

Используется для:
- визуализации долей крупнейших позиций.

Ограничение:
- не более `7` сегментов, остальное уходит в `Other`.

## Правила хранения

- `ChartSpec` хранится как metadata артефакта;
- dataset хранится отдельно;
- итоговый PNG хранится как artifact;
- renderer должен быть идемпотентным при одинаковом `ChartSpec + dataset_ref`.

## Связанные документы

- `docs/specs/tool-schemas.md`
- `docs/specs/frontend-response-contract.md`
- `docs/specs/analytics-snapshot-schema.md`
- `docs/specs/instrument-mapping-policy.md`
