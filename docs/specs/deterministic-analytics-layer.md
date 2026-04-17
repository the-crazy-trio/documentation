# Спецификация: Deterministic Analytics Layer

## Назначение

Рассчитывает воспроизводимые портфельные метрики и strategy-fit опираясь на экономические показатели

## Вход

- `PortfolioSnapshot`
- Данные показателей
- `StructuredStrategy` (для strategy-fit)

## Выход

- `AnalyticsSnapshot`
- `metrics`
- `strategy_fit.verdict`
- `unsupported_criteria[]`
- `coverage`, `freshness`, `partial`

## Каркас анализа актива (top-down + bottom-up)

Последовательность обязательна:
1. макрорежим (leading/coincident/lagging сигналы)
2. отраслевые чувствительности (ставка, инфляция, FX, сырье, регулирование, спрос)
3. качество и оценка компании/инструмента
4. портфельный strategy-fit

## Метрики PoC

- `weight_i = position_value_i / portfolio_value`
- `top_n_concentration = sum(top_n_weights)`
- `hhi = sum(weight_i^2)`
- `class_exposure`
- `unknown_share`

## Источники факторных сигналов

- макро: ключевая ставка, кривая ОФЗ, инфляция/ожидания, M2, промпроизводство, рынок труда
- сектор/компания: маржа, ROIC/ROE, leverage, coverage, стабильность прибыли
- события: отчетность, дивиденды, корпоративные события, регуляторные изменения
- техника/сентимент: только как overlay для тайминга и риска, не как фундаментальное ядро

## Business rules (hard)

- Не экстраполировать разовые дивиденды как устойчивую платежеспособность
- Не считать нарратив достаточным драйвером без подтверждения выручкой/маржей/ROIC
- Дешевые мультипликаторы сами по себе не сигнал покупки для капиталоемких историй
- Если у клиента нет доступа к рынку/инструменту, позиция не может быть основным фактором вывода
- Cross-country сравнение мультипликаторов без risk/currency/regime корректировок запрещено

## Threshold policy

- Пороги strategy-fit и coverage берутся из взаимодействия с пользователем. По умолчанию из конфига
- Пороговые значения не хардкодятся в аналитическом коде
- При `coverage` ниже порога verdict возвращается как `partial_fit|unknown` с явной причиной

## Минимальная структура `AnalyticsSnapshot`

```json
{
  "schema_version": "1.0.0",
  "snapshot_id": "an_...",
  "user_id": "u_...",
  "account_id": "acc_...",
  "as_of_ts": "...",
  "metrics": {},
  "strategy_fit": {},
  "freshness": {},
  "partial": false
}
```
