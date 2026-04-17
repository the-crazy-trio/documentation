# Спецификация: Observability / Evals Layer

## Назначение

Обеспечивает контроль технического здоровья и качества ответов.

## Метрики

Технические:
- `request_latency_ms` (p50/p95)
- `tool_latency_ms`
- `llm_tokens_in/out`
- `retry_count`
- `error_rate`

Качественные:
- `groundedness_pass_rate`
- `fabrication_rate`
- `strategy_fit_correctness`
- `unsupported_disclosure_rate`
- `partial_response_rate`

## Логи и трейсы

- Единый `correlation_id` на весь request path
- Lifecycle события: intake -> workflow -> validation -> response
- Tool calls без секретов

## Offline eval

- Наборы: strategy parsing, strategy-fit, portfolio Q&A, fallback
- Запуск перед изменениями prompt/schema/workflow

Дополнительно для `asset_analysis`:
- Проверка корректной декомпозиции top-down + bottom-up (macro -> sector -> issuer -> portfolio-fit);
- Проверка соблюдения бизнес правил (разовые дивиденды, narrative-only, market access)

## Retention policy (logs/traces)

- `dev`:
  - Application logs: 7 дней
  - Traces: 3 дня
  - Eval artifacts: 30 дней
- `staging`:
  - Application logs: 14 дней
  - Traces: 7 дней
  - Eval artifacts: 60 дней
- `prod` (=PoC):
  - Application logs: 30 дней
  - Traces: 14 дней
  - Eval artifacts: 90 дней

Общие правила:
- секреты, токены и PII не логируются
- удаление по TTL автоматическое через настройки БД
