# Схема `AnalyticsSnapshot`

## Назначение

Этот документ фиксирует каноническую структуру `AnalyticsSnapshot`, который используется как основной источник фактов для:
- strategy-fit-анализа;
- portfolio Q&A;
- final answer synthesis;
- validator checks;
- visualization preparation.

Документ заполнен на основе актуальной документации T-Invest API и задает рабочую схему для PoC.

## Статус документа

- Версия схемы: `0.2.0`
- Статус: `working-draft`
- Владелец: `favead`
- Последнее обновление: `2026-03-31`

## Источники данных T-Invest API, на которые опирается `AnalyticsSnapshot`

### Обязательные broker endpoints

1. `UsersService/GetAccounts`
- нужен для определения доступных счетов и их статуса;
- используется до построения snapshot.

2. `OperationsService/GetPortfolio`
- основной источник текущего состояния портфеля по счету;
- дает рыночную стоимость, позиции, expected yield и денежные остатки;
- является базой для `portfolio_totals`, `positions`, `allocations`, `concentration`.

3. `OperationsService/GetPositions`
- вспомогательный источник по позициям и остаткам;
- может использоваться для сверки состава позиций и money balances.

4. `OperationsService/GetOperations` или `GetOperationsByCursor`
- нужен только если PoC хочет поддерживать более корректный cost basis или дополнительные performance-метрики;
- без него performance-блок может быть частично недоступен.

### Обязательные enrichment endpoints

5. `InstrumentsService/GetInstrumentBy`
- основной источник нормализованной информации по инструменту;
- нужен для `ticker`, `figi`, `instrument_uid`, `class_code`, `lot`, `currency`, `instrument_type`.

6. `MarketDataService/GetLastPrices`
- используется для актуализации последних цен, если их недостаточно в broker snapshot;
- нужен для контроля свежести market data.

7. `MarketDataService/GetCandles`
- нужен для price history и risk proxies;
- используется для графиков, period returns и простых volatility/drawdown proxies.

## Ключевой принцип

`AnalyticsSnapshot` должен хранить не сырой ответ T-Invest API, а нормализованный и проверенный аналитический срез, пригодный для повторного использования downstream-модулями без повторного обращения к LLM или брокеру.

## Требования к `AnalyticsSnapshot`

- snapshot должен быть привязан к конкретному `user_id`, `account_id` и `portfolio_snapshot_id`;
- snapshot должен содержать `schema_version` и `formula_version`;
- snapshot должен содержать `computed_at` и `source_freshness`;
- snapshot должен явно отражать `coverage` и `unknown/unmapped` части данных;
- snapshot должен быть пригоден для повторного использования без обращения к сырому prompt;
- snapshot должен отделять поля, полученные напрямую из T-Invest, от полей, полученных через enrichment или вычисление.

## Канонический объект `AnalyticsSnapshot`

```json
{
  "snapshot_id": "asnap_123",
  "schema_version": "0.2.0",
  "formula_version": "metrics-v1",
  "user_id": "user_123",
  "account_id": "acc_001",
  "portfolio_snapshot_id": "psnap_123",
  "strategy_id": "strat_123",
  "computed_at": "2026-03-31T10:00:00Z",
  "source_freshness": {
    "broker_snapshot_at": "2026-03-31T09:55:00Z",
    "market_data_at": "2026-03-31T09:50:00Z",
    "macro_data_at": "2026-03-30T00:00:00Z",
    "staleness_status": "fresh"
  },
  "portfolio_totals": {
    "total_market_value": 1000000.0,
    "cash_value": 100000.0,
    "invested_value": 900000.0,
    "expected_yield": 25000.0,
    "currency_breakdown": [
      {
        "currency": "RUB",
        "market_value": 800000.0,
        "weight_pct": 80.0
      },
      {
        "currency": "USD",
        "market_value": 200000.0,
        "weight_pct": 20.0
      }
    ]
  },
  "positions": [
    {
      "instrument_id": "figi_123",
      "instrument_uid": "uid_123",
      "position_uid": "pos_123",
      "ticker": "SBER",
      "class_code": "TQBR",
      "name": "Sberbank",
      "broker_asset_type": "share",
      "normalized_asset_class": "stocks",
      "currency": "RUB",
      "quantity": 10,
      "lot": 10,
      "current_price": 310.12,
      "market_value": 3101.2,
      "expected_yield": 120.5,
      "average_price": 298.07,
      "weight_pct": 0.31,
      "sector": "financials",
      "is_unknown_mapping": false,
      "source_flags": {
        "price_from_portfolio": true,
        "price_from_market_data": false,
        "cost_basis_available": true,
        "classification_enriched": true
      }
    }
  ],
  "allocations": {
    "by_asset_class": [],
    "by_sector": [],
    "by_currency": []
  },
  "concentration": {
    "top_1_weight_pct": 0.0,
    "top_3_weight_pct": 0.0,
    "top_5_weight_pct": 0.0,
    "max_sector_weight_pct": 0.0,
    "herfindahl_index": null
  },
  "performance": {
    "has_cost_basis": false,
    "unrealized_pl": null,
    "unrealized_pl_pct": null,
    "period_returns": []
  },
  "risk_proxies": {
    "volatility_30d_pct": null,
    "max_drawdown_1y_pct": null,
    "price_change_30d_pct": null
  },
  "coverage": {
    "covered_market_value_pct": 0.0,
    "unknown_assets_count": 0,
    "unknown_market_value_pct": 0.0,
    "unsupported_dimensions": []
  },
  "strategy_fit_inputs": {
    "constraints_checked": [],
    "constraints_unchecked": []
  },
  "artifacts": {
    "chart_ready_tables": []
  },
  "validation": {
    "is_complete": false,
    "is_partial": true,
    "warnings": [],
    "errors": []
  }
}
```

