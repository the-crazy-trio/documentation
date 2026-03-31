# Схемы входов, выходов и ошибок инструментов

## Единый формат ошибки

Все внутренние инструменты должны возвращать ошибки в едином формате:

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Field 'user_id' is required",
    "retryable": false,
    "source": "guarded_retrieval",
    "details": {
      "field": "user_id"
    }
  }
}
```

Обязательные поля:
- `code`
- `message`
- `retryable`
- `source`

## Strategy parser

### Input schema

```json
{
  "user_id": "user_123",
  "raw_strategy_text": "Хочу умеренный риск и не более 20% в одной акции",
  "language": "ru",
  "existing_strategy": null
}
```

### Output schema

```json
{
  "structured_strategy": {
    "schema_version": "1.0.0",
    "profile": {
      "risk_level": "moderate"
    }
  },
  "confidence": {
    "overall": 0.84,
    "needs_clarification": false
  },
  "validation": {
    "is_valid": true,
    "errors": []
  }
}
```

### Error codes

- `STRATEGY_PARSE_FAILED`
- `STRATEGY_VALIDATION_ERROR`
- `STRATEGY_CONTRADICTION_FOUND`
- `MODEL_SCHEMA_VIOLATION`

## Guarded retrieval tool

### Input schema

```json
{
  "user_id": "user_123",
  "query_mode": "sql",
  "query": "SELECT ticker, weight_pct FROM portfolio_positions WHERE user_id = :user_id LIMIT 20",
  "parameters": {
    "user_id": "user_123"
  }
}
```

### Output schema

```json
{
  "columns": ["ticker", "weight_pct"],
  "rows": [
    ["SBER", 12.5]
  ],
  "row_count": 1,
  "truncated": false,
  "source_timestamp": "2026-03-30T12:00:00Z"
}
```

### Error codes

- `UNSAFE_QUERY`
- `QUERY_TIMEOUT`
- `UNSUPPORTED_FIELD`
- `ROW_LIMIT_EXCEEDED`
- `USER_SCOPE_MISSING`

## Visualization tool

### Input schema

```json
{
  "chart_type": "line",
  "title": "Динамика актива",
  "x_field": "date",
  "y_fields": ["close"],
  "dataset": [
    {
      "date": "2026-03-01",
      "close": 300.5
    }
  ],
  "output_format": "png"
}
```

### Output schema

```json
{
  "artifact_id": "art_123",
  "mime_type": "image/png",
  "width": 1280,
  "height": 720,
  "created_at": "2026-03-30T12:00:00Z"
}
```

### Error codes

- `INVALID_CHART_SPEC`
- `RENDER_TIMEOUT`
- `RENDER_FAILED`
- `DATASET_TOO_LARGE`

## Валидатор финального ответа

### Input schema

```json
{
  "draft_answer": "Портфель частично соответствует стратегии...",
  "evidence_bundle": {
    "metrics": {
      "top1_weight_pct": 24.0
    },
    "unsupported_criteria": ["esg_rating"]
  }
}
```

### Output schema

```json
{
  "is_valid": true,
  "violations": []
}
```

### Error codes

- `UNSUPPORTED_NUMERIC_CLAIM`
- `MISSING_DISCLOSURE`
- `UNSUPPORTED_CRITERION_USED`
- `STALE_DATA_NOT_DISCLOSED`
