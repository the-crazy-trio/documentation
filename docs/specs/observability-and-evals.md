# Спецификация observability и evals

## Метрики

### Технические
- число запросов по классам;
- доля `success / partial / failed`;
- end-to-end latency;
- latency и timeout rate по инструментам;
- число retries;
- стоимость запроса.

### Качественные
- strategy parsing accuracy;
- groundedness pass rate;
- fabrication rate;
- correctness strategy-fit-анализа;
- unsupported-criteria disclosure rate.

## Логи

### Что логируем
- correlation ids;
- request class;
- вызовы инструментов и их статусы;
- latency и retry metadata;
- cost metadata;
- artifact ids.

### Что не логируем
- секреты, токены, headers;
- raw broker credentials;
- полные raw portfolio payloads по умолчанию;
- полный raw strategy text по умолчанию.

## Traces

Минимальные обязательные spans:
- API intake;
- classification;
- memory retrieval;
- broker retrieval;
- market retrieval;
- analytics computation;
- LLM calls;
- validation;
- rendering.

## Online checks

- schema validation для LLM outputs;
- валидатор числовых утверждений в финальном ответе;
- проверка freshness/source-date disclosure;
- unsafe query rejection.

## Offline evals

Конкретные benchmark-значения и состав наборов вынесены в `docs/specs/offline-eval-benchmarks.md`.