## Что приходит из T-Invest напрямую

Из `GetPortfolio` и связанных broker endpoints в snapshot можно использовать напрямую или почти напрямую:
- `account_id`
- денежные остатки
- текущие позиции
- `quantity`
- `expected_yield`
- текущую стоимость позиции
- часть ценовых полей для портфельной оценки
- валюту позиции

Из `GetInstrumentBy` можно брать:
- `figi`
- `instrument_uid`
- `ticker`
- `class_code`
- `lot`
- `name`
- `instrument_type`
- `currency`

Из `GetLastPrices` и `GetCandles` можно брать:
- последнюю цену
- историю свечей
- timestamps рыночных данных

## Что не приходит из T-Invest как гарантированное поле snapshot

Следующие поля не следует считать гарантированно доступными только из broker API:
- нормализованный `asset_class` для бизнес-аналитики;
- `sector` в нужном для PoC виде;
- `country` экспозиции в пригодном для strategy-fit-анализа виде;
- корректный `cost_basis` для всех сценариев;
- фундаментальные показатели;
- макроэкономические индикаторы.

Поэтому эти поля должны быть либо:
- рассчитаны в analytics layer,
- либо обогащены из `InstrumentsService` и дополнительных reference-источников,
- либо помечены как `unknown` / `unsupported`.

## Обязательные поля

- `snapshot_id`
- `schema_version`
- `formula_version`
- `user_id`
- `account_id`
- `portfolio_snapshot_id`
- `computed_at`
- `source_freshness`
- `portfolio_totals.total_market_value`
- `positions`
- `coverage`
- `validation`

## Нормализация позиций

Каждая позиция в snapshot должна содержать:
- брокерский идентификатор позиции, если доступен;
- инструментальный идентификатор `figi` или `instrument_uid`;
- нормализованный `normalized_asset_class`;
- `market_value`;
- `weight_pct`;
- флаги качества источника.

Если идентификатор инструмента известен, но enrichment по сектору/стране не найден, позиция не считается полностью unknown. В этом случае:
- `is_unknown_mapping=false`
- соответствующая dimension попадает в `coverage.unsupported_dimensions` или отдельный warning.

Для текущего PoC обязательным enrichment считается только `sector` и `normalized_asset_class`. `Country` не входит в обязательный аналитический контур.

## Заполненные секции snapshot

### 1. `allocations.by_asset_class`

Обязательные allocation dimensions для PoC:
- `stocks`
- `bonds`
- `etf`
- `currencies`
- `cash`
- `futures`
- `options`
- `other`
- `unknown`

Правила нормализации:
- первичный тип берется из `instrument_type` / broker type в T-Invest;
- затем маппится во внутренний `normalized_asset_class`;
- если тип не распознан или не поддерживается, используется `unknown` или `other`;
- `cash` всегда выделяется отдельно и не смешивается с `currencies`.

Пример элемента:

```json
{
  "key": "stocks",
  "market_value": 500000.0,
  "weight_pct": 50.0
}
```

### 2. `allocations.by_sector`

Источник сектора:
- не гарантируется напрямую `GetPortfolio`;
- должен получаться из внутренней mapping table на базе `instrument_uid` / `figi`, с использованием `GetInstrumentBy` для нормализации идентификаторов.

Правило при конфликте источников:
- первичной считается внутренняя mapping table системы;
- если sector неизвестен, позиция попадает в `unknown_sector`;
- sector allocation считается только по покрытой стоимости, а доля непокрытой части отражается в `coverage`.

### 3. `allocations.by_country`

Для текущего PoC секция сохраняется как резерв на будущее, но не используется в обязательной аналитике и strategy-fit-анализе.

Правило:
- `by_country` не заполняется как обязательная часть `AnalyticsSnapshot`;
- отсутствие country enrichment не должно ухудшать verdict strategy-fit-анализа;
- если поле появится позже, оно должно использоваться только после отдельного утверждения source-of-truth.

### 4. `performance.period_returns`

Поддерживаемые горизонты для PoC:
- `1d`
- `7d`
- `30d`
- `ytd`
- `1y`

