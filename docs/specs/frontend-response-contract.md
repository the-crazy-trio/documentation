# Контракт ответа frontend/backend

## Назначение

Этот документ задает шаблон контракта между backend и клиентскими интерфейсами `Telegram Bot / Mini App`.

Цель — заранее договориться, как backend возвращает:
- текстовые ответы;
- структурированные summary blocks;
- таблицы;
- disclosure blocks;
- изображения и артефакты;
- action buttons.

Документ подготовлен как шаблон для заполнения.

## Статус документа

- Версия: `0.1.0-draft`
- Статус: `template`
- Владелец: `TODO`

## Общий envelope ответа

```json
{
  "request_id": "req_123",
  "response_type": "analysis",
  "status": "success",
  "summary_text": "Портфель частично соответствует стратегии.",
  "blocks": [],
  "artifacts": [],
  "actions": [],
  "disclosures": [],
  "meta": {
    "generated_at": "2026-03-31T10:00:00Z",
    "partial": false,
    "source_timestamps": []
  }
}
```

## Поля envelope

### `response_type`

Допустимые значения:
- `analysis`
- `clarification`
- `safe_refusal`
- `portfolio_snapshot`
- `asset_report`
- `strategy_fit`
- `visualization`

### `status`

Допустимые значения:
- `success`
- `partial`
- `needs_confirmation`
- `rejected`
- `error`

## Типы блоков

### 1. TextBlock

```json
{
  "type": "text",
  "title": "Краткий вывод",
  "content": "Портфель частично соответствует стратегии..."
}
```

### 2. MetricListBlock

```json
{
  "type": "metric_list",
  "title": "Ключевые показатели",
  "items": [
    {
      "label": "Доля топ-1 позиции",
      "value": "24%",
      "metric_key": "top_1_weight_pct"
    }
  ]
}
```

### 3. TableBlock

```json
{
  "type": "table",
  "title": "Топ позиции",
  "columns": ["Тикер", "Доля"],
  "rows": [
    ["SBER", "24%"]
  ],
  "truncated": false
}
```

### 4. DisclosureBlock

```json
{
  "type": "disclosure",
  "level": "warning",
  "content": "Часть активов не была сматчена, поэтому вывод покрывает 82% стоимости портфеля."
}
```

### 5. ArtifactBlock

```json
{
  "type": "artifact",
  "artifact_id": "art_123",
  "mime_type": "image/png",
  "caption": "Динамика стоимости актива"
}
```

## Action buttons

```json
[
  {
    "action_id": "confirm_strategy_save",
    "label": "Подтвердить",
    "style": "primary"
  },
  {
    "action_id": "cancel",
    "label": "Отмена",
    "style": "secondary"
  }
]
```

## Шаблоны для основных response types

### `strategy_fit`

Должен включать:
- summary_text;
- verdict block;
- key reasons block;
- metrics block;
- disclosure block при partial coverage;
- optional chart artifact.

### `portfolio_snapshot`

Должен включать:
- summary_text;
- top positions table;
- allocation metrics;
- disclosure по отсутствующим данным, если есть.

### `clarification`

Должен включать:
- один конкретный вопрос;
- при необходимости список интерпретированных assumptions;
- actions: `подтвердить` / `исправить`.

## Правила safe output

- нельзя отправлять токены, account ids в полном виде и другие секреты;
- частичные ответы должны явно маркироваться;
- stale data должна раскрываться в `disclosures`;
- unsupported criteria должны быть явно перечислены, если они затрагивают вывод.

## Вопросы на заполнение

1. Mini App и Telegram Bot используют один контракт или разные адаптеры поверх общего envelope?
2. Какие блоки обязательны для каждого response type?
3. Нужно ли поддерживать markdown/html formatting в `summary_text`?
4. Нужен ли отдельный тип блока для рекомендаций/next steps?

## Связанные документы

- `docs/system-design.md`
- `docs/specs/tool-schemas.md`
- `docs/specs/strategy-schema.md`
