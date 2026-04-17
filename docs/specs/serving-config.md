# Спецификация: Serving / Config

## Назначение

Конфигурация запуска PoC и управление runtime-параметрами.

## Runtime компоненты

- `Client Interface` gateway
- `Orchestrator` service
- `Retriever/Tool` gateway
- storage adapters

## Обязательные параметры

- `APP_ENV`
- `OPENROUTER_API_KEY`
- `TINVEST_READONLY_TOKEN`
- `DEFAULT_MODEL_INTENT`
- `DEFAULT_MODEL_SYNTHESIS`
- `REQUEST_TIMEOUT_MS`
- `TOOL_TIMEOUT_MS`
- `LLM_TIMEOUT_MS`
- `MAX_RETRIES_TRANSIENT`
- `TOKENS_INTENT_MAX`
- `TOKENS_SYNTHESIS_MAX`
- `PORTFOLIO_SNAPSHOT_MAX_AGE_MIN`
- `ANALYTICS_SNAPSHOT_MAX_AGE_MIN`
- `STRATEGY_FIT_MIN_COVERAGE`

## Feature flags

- `ENABLE_GUARDED_TEXT2SQL`
- `ENABLE_GUARDED_TEXT2API`
- `ENABLE_OFFLINE_EVAL_EXPORT`

## Секреты

- Для PoC: только env (env-only)

## Версионирование runtime

В telemetry должны фиксироваться:
- `model_id`
- `prompt_version`
- `schema_versions`
- `workflow_version`

## Model matrix (by request class)

- `strategy_parsing`: `DEFAULT_MODEL_INTENT`
- `portfolio_analysis`: `DEFAULT_MODEL_SYNTHESIS`
- `strategy_fit`: `DEFAULT_MODEL_SYNTHESIS`
- `asset_analysis`: `DEFAULT_MODEL_SYNTHESIS`
- `portfolio_qa`: `DEFAULT_MODEL_SYNTHESIS`

Правило: intent классификация всегда может использовать более дешевую модель, synthesis — более качественную

## Changelog format

Для prompt/schema/workflow используется единый changelog в формате Keep a Changelog + SemVer

Расположение:
- `docs/specs/changelog.md`

Минимальная запись:
- `date`
- `version`
- `component` (`prompt|schema|workflow|model-routing`)
- `change_type` (`added|changed|fixed|removed`)
- `summary`
- `migration_notes` (если требуется)
- `eval_impact` (baseline vs current)