Источник:
- `GetCandles`;
- либо derived calculation из price history.

Зависимость от cost basis:
- period returns по рыночной цене не требуют cost basis;
- unrealized P/L требует поля cost basis или достаточно надежной reconstruction по операциям;
- если cost basis недоступен, `unrealized_pl` и `unrealized_pl_pct` должны быть `null`.

### 5. `risk_proxies`

Допустимые risk proxies для PoC:
- `volatility_30d_pct`
- `max_drawdown_1y_pct`
- `price_change_30d_pct`

Обязательность:
- обязательной считается только `price_change_30d_pct`, если есть candles;
- `volatility_30d_pct` и `max_drawdown_1y_pct` являются optional;
- если history unavailable, risk proxies не вычисляются.

### 6. `strategy_fit_inputs`

Поля, которые можно проверять в PoC:
- доли по `asset_class`
- доли по `currency`
- доли по `sector`, если enrichment покрыт достаточно
- концентрация по позиции
- концентрация по сектору
- наличие `cash`

Поля, которые запрещено использовать без explicit support:
- ESG
- дивидендная доходность, если нет отдельного подтвержденного источника
- кредитный рейтинг
- фундаментальные мультипликаторы, если они не заведены в отдельный validated data source

## Политика частичного snapshot

Snapshot должен считаться `partial`, если выполняется хотя бы одно из условий:
- часть активов не сматчена на instrument metadata;
- отсутствуют цены по части позиций;
- отсутствуют нужные данные для performance-метрик;
- стратегия требует неподдерживаемую dimension;
- sector enrichment покрывает не весь портфель;
- операции недоступны и из-за этого нельзя корректно посчитать cost basis.

Для каждого partial случая обязательно заполняются:
- `validation.warnings`
- `coverage.unsupported_dimensions`
- `coverage.unknown_market_value_pct`
- `strategy_fit_inputs.constraints_unchecked`

## Freshness policy

На основе характера T-Invest API и текущего PoC фиксируются следующие правила:

- broker data: допустимая свежесть `<= 15 минут` для reuse без пересчета;
- market data: допустимая свежесть `<= 15 минут` для ответов по текущим ценам;
- macro data: допустимая свежесть определяется отдельным macro source и для PoC может быть `<= 30 дней` или по дате последней публикации;
- snapshot считается reusable без пересчета, если:
  - broker snapshot свежий,
  - нет запроса на новый time horizon,
  - не менялась стратегия,
  - не требуется новый chart dataset,
  - и quality flags не показывают incomplete state для нужной dimension.

## Поля snapshot, которые используются валидатором финального ответа

Валидатор финального ответа должен иметь доступ как минимум к следующим полям:
- `portfolio_totals.total_market_value`
- `allocations`
- `concentration`
- `performance`
- `risk_proxies`
- `coverage`
- `strategy_fit_inputs.constraints_checked`
- `strategy_fit_inputs.constraints_unchecked`
- `source_freshness`
- `validation.warnings`

## Хранение chart-ready tables

Для PoC допустимо хранить `chart_ready_tables` внутри snapshot только в одном из случаев:
- таблица маленькая и напрямую связана с snapshot;
- повторное построение дорогое или увеличивает latency.

Во всех остальных случаях предпочтительнее хранить отдельно:
- ссылку на artifact dataset;
- метаданные dataset;
- версию запроса на визуализацию.

Рекомендуемое правило для PoC:
- в `AnalyticsSnapshot` хранить только небольшие summary tables;
- большие series и candle datasets хранить отдельно.

## Согласование с T-Invest API ограничениями

При проектировании snapshot нужно учитывать:
- `GetPortfolio` и `GetPositions` завязаны на `accountId`;
- `GetCandles` требует явный `instrumentId`, `from`, `to`, `interval` и имеет ограничения по диапазону и `limit`;
- `GetLastPrices` принимает массив `instrumentId`;
- `GetInstrumentBy` требует корректный `idType` и при `ticker` дополнительно `classCode`.

Следствие для snapshot:
- snapshot должен хранить как минимум один устойчивый идентификатор инструмента: предпочтительно `instrument_uid`, резервно `figi`;
- для запросов history/chart нужно сохранять идентификатор в форме, совместимой с market data endpoints;
- для multi-currency портфеля требуется хранить currency breakdown отдельно от asset class breakdown.

## Зависимости

Этот документ должен быть согласован с:
- `docs/specs/strategy-schema.md`
- `docs/specs/tool-schemas.md`
- `docs/specs/portfolio-metrics-formulas.md`
- `docs/specs/tinvest-api-contract.md`

## Что еще остается зафиксировать отдельно

Хотя схема теперь заполнена, для полного перехода к реализации еще нужны:

1. точные формулы `volatility_30d_pct` и `max_drawdown_1y_pct`;
2. таблица маппинга `instrument_type -> normalized_asset_class`;
3. policy для sector enrichment и внутренней mapping table;
4. правила округления для отображаемых money/percentage fields.
