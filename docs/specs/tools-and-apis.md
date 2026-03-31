# Спецификация инструментов и API

## OpenRouter

### Назначение

- bounded LLM inference;
- intent parsing;
- strategy parsing;
- grounded synthesis;
- query planning.

### Контракт

- вход: system instructions + compact validated context;
- выход: schema-bound JSON или bounded text response;
- timeout: зависит от request class;
- ошибки: timeout, rate limit, invalid schema, unavailable model;
- side effects: отсутствуют;
- guardrails: token caps, schema validation, provider fallback, запрет на секреты в prompt/logs.

## T-Invest API

### Назначение

- read-only доступ к счетам, портфелю и операциям.

### Контракт

- вход: user-bound broker credentials или token reference;
- выход: accounts, positions, операции, money balances;
- timeout: ограниченный, с retry/backoff;
- ошибки: auth failure, timeout, rate limit, partial data;
- side effects: отсутствуют;
- guardrails: read-only scope, account ownership validation, no token logging.

Подробный адаптерный контракт описан в `docs/specs/tinvest-api-contract.md`.

## Market / macro providers

### Назначение

- обогащение анализа ценами, историей, классификациями и индикаторами.

### Контракт

- вход: ticker/FIGI/ISIN или ключ индикатора;
- выход: timestamped market/macro data;
- timeout: ограниченный;
- ошибки: stale data, unavailable symbol, source timeout;
- side effects: отсутствуют;
- guardrails: обязательные source timestamps и disclosure stale-data.

## Guarded retrieval tool

### Назначение

- безопасное выполнение запросов к данным портфеля.

### Контракт

- вход: validated query plan или generated SQL/API intent;
- выход: bounded table result + metadata;
- timeout: короткий, ограниченный;
- ошибки: unsafe query, timeout, unsupported field, row limit exceeded;
- side effects: отсутствуют;
- guardrails: allowlisted views only, mandatory user filter, no DDL/DML, row cap.

## Sandbox renderer

### Назначение

- построение PNG-графиков из подготовленных датасетов.

### Контракт

- вход: typed dataset + chart spec;
- выход: artifact metadata;
- timeout: ограниченный;
- ошибки: rendering error, timeout, invalid spec;
- side effects: создание PNG-артефакта;
- guardrails: no arbitrary code path, no network, no unrestricted filesystem access.

## Дополнение

Конкретные JSON-схемы входов, выходов и ошибок вынесены в `docs/specs/tool-schemas.md`.
