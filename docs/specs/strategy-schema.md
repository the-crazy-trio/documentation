# Структурированная схема стратегии

## Назначение

Эта схема определяет каноническое представление стратегии пользователя после парсинга свободного текста.

Все downstream-модули должны опираться именно на нее, а не на сырой текст.

## Объект `StructuredStrategy`

```json
{
  "schema_version": "1.0.0",
  "strategy_id": "strat_123",
  "user_id": "user_123",
  "source": {
    "raw_text": "Хочу умеренный риск, горизонт 5 лет, не больше 20% в одной акции",
    "language": "ru",
    "captured_at": "2026-03-30T12:00:00Z"
  },
  "profile": {
    "risk_level": "moderate",
    "time_horizon_months": 60,
    "liquidity_need": "medium"
  },
  "target_allocations": [
    {
      "dimension": "asset_class",
      "key": "stocks",
      "target_min_pct": 40,
      "target_max_pct": 70,
      "priority": "high"
    }
  ],
  "constraints": {
    "max_single_position_pct": 20,
    "max_sector_pct": 35,
    "max_country_pct": null,
    "allowed_currencies": ["RUB", "USD"],
    "forbidden_assets": [],
    "forbidden_sectors": []
  },
  "preferences": {
    "preferred_sectors": ["energy"],
    "avoided_sectors": ["high_growth_tech"],
    "income_focus": false,
    "capital_growth_focus": true
  },
  "unsupported_criteria": [
    {
      "name": "esg_rating",
      "reason": "not_supported_in_poc"
    }
  ],
  "assumptions": [
    "currency diversification interpreted as exposure to at least 2 currencies"
  ],
  "confidence": {
    "overall": 0.82,
    "needs_clarification": false,
    "low_confidence_fields": []
  },
  "validation": {
    "is_valid": true,
    "errors": []
  }
}
```

## Обязательные поля

- `schema_version`
- `strategy_id`
- `user_id`
- `source.captured_at`
- `profile.risk_level`
- `validation.is_valid`

## Нормализованные значения

### `risk_level`
- `conservative`
- `moderate`
- `aggressive`
- `unspecified`

### `liquidity_need`
- `low`
- `medium`
- `high`
- `unspecified`

### `priority`
- `high`
- `medium`
- `low`

## Правила валидации

- `target_min_pct <= target_max_pct`
- все проценты лежат в диапазоне `0..100`
- `time_horizon_months > 0`, если указан
- противоречащие ограничения не допускаются без явной записи в `validation.errors`
- unsupported criteria не могут участвовать в финальном strategy-fit verdict

## Минимальный набор слотов для PoC

Даже если стратегия неполная, parser должен стараться извлечь:
- risk level;
- horizon;
- ограничения по концентрации;
- предпочтения/запреты по секторам или классам активов;
- currency preferences;
- unsupported criteria.

## Поведение при низкой уверенности

Если `confidence.needs_clarification=true`, оркестратор может:
- задать один уточняющий вопрос;
- сохранить стратегию с assumptions;
- или отклонить сохранение, если противоречий слишком много.
