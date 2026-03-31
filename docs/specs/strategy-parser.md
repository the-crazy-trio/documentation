# Спецификация Strategy Parser

## Назначение

`Strategy Parser` преобразует свободный текст стратегии пользователя в каноническую структуру `StructuredStrategy`.

Компонент нужен для того, чтобы все downstream-модули работали не с сырым текстом, а с валидированными слотами, пригодными для:
- strategy-fit-анализа;
- сохранения пользовательской стратегии;
- context assembly;
- disclosure unsupported criteria;
- повторного использования между сессиями.

## Входы

- `user_id`
- `raw_strategy_text`
- `language`
- `existing_strategy`, если пользователь обновляет уже сохраненную стратегию

Точный input schema описан в `docs/specs/tool-schemas.md`.

## Выходы

- объект `StructuredStrategy`
- confidence metadata
- validation result

Точная структура `StructuredStrategy` описана в `docs/specs/strategy-schema.md`.

## Внутренний pipeline

1. Pre-parse normalization
- очистка служебного шума;
- нормализация языка и очевидных числовых формулировок;
- ограничение слишком длинного входа.

2. LLM parsing
- LLM получает bounded prompt и schema-oriented instructions;
- LLM возвращает JSON, совместимый с `StructuredStrategy`.

3. Schema validation
- проверка обязательных полей;
- проверка диапазонов процентов;
- проверка непротиворечивости ограничений.

4. Post-processing
- нормализация enum values;
- выделение assumptions;
- выделение unsupported criteria;
- вычисление confidence flags.

5. Decision
- сохранить стратегию;
- запросить clarification;
- или вернуть validation error.

## Разрешенные роли LLM

LLM разрешено:
- извлекать risk level, horizon, ограничения и предпочтения;
- преобразовывать неструктурированный текст в JSON-структуру;
- выделять ambiguous / unsupported criteria.

LLM запрещено:
- интерпретировать unsupported criteria как поддерживаемые;
- генерировать портфельные рекомендации;
- подменять отсутствующие ограничения выдуманными фактами;
- менять schema version или contract fields.

## Clarification policy

Parser может инициировать clarification только если:
- есть противоречащие ограничения;
- отсутствует критически важный смысловой слот;
- confidence по ключевым полям низкий.

Для PoC предпочтительно не более одного уточняющего вопроса на один parse flow.

## Failure modes

- `MODEL_SCHEMA_VIOLATION`
- `STRATEGY_PARSE_FAILED`
- `STRATEGY_VALIDATION_ERROR`
- `STRATEGY_CONTRADICTION_FOUND`

При ошибке parser должен возвращать нормализованный error schema из `docs/specs/tool-schemas.md`.

## Retry policy

- допускается одна повторная генерация при transient model/provider failure;
- допускается одна повторная генерация при schema violation, если prompt/evidence не менялись;
- бесконечные циклы повтора запрещены.

## Persist policy

- сырой текст стратегии можно хранить отдельно;
- каноническим представлением считается только `StructuredStrategy`;
- вместе со стратегией должны сохраняться `schema_version`, parser metadata и timestamps.

## Метрики качества

- strategy parsing accuracy;
- unsupported criteria recall;
- false clarification rate;
- schema validation failure rate.

Эти метрики должны быть согласованы с `docs/specs/offline-eval-benchmarks.md`.

## Зависимости

- `docs/specs/strategy-schema.md`
- `docs/specs/tool-schemas.md`
- `docs/specs/prompt-schema-versioning-policy.md`
- `docs/specs/offline-eval-benchmarks.md`
