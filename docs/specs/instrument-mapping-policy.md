# Политика маппинга инструментов

## Назначение

Этот документ описывает, как система маппит идентификаторы и метаданные инструментов из T-Invest API во внутренние аналитические dimensions.

Политика нужна для того, чтобы:
- одинаково обрабатывать `figi`, `instrument_uid`, `ticker`, `class_code`, `position_uid`;
- стабильно строить allocations и concentration metrics;
- корректно считать coverage;
- не смешивать broker-native типы с внутренними бизнес-классами активов.

## Статус документа

- Версия: `0.1.0`
- Статус: `working-draft`
- Владелец: `favead`
- Последнее обновление: `2026-03-31`

## Источники идентификации

С учетом T-Invest API инструмент может приходить или искаться по:
- `instrument_uid`
- `figi`
- `ticker + class_code`
- `position_uid`
- универсальному `id`

Для PoC канонический приоритет идентификаторов такой:
1. `instrument_uid`
2. `figi`
3. `ticker + class_code`
4. `position_uid` только как вспомогательная связь позиции

## Каноническая идентичность инструмента

Внутри системы каждый инструмент должен нормализоваться до объекта:

```json
{
  "instrument_uid": "uid_123",
  "figi": "figi_123",
  "ticker": "SBER",
  "class_code": "TQBR",
  "position_uid": "pos_123",
  "canonical_instrument_key": "uid:uid_123"
}
```

Если `instrument_uid` недоступен, канонический ключ формируется как:
- `figi:<figi>`
- иначе `ticker:<ticker>:<class_code>`

## Основной workflow маппинга

### Шаг 1. Нормализация broker payload

Из `GetPortfolio` / `GetPositions` извлекаются все доступные идентификаторы позиции и инструмента.

### Шаг 2. Enrichment через `GetInstrumentBy`

Если есть `instrument_uid` или `figi`, enrichment выполняется по ним.

Если есть только `ticker`, обязательно используется связка:
- `idType = TICKER`
- `classCode`

### Шаг 3. Внутренняя аналитическая нормализация

После enrichment система вычисляет:
- `normalized_asset_class`
- `sector`
- `currency`
- `lot`
- `instrument_name`

Для текущего PoC `country` не входит в обязательный контур enrichment и не должен блокировать аналитику или strategy-fit-анализ.

### Шаг 4. Coverage classification

Для каждой позиции отдельно фиксируется:
- полное покрытие;
- частичное покрытие;
- unknown mapping.

## Маппинг broker type -> normalized_asset_class

Рекомендуемый default mapping для PoC:

| Broker / instrument type | normalized_asset_class |
|---|---|
| `share` | `stocks` |
| `bond` | `bonds` |
| `etf` | `etf` |
| `currency` | `currencies` |
| `futures` / `future` | `futures` |
| `option` | `options` |
| money balance | `cash` |
| unknown / unsupported | `unknown` |

Если T-Invest тип не совпадает напрямую с бизнес-классом, используется отдельная mapping table в коде, а не `if/else` по всей системе.

## Правила маппинга sector и country

### Sector

- sector не считается надежно доступным только из broker response;
- sector должен приходить из внутренней mapping table;
- если sector не найден, используется `unknown_sector`.

### Country

- country не используется как обязательная dimension в текущем PoC;
- страна не должна угадываться из ticker;
- если информация о стране появится позже, она должна приходить из отдельного reference metadata слоя, а не из эвристик.

## Политика конфликтов источников

Если разные источники дают разную классификацию:
1. внутренняя validated mapping table
2. прямой enrichment из T-Invest instrument metadata
3. fallback на `unknown`

LLM не используется для разрешения конфликтов классификации.

## Правила unknown / partial mapping

Позиция считается `unknown`, если:
- нет устойчивого идентификатора инструмента;
- enrichment не дал базовый `instrument_type`;
- позицию нельзя уверенно отнести ни к одному supported asset class.

Позиция считается `partial`, если:
- есть базовый идентификатор и asset class;
- но отсутствует `sector`;
- или отсутствует часть enrichment fields.

## Влияние на `AnalyticsSnapshot`

Результат маппинга обязан попадать в snapshot в виде:
- `normalized_asset_class`
- `sector`
- `is_unknown_mapping`
- `source_flags.classification_enriched`

И дополнительно отражаться в:
- `coverage.covered_market_value_pct`
- `coverage.unknown_assets_count`
- `coverage.unknown_market_value_pct`
- `coverage.unsupported_dimensions`

## Правила для strategy-fit-анализа

- asset-class checks можно выполнять только если есть `normalized_asset_class`;
- sector checks можно выполнять только при достаточном sector coverage;
- country checks не входят в обязательный контур текущего PoC;
- если coverage недостаточно, constraint должен быть `unchecked`, а не `violated`.

## Кэширование и reuse

Результат enrichment по инструменту можно кэшировать отдельно от portfolio snapshot, если:
- `instrument_uid` / `figi` стабилен;
- metadata меняется редко;
- есть версия reference mapping.

Рекомендуемые поля кэша:
- `canonical_instrument_key`
- `instrument_uid`
- `figi`
- `ticker`
- `class_code`
- `instrument_type`
- `normalized_asset_class`
- `sector`
- `currency`
- `lot`
- `metadata_version`

## Failure modes

- `GetInstrumentBy` вернул `404` или аналогичный not found: позиция уходит в `unknown`;
- есть `ticker`, но нет `class_code`: нельзя надежно искать по ticker path;
- instrument metadata пришла, но не содержит нужной classification: позиция становится `partial`;
- reference mapping конфликтует с T-Invest metadata: брать внутренний validated mapping и логировать расхождение.

## Зафиксированное решение для PoC

- Source of truth для enrichment: внутренняя mapping table.
- Ключ маппинга: `instrument_uid`, резервно `figi`.
- Обязательные поля enrichment: `normalized_asset_class`, `sector`.
- `Country` не является обязательной dimension для текущего PoC.

## Минимальные тест-кейсы для реализации

1. Позиция с `instrument_uid` и полным enrichment.
2. Позиция только с `figi`.
3. Позиция только с `ticker + class_code`.
4. Позиция без sector/country.
5. Unknown instrument.
6. Конфликт классификации между источниками.

## Связанные документы

- `docs/specs/analytics-snapshot-schema.md`
- `docs/specs/tinvest-api-contract.md`
- `docs/specs/portfolio-metrics-formulas.md`
