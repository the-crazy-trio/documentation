# Спецификация формул портфельных метрик

## Назначение

Этот документ фиксирует формулы и правила вычисления портфельных метрик, используемых в PoC.

Он должен использоваться как единый источник истины для:
- `AnalyticsSnapshot`;
- strategy-fit-анализа;
- portfolio summary;
- asset analysis;
- проверок валидатора финального ответа;
- offline evaluation.

## Статус документа

- Версия: `0.2.0`
- Статус: `working-draft`
- Владелец: `favead`
- Последнее обновление: `2026-03-31`

## Общие правила

- все метрики вычисляются детерминированно;
- каждая метрика должна иметь явные `inputs`, `preconditions` и `failure_mode`;
- если входные данные неполны, метрика возвращает `not_available`, а не приближенное значение без disclosure;
- все money values хранятся в валюте расчета snapshot;
- все проценты в snapshot хранятся в шкале `0..100`;
- округление в storage и округление в UI должны различаться: snapshot хранит точность выше, UI округляет отдельно.

## Политика округления

- денежные значения в snapshot: до `4` знаков после запятой;
- проценты в snapshot: до `4` знаков после запятой;
- UI-отображение денег: до `2` знаков;
- UI-отображение процентов: до `1-2` знаков;
- intermediate calculations не округляются до финального шага.

## Базовые обозначения

- `total_market_value`: общая рыночная стоимость портфеля, включая кэш;
- `invested_value`: рыночная стоимость инвестиционных позиций без кэша;
- `cash_value`: денежные остатки;
- `market_value_i`: рыночная стоимость позиции `i`;
- `cost_basis_value_i`: стоимость входа по позиции `i`, если доступна;
- `weight_i`: доля позиции `i` в портфеле в шкале `0..1`;
- `weight_pct_i`: доля позиции `i` в портфеле в шкале `0..100`.

## Метрики аллокации

### WeightPct

- Назначение: доля позиции или агрегата в общей стоимости портфеля.
- Формула: `market_value / total_market_value * 100`
- Входы: `market_value`, `total_market_value`
- Preconditions: `total_market_value > 0`
- Правила округления: до `4` знаков в snapshot
- Что делать при отсутствии данных: `return not_available`
- Где используется: allocations, concentration, strategy-fit-анализ

### InvestedWeightPct

- Назначение: доля позиции или агрегата в инвестированной части портфеля без учета кэша.
- Формула: `market_value / invested_value * 100`
- Входы: `market_value`, `invested_value`
- Preconditions: `invested_value > 0`
- Правила округления: до `4` знаков в snapshot
- Что делать при отсутствии данных: `return not_available`
- Где используется: вспомогательная аналитика, optional UI breakdown

### CashWeightPct

- Назначение: доля денежных средств в портфеле.
- Формула: `cash_value / total_market_value * 100`
- Входы: `cash_value`, `total_market_value`
- Preconditions: `total_market_value > 0`
- Правила округления: до `4` знаков
- Что делать при отсутствии данных: `return not_available`
- Где используется: strategy-fit-анализ, portfolio summary

## Метрики концентрации

### TopNWeightPct

- Назначение: доля суммы крупнейших `N` позиций.
- Формула: `sum(top_n_position_market_values) / total_market_value * 100`
- Входы: список позиций, `N`, `total_market_value`
- Preconditions: позиции отсортированы по `market_value desc`
- Правила округления: до `4` знаков
- Что делать при отсутствии данных: `return not_available`
- Где используется: concentration analysis, strategy-fit-анализ

### MaxPositionWeightPct

- Назначение: доля крупнейшей позиции.
- Формула: `max(market_value_i) / total_market_value * 100`
- Входы: список позиций, `total_market_value`
- Preconditions: есть хотя бы одна позиция
- Правила округления: до `4` знаков
- Что делать при отсутствии данных: `return not_available`
- Где используется: strategy-fit-анализ, portfolio summary

### MaxSectorWeightPct

- Назначение: максимальная доля сектора.
- Формула: `max(sector_market_value / total_market_value * 100)`
- Входы: sector aggregation, `total_market_value`
- Preconditions: sector mapping coverage достаточна для данного use case
- Правила округления: до `4` знаков
- Что делать при отсутствии данных: `return not_available` и записать `sector` в unchecked constraints
- Где используется: diversification guidance, strategy-fit-анализ

### HerfindahlIndex

- Назначение: прокси концентрации портфеля.
- Формула: `sum((weight_i)^2)`
- Входы: `weight_i` для всех покрытых позиций
- Preconditions: веса нормализованы в шкалу `0..1`
- Правила округления: до `6` знаков
- Что делать при отсутствии данных: `return not_available`
- Где используется: optional concentration proxy

## Метрики доходности

### UnrealizedPL

- Назначение: нереализованная прибыль/убыток по позиции или портфелю.
- Формула: `market_value - cost_basis_value`
- Входы: `market_value`, `cost_basis_value`
- Preconditions: cost basis available
- Правила округления: до `4` знаков
- Что делать при отсутствии данных: `return not_available`
- Где используется: portfolio summary, asset analysis

### UnrealizedPLPct

- Назначение: нереализованная доходность в процентах.
- Формула: `(market_value - cost_basis_value) / cost_basis_value * 100`
- Входы: `market_value`, `cost_basis_value`
- Preconditions: `cost_basis_value > 0`
- Правила округления: до `4` знаков
- Что делать при отсутствии данных: `return not_available`
- Где используется: portfolio summary, asset analysis

### PeriodReturnPct

- Назначение: изменение цены инструмента или агрегата за фиксированный период.
- Формула: `(end_price - start_price) / start_price * 100`
- Входы: `start_price`, `end_price`
- Preconditions: обе цены доступны, `start_price > 0`
- Правила округления: до `4` знаков
- Что делать при отсутствии данных: `return not_available`
- Где используется: asset analysis, summary, charts

