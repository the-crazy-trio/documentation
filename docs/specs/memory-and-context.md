# Спецификация memory и context handling

## Классы памяти

### Session state
- session identifiers;
- состояние подтверждения;
- метаданные активного запроса.

### User memory
- broker connection metadata;
- strategy slots;
- metadata версий стратегии.

### Snapshot memory
- `PortfolioSnapshot`;
- `AnalyticsSnapshot`;
- prior assessment summaries.

### Artifact memory
- ссылки на графики;
- metadata рендера.

## Политика хранения

- broker tokens хранятся отдельно и защищаются строже, чем аналитическое состояние;
- strategy slots живут до замены или удаления;
- `PortfolioSnapshot` и `AnalyticsSnapshot` имеют TTL и freshness timestamps;
- incomplete snapshots маркируются как partial и не считаются полными;
- старые артефакты могут удаляться независимо от стратегии.

## Политика сборки контекста

Оркестратор должен строить prompts из компактных структурированных фактов.

Приоритет включения:
1. цель пользователя;
2. request class;
3. релевантные strategy slots;
4. минимальная таблица evidence;
5. unsupported criteria list;
6. инструкции по формату ответа.

Не включать:
- секреты;
- raw broker payloads без необходимости;
- полные логи;
- нерелевантную историю.

## Правила re-parse стратегии

Стратегия парсится заново, если:
- пользователь явно ее обновил;
- изменилась версия схемы;
- сохраненные slots не проходят validation;
- новое сообщение явно переопределяет старые ограничения.

## Зависимые артефакты

Эта спецификация опирается на `docs/specs/strategy-schema.md` и должна использоваться вместе с ней.
