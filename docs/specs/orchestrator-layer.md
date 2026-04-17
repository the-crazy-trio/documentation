# Спецификация: Orchestrator Layer

## Назначение

Управляет полным жизненным циклом запроса: классификация, запуск workflow, валидация результата, сохранение состояния.

## Шаги

1. Intake и базовая валидация
2. Intent/request class classification (LLM)
3. Загрузка минимального контекста
4. Запуск одного primary workflow
5. Validation gate (evidence, freshness, user-scope)
6. Response synthesis
7. Persist

## Request classes

`request_class` соответствует workflow 1:1:
- `strategy_parsing` -> `Strategy Parsing Workflow`
- `portfolio_analysis` -> `Portfolio Analysis Workflow`
- `strategy_fit` -> `Strategy-Fit Workflow`
- `asset_analysis` -> `Asset Analysis Workflow`
- `portfolio_qa` -> `Portfolio Q&A Workflow`

## Правила переходов

- Один запрос -> один primary workflow
- Между workflow переход только через оркестратор
- Максимум один уточняющий вопрос при неоднозначности

## Stop conditions

- Финальный ответ прошел validation gate
- Достигнут timeout budget
- Критическая ошибка без fallback

## Retry/Fallback

- Retry для таймаутов API, request limit и т.д.
- Fallback на валидный snapshot
- При невозможности fallback - ответ с `partial|unavailable`

## Runtime limits

- Лимиты токенов, timeout и retry берутся из runtime-конфига
- Для stale-данных применяется max-age policy из конфига
- Если обновить данные нельзя и max-age превышен, оркестратор обязан вернуть явный ответ с ошибкой пользователю