Поддерживаемые горизонты PoC:
- `1d`
- `7d`
- `30d`
- `ytd`
- `1y`

Источник:
- `MarketDataService/GetCandles`
- либо derived aggregation по свечам

## Risk proxies

### Volatility30dPct

- Назначение: простая прокси-оценка волатильности для PoC.
- Формула:
  `stddev(daily_returns_30d) * sqrt(252) * 100`
- Входы:
  - массив дневных доходностей за последние 30 торговых дней
- Preconditions:
  - есть не менее `20` дневных свечей;
  - свечи упорядочены по времени;
  - отсутствуют критичные пропуски по ряду.
- Правила округления: до `4` знаков
- Что делать при отсутствии данных: `return not_available`
- Где используется: asset analysis, optional explanation layer

### MaxDrawdown1yPct

- Назначение: прокси максимальной просадки по историческому ряду.
- Формула:
  `min((price_t - rolling_peak_t) / rolling_peak_t) * 100`

Где:
- `rolling_peak_t = max(price_0 ... price_t)`

- Входы:
  - дневной price series за `1y`
- Preconditions:
  - есть достаточный history window;
  - используется закрытие дневных свечей.
- Правила округления: до `4` знаков
- Что делать при отсутствии данных: `return not_available`
- Где используется: optional explanation layer

### PriceChange30dPct

- Назначение: простая и безопасная proxy-метрика изменения цены за 30 дней.
- Формула:
  `(close_30d_end - close_30d_start) / close_30d_start * 100`
- Входы:
  - start/end close prices по дневным свечам
- Preconditions:
  - доступны обе цены;
  - `close_30d_start > 0`
- Правила округления: до `4` знаков
- Что делать при отсутствии данных: `return not_available`
- Где используется: explanation layer для strategy-fit-анализа, asset analysis

## Метрики покрытия и качества данных

### CoveredMarketValuePct

- Назначение: доля стоимости портфеля, по которой доступны необходимые для текущего use case измерения.
- Формула:
  `covered_market_value / total_market_value * 100`
- Входы:
  - `covered_market_value`
  - `total_market_value`
- Preconditions:
  - `total_market_value > 0`
- Правила округления: до `4` знаков
- Что делать при отсутствии данных: `return not_available`
- Где используется: disclosures, confidence для strategy-fit-анализа, валидатор финального ответа

### UnknownMarketValuePct

- Назначение: доля стоимости портфеля, которая не была сматчена на нужные dimensions.
- Формула:
  `unknown_market_value / total_market_value * 100`
- Входы:
  - `unknown_market_value`
  - `total_market_value`
- Preconditions:
  - `total_market_value > 0`
- Правила округления: до `4` знаков
- Что делать при отсутствии данных: `return not_available`
- Где используется: disclosures, fallback logic

## Правила strategy-fit scoring

### Базовые принципы

- verdict не должен зависеть от LLM напрямую;
- LLM может только сформулировать объяснение уже вычисленного verdict;
- unchecked constraints не могут интерпретироваться как satisfied;
- unsupported criteria не участвуют в score и должны явно раскрываться пользователю.

### Constraint evaluation

Каждое правило стратегии должно оцениваться как одно из:
- `satisfied`
- `violated`
- `unchecked`
- `unsupported`

### Verdict mapping

Рекомендуемая логика для PoC:

- `fit`
  - нет `violated` high-priority constraints;
  - нет критичных `unchecked` constraints;
  - `covered_market_value_pct >= 85`

- `partial_fit`
  - есть ограниченное число `violated` non-critical constraints;
  - или есть значимая доля `unchecked` constraints;
  - или покрытие недостаточно для уверенного `fit`, но достаточно для полезного вывода

- `not_fit`
  - нарушен хотя бы один high-priority concentration/allocation constraint;
  - или нарушено несколько medium/high constraints;
  - при этом покрытие достаточно, чтобы такой вывод был обоснован

Для текущего PoC verdict не должен зависеть от `country`-критериев.

### Coverage gating

- если `covered_market_value_pct < 60`, система не должна выдавать уверенный `fit` или `not_fit` без явного disclosure;
- при `covered_market_value_pct < 60` предпочтителен `partial_fit` или safe clarification;
- если ключевая dimension unchecked, она не должна использоваться как reason в verdict.

### Reason ranking

Причины ранжируются по приоритету:
1. high-priority violated constraints
2. high-priority satisfied constraints
3. medium-priority violated constraints
4. coverage/disclosure warnings

В финальный ответ обычно попадают `2-3` причины с наибольшей пользовательской значимостью.

## Failure modes

- `total_market_value = 0`: большинство allocation/concentration метрик недоступно;
- нет candles: history-based returns и risk proxies недоступны;
- нет cost basis / операций: P/L блок недоступен;
- sector enrichment неполон: соответствующие strategy checks переводятся в `unchecked`.

## Таблица открытых решений

| Вопрос | Решение по умолчанию для PoC |
|---|---|
| Округление | Хранить до 4 знаков, показывать 1-2 |
| Горизонты доходности | `1d`, `7d`, `30d`, `ytd`, `1y` |
| Volatility proxy | Годовая волатильность по 30 дням |
| Drawdown proxy | Max drawdown по `1y`, optional |
| Coverage penalty | Не давать уверенный verdict при покрытии < 60% |

## Связанные документы

- `docs/specs/analytics-snapshot-schema.md`
- `docs/specs/strategy-schema.md`
- `docs/specs/offline-eval-benchmarks.md`
- `docs/specs/instrument-mapping-policy.md`
