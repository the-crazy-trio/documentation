# Контракт адаптера T-Invest API

## Назначение

Этот документ описывает не полный внешний API брокера, а внутренний адаптерный контракт, на который должна опираться система PoC.

Такой контракт нужен, чтобы оркестратор и retriever layer не зависели от деталей SDK/REST/gRPC клиента конкретного брокера.

## Принципы

- только read-only операции;
- все ошибки нормализуются к единому internal error format;
- все ответы содержат `source_timestamp`;
- адаптер работает от имени уже связанного пользователя и счета;
- никаких write side effects.

## Метод `GetBrokerAccounts`

### Вход

```json
{
  "user_id": "user_123",
  "broker_connection_id": "conn_123"
}
```

### Выход

```json
{
  "accounts": [
    {
      "account_id": "acc_001",
      "account_type": "broker",
      "status": "open",
      "currency": "RUB"
    }
  ],
  "source_timestamp": "2026-03-30T12:00:00Z"
}
```

## Метод `GetPortfolioSnapshot`

### Вход

```json
{
  "user_id": "user_123",
  "broker_connection_id": "conn_123",
  "account_id": "acc_001",
  "include_money": true,
  "include_expected_yield": true
}
```

### Выход

```json
{
  "account_id": "acc_001",
  "positions": [
    {
      "instrument_id": "figi_123",
      "ticker": "SBER",
      "name": "Sberbank",
      "asset_type": "share",
      "quantity": 10,
      "currency": "RUB",
      "current_price": 310.12,
      "market_value": 3101.2,
      "expected_yield": 120.5,
      "average_price": 298.07,
      "source_flags": {
        "average_price_available": true,
        "expected_yield_available": true
      }
    }
  ],
  "cash_balances": [
    {
      "currency": "RUB",
      "amount": 12000.0
    }
  ],
  "source_timestamp": "2026-03-30T12:00:00Z"
}
```

## Метод `GetOperations`

### Вход

```json
{
  "user_id": "user_123",
  "broker_connection_id": "conn_123",
  "account_id": "acc_001",
  "from": "2025-01-01T00:00:00Z",
  "to": "2026-03-30T12:00:00Z",
  "cursor": null,
  "limit": 500
}
```

### Выход

```json
{
  "operations": [
    {
      "operation_id": "op_001",
      "instrument_id": "figi_123",
      "operation_type": "buy",
      "quantity": 5,
      "price": 280.0,
      "currency": "RUB",
      "executed_at": "2025-06-01T10:00:00Z"
    }
  ],
  "next_cursor": null,
  "source_timestamp": "2026-03-30T12:00:00Z"
}
```

## Метод `GetInstrumentInfo`

### Вход

```json
{
  "instrument_id": "figi_123"
}
```

### Выход

```json
{
  "instrument_id": "figi_123",
  "ticker": "SBER",
  "isin": "RU0009029540",
  "name": "Sberbank",
  "asset_type": "share",
  "lot": 10,
  "currency": "RUB",
  "source_timestamp": "2026-03-30T12:00:00Z"
}
```

## Нормализованный формат ошибки

```json
{
  "error": {
    "code": "BROKER_TIMEOUT",
    "message": "T-Invest API did not respond within timeout budget",
    "retryable": true,
    "source": "tinvest",
    "details": {
      "account_id": "acc_001"
    }
  }
}
```

## Минимальный набор кодов ошибок

- `BROKER_AUTH_FAILED`
- `BROKER_ACCESS_DENIED`
- `BROKER_TIMEOUT`
- `BROKER_RATE_LIMITED`
- `BROKER_PARTIAL_DATA`
- `BROKER_ACCOUNT_NOT_FOUND`
- `BROKER_INSTRUMENT_NOT_FOUND`

## Что еще нужно уточнить

- точный способ авторизации и хранения токенов;
- перечень полей, гарантированно доступных в sandbox/demo контуре;
- ограничения пагинации и rate limits;
- доступность операций и cost basis в выбранном тарифе/режиме.
