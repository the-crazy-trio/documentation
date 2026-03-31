# Политика версионирования prompt и schema

## Назначение

Этот документ описывает, как в системе версионируются:
- prompt templates;
- JSON schemas для LLM outputs;
- evaluator rules;
- validation rules;
- связки `model + prompt + schema`.

Цель политики — сделать изменения воспроизводимыми и безопасными для quality control.

## Статус документа

- Версия: `0.1.0`
- Статус: `working-draft`
- Владелец: `favead`
- Последнее обновление: `2026-03-31`

## Что должно иметь версию

Версионировать обязательно:
- prompts для intent classification;
- prompts для strategy parsing;
- prompts для final answer synthesis;
- schemas для strategy parser output;
- schemas для tool contracts, если они участвуют в LLM interaction;
- validator rules;
- offline benchmark definitions.

## Канонический идентификатор версии

Рекомендуемый формат:

```text
<component>@<semantic-version>
```

Примеры:
- `strategy-parser-prompt@1.2.0`
- `strategy-schema@1.0.0`
- `final-answer-validator@0.3.1`
- `groundedness-eval@0.2.0`

## Что записывать в runtime metadata

Каждый LLM-запрос должен быть связан с:
- `model_id`
- `provider`
- `prompt_version`
- `schema_version`
- `validator_version`
- `request_class`

Пример:

```json
{
  "provider": "openrouter",
  "model_id": "openai/gpt-4o-mini",
  "prompt_version": "strategy-parser-prompt@1.2.0",
  "schema_version": "strategy-schema@1.0.0",
  "validator_version": "strategy-validator@0.4.0",
  "request_class": "strategy_save"
}
```

## Правила semantic versioning

### Major

Повышается, если:
- schema несовместима назад;
- prompt меняет смысл output contract;
- validator меняет pass/fail semantics.

### Minor

Повышается, если:
- добавлены новые optional fields;
- prompt расширен без breaking changes;
- evaluator получил новые non-breaking checks.

### Patch

Повышается, если:
- исправлены wording issues;
- уточнены инструкции без изменения contract semantics;
- исправлены ошибки в validation/eval logic без изменения интерфейса.

## Связка model + prompt + schema

Изменение любой части из тройки:
- `model`
- `prompt`
- `schema`

должно считаться новой конфигурацией качества и требовать повторной проверки на offline benchmarks.

## Change policy

При изменении prompt или schema нужно:
1. увеличить версию;
2. обновить связанные references в конфиге;
3. прогнать offline benchmarks;
4. сохранить результаты сравнения с предыдущей версией;
5. обновить changelog или release note для этой связки.

## Минимальный changelog entry

```md
## strategy-parser-prompt@1.2.0

- Уточнен extraction currency constraints
- Снижено число false clarification cases
- Совместим со `strategy-schema@1.0.0`
```

## Compatibility rules

- prompt не должен ссылаться на schema fields, которых нет в объявленной версии schema;
- validator не должен требовать поля, которых нет в объявленной schema version;
- frontend/backend contracts должны зависеть только от стабилизированных schema versions.

## Runtime fallback policy

Если в runtime обнаружено несоответствие версий:
- запрос не должен silently проходить;
- система должна вернуть internal error или fallback path;
- mismatch должен логироваться как configuration error.

Рекомендуемые error codes:
- `PROMPT_SCHEMA_VERSION_MISMATCH`
- `VALIDATOR_SCHEMA_VERSION_MISMATCH`
- `UNSUPPORTED_PROMPT_VERSION`
- `UNSUPPORTED_SCHEMA_VERSION`

## Что хранить в snapshot и history

В следующих сущностях желательно сохранять версии:
- `StructuredStrategy`
  - `schema_version`
  - `parser_prompt_version`
- `AnalyticsSnapshot`
  - `formula_version`
- `UserFacingResponse`
  - `synthesis_prompt_version`
  - `validator_version`

## Когда обязателен rerun offline evals

Offline evals обязательны при:
- смене модели;
- изменении prompt для strategy parser;
- изменении final answer synthesis prompt;
- изменении strategy schema;
- изменении groundedness validator;
- изменении logic verdict mapping.

## Рекомендуемая структура хранения в репозитории

```text
docs/specs/
prompts/
schemas/
evals/
```

Минимально в коде или конфиге должна быть одна таблица соответствия:

```json
{
  "strategy_save": {
    "model": "openai/gpt-4o-mini",
    "prompt_version": "strategy-parser-prompt@1.2.0",
    "schema_version": "strategy-schema@1.0.0",
    "validator_version": "strategy-validator@0.4.0"
  }
}
```

## Связанные документы

- `docs/specs/strategy-schema.md`
- `docs/specs/tool-schemas.md`
- `docs/specs/offline-eval-benchmarks.md`
- `docs/specs/observability-and-evals.md`
- `docs/specs/analytics-snapshot-schema.md`
