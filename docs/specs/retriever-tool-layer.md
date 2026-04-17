# Спецификация: Retriever / Tool Layer

## Назначение

Единый guarded слой доступа к данным и внешним API.

## Источники

1. T-Invest API (read-only)
2. Макро и микро провайдеры
3. Memory storage
4. Analytics storage

Для контрактов T-Invest используется официальная документация:
- `https://developer.tbank.ru/invest/api`

## Обязательные контракты тулов

- `portfolio_collect`
- `portfolio_show`
- `strategy_save`
- `strategy_fit`

Общий формат ответа:

```json
{
  "status": "ok|partial|error",
  "data": {},
  "errors": [],
  "meta": {
    "latency_ms": 0,
    "source_ts": "...",
    "coverage": 0.0
  }
}
```

## Ограничения

- Обязательный `user_id`
- Allowlist действий/маршрутов
- Ограничения на SQL запросы и API вызовы

## Нормализованные ошибки

- `VALIDATION_ERROR`
- `UNAUTHORIZED_SCOPE`
- `TIMEOUT`
- `UPSTREAM_UNAVAILABLE`
- `RATE_LIMITED`
- `MAPPING_NOT_FOUND`

## T-Invest error mapping table

Базовая нормализация upstream ошибок (по официальной API-документации и gRPC/transport статусам):

| Upstream category | Internal code | Retry | Комментарий |
|---|---|---|---|
| invalid argument / bad request | `VALIDATION_ERROR` | no | Некорректные параметры запроса |
| unauthenticated / invalid token | `UNAUTHORIZED_SCOPE` | no | Ошибка токена или auth контекста |
| permission denied | `UNAUTHORIZED_SCOPE` | no | Нет прав на ресурс/операцию |
| account not found | `UPSTREAM_UNAVAILABLE` | no | Для клиента возвращается как unavailable с пояснением |
| not found (resource) | `UPSTREAM_UNAVAILABLE` | no | Ресурс отсутствует/недоступен |
| rate limit / too many requests | `RATE_LIMITED` | yes | Retry с backoff по конфигу |
| deadline exceeded / timeout | `TIMEOUT` | yes | Retry с backoff по конфигу |
| unavailable / upstream down | `UPSTREAM_UNAVAILABLE` | yes | Retry с backoff по конфигу |
| internal / unknown upstream | `UPSTREAM_UNAVAILABLE` | yes | Ограниченный retry, затем fallback |

## T-Invest adapter policy

- Только read-only операции (на уровне API ключа)
- Обязательная проверка `user_id` -> `account_id`
- Нормализация ответа в `PortfolioSnapshot`
- Если данные не обновляются и превышен max-age, возврат только `partial|unavailable`

## Instrument mapping

- Ключ: `instrument_uid` (fallback: `figi`)
- Неизвестный инструмент маркируется `unmapped`
