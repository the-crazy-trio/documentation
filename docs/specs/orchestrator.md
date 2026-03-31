# Спецификация оркестратора

## Назначение

Оркестратор — единственный компонент, который координирует полный пользовательский запрос end-to-end.

Он отвечает за:
- определение класса запроса;
- сбор контекста;
- выбор шага исполнения;
- применение policy/guardrails;
- запуск разрешенных LLM-задач;
- вызов retrieval и вычислительных модулей;
- валидацию результата;
- выбор между полным ответом, частичным ответом, clarification и безопасным отказом.

## Входы

- нормализованный пользовательский запрос;
- `user_id`, `chat_id`, `session_id`;
- текущий session state;
- latest strategy slots при наличии;
- latest `PortfolioSnapshot` при наличии;
- latest `AnalyticsSnapshot` при наличии;
- capability flags внешних источников;
- request metadata: correlation id, deadlines, budget class.

## Выходы

- полный grounded answer;
- частичный ответ с явным описанием ограничений;
- уточняющий вопрос;
- безопасный отказ с объяснением причин.

## Режимы исполнения

1. `portfolio_snapshot`
2. `asset_analysis`
3. `strategy_save`
4. `strategy_fit`
5. `diversification_guidance`
6. `portfolio_query`
7. `visualization`

## Машина состояний

Основной путь:

`received -> classified -> context_ready -> execution_started -> validation -> responded`

Альтернативные ветки:
- `classified -> clarification_required`
- `execution_started -> partial_ready`
- `validation -> degraded_response`
- `classified -> rejected`

## Разрешенные роли LLM

LLM разрешено использовать только для:
- ambiguous intent classification;
- strategy parsing;
- query planning;
- final answer synthesis по validated evidence.

LLM запрещено использовать для:
- unrestricted tool selection;
- прямого SQL execution;
- вычисления чисел без deterministic evidence;
- принятия решений о правах доступа.

## Stop conditions

Оркестратор прекращает выполнение, если:
- запрос вне поддерживаемого scope;
- пользователь невалиден или не авторизован;
- query/tool call не проходит policy validation;
- нет достаточных данных для безопасного ответа и partial path не подходит;
- исчерпан timeout или retry budget.

## Retry policy

- внешние transient API ошибки: ограниченный retry с exponential backoff;
- transient ошибки LLM provider: один повтор, затем fallback model или degraded response;
- validation failure финального текста: максимум одна повторная генерация по тому же evidence bundle.

## Fallback policy

- сначала использовать последний приемлемый snapshot, если он достаточно свежий;
- иначе вернуть partial answer с явным перечислением пробелов в данных;
- никогда не заполнять пробелы догадками.

## Ключевые внутренние интерфейсы

Оркестратор должен оперировать нормализованными внутренними сущностями:
- `RequestEnvelope`
- `ExecutionPlan`
- `EvidenceBundle`
- `ValidationResult`
- `UserFacingResponse`

Эти типы должны быть зафиксированы в коде и тестах до начала бизнес-логики.
