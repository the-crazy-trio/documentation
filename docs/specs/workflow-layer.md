# Спецификация: Workflow Layer

## Назначение

Набор детерминированных workflow с фиксированными шагами и контрактами входа/выхода.

## Workflow list

1. `Strategy Parsing Workflow`
- извлекает strategy slots из текста;
- нормализует в `StructuredStrategy`;
- при нехватке данных задает 1 уточняющий вопрос.

2. `Portfolio Analysis Workflow`
- получает/обновляет `PortfolioSnapshot`;
- запускает расчет аналитики;
- возвращает summary + coverage.

3. `Strategy-Fit Workflow`
- применяет rules к `AnalyticsSnapshot` + `StructuredStrategy`;
- возвращает verdict + unsupported criteria.

4. `Asset Analysis Workflow`
- анализирует отдельный актив на доступных данных;
- возвращает выводы с disclosure ограничений покрытия.

5. `Portfolio Q&A Workflow`
- отвечает на вопросы пользователя через guarded retrieval/query path.

## Must-have структура `StructuredStrategy`

```json
{
  "client_id": "uuid",
  "base_currency": "RUB",
  "tax_residency": "RU",
  "jurisdiction_constraints": {
    "allowed_markets": ["MOEX", "SPB", "HKEX_via_access_if_any"],
    "forbidden_markets": [],
    "sanctions_or_broker_access_notes": ""
  },
  "knowledge_experience": {
    "experience_level": "none|basic|intermediate|advanced",
    "instruments_used": ["stocks", "bonds", "etf", "fx", "futures"],
    "understands_leverage": false,
    "understands_duration_risk": true
  },
  "financial_profile": {
    "monthly_income": 0,
    "monthly_expenses": 0,
    "liquid_reserve_months": 6,
    "emergency_fund_ready": true,
    "debt_load_level": "low|medium|high"
  },
  "goals": [
    {
      "goal_id": "retirement_2045",
      "goal_name": "Пенсия",
      "priority": 1,
      "target_amount": 0,
      "target_date": "2045-12-31",
      "horizon_months": 240,
      "contribution_plan": {
        "initial_amount": 0,
        "monthly_contribution": 0
      },
      "required_liquidity_windows": [],
      "target_return_nominal": null,
      "target_return_real": null,
      "max_drawdown_pct": 25,
      "acceptable_volatility_pct": 18,
      "loss_tolerance_rule": "готов терпеть просадку до 25% без продажи",
      "income_need": false,
      "dividend_cash_need": false
    }
  ],
  "risk_preferences": {
    "risk_tolerance_self_assessed": "conservative|balanced|growth|aggressive",
    "risk_capacity_model_score": 0.0,
    "max_single_name_weight": 0.1,
    "max_sector_weight": 0.25,
    "max_country_weight": 1.0,
    "max_fx_unhedged_weight": 0.3,
    "illiquid_assets_allowed": false,
    "leverage_allowed": false
  },
  "investable_universe": {
    "allowed_instruments": ["stocks", "bonds", "money_market_funds"],
    "disallowed_instruments": ["options", "structured_products"],
    "preferred_sectors": [],
    "excluded_sectors": [],
    "dividend_preference": "none|balanced|income",
    "esg_or_personal_restrictions": []
  },
  "portfolio_policy": {
    "benchmark": "custom",
    "target_asset_allocation": {
      "equities": 0.5,
      "bonds": 0.35,
      "cash": 0.15
    },
    "rebalance_policy": {
      "frequency": "quarterly",
      "drift_threshold_pct": 5
    },
    "buy_rules": {
      "min_expected_return_spread_vs_cash_pct": 4,
      "required_thesis_confidence": 0.65
    },
    "sell_rules": {
      "thesis_broken": true,
      "better_opportunity_threshold": 0.15,
      "max_position_review_days": 90
    }
  },
  "explainability_preferences": {
    "wants_short_reports": false,
    "wants_scenarios": true,
    "wants_probabilities": true
  }
}
```

## Общие правила

- Никаких свободных agent-loop без stop condition.
- Все числовые выводы должны опираться на evidence.
- Неподдержанные критерии не маскируются, а маркируются `unsupported`.
- Недоступный для клиента рынок/инструмент не используется как core basis для вывода.

## Что требует уточнения

- Версия схемы и правила backward compatibility при эволюции `StructuredStrategy`.
