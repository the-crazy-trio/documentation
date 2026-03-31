# Спецификация retriever/tool layer

## Назначение

Retriever/tool layer предоставляет source-aware доступ к структурированным данным и внешним системам. Он скрывает различия между API и хранилищами от оркестратора и возвращает typed, validated payloads.

## Источники retrieval

1. Брокерский источник портфеля
- позиции;
- количество бумаг;
- текущая стоимость;
- при наличии история операций и cost basis.

2. Analytics storage
- instrument metadata;
- классификации;
- price history;
- macro indicators.

3. Memory storage
- strategy slots;
- `PortfolioSnapshot`;
- `AnalyticsSnapshot`;
- prior assessment summaries.

## Этапы retrieval

1. Выбор источника.
2. Нормализация запроса.
3. Выполнение запроса.
4. Schema validation.
5. Freshness check.
6. Coverage assessment.

## Приоритизация

PoC не требует semantic reranking. Приоритизация rule-based:
- сначала latest valid user-specific snapshot;
- затем свежий `AnalyticsSnapshot`;
- затем прямой структурированный источник;
- derived summaries используются только как вспомогательный слой.

## Ограничения

- ограничивать число строк, возвращаемых в prompt-building контур;
- при больших результатах возвращать aggregates + top-N;
- помечать truncated results;
- отклонять запросы, которые нарушают source/scope policy.

## Поведение при отсутствии данных

Если обязательное поле отсутствует:
- возвращать structured absence, а не подставлять `null` без причины;
- прикладывать причину, если она известна;
- передавать в оркестратор информацию, достаточную для частичного ответа или clarification.

## Минимальные гарантии retriever layer

- не возвращать данные другого пользователя;
- не скрывать неполноту покрытия;
- всегда прикладывать freshness metadata;
- нормализовать ошибки к единому error schema.
