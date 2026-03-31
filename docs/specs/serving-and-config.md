# Спецификация serving и конфигурации

## Runtime configuration

Обязательные конфигурируемые параметры:
- LLM provider и модель по request class;
- provider timeouts;
- retry limits;
- max input/output token caps;
- broker API endpoint и timeout;
- market/macro source endpoints и freshness tolerances;
- snapshot TTLs;
- timeout и resource limits для rendering;
- feature flags для text2sql/text2api и visualization.

## Storage decisions for PoC

- `User Storage`: `MongoDB`
- `Memory Storage`: `MongoDB`
- `Analytics Storage`: `SQLite`

Эти решения считаются допустимыми для PoC и могут быть пересмотрены после появления требований по масштабированию, конкурентной записи или объему исторических данных.

## Работа с секретами

- секреты приходят из environment или secret storage, а не из статических файлов;
- broker credentials и provider keys редактируются из логов;
- доступ к секретам есть только у backend-компонентов, которым он нужен.

## Версионирование

- model names и prompt/schema versions должны быть явными в конфиге;
- версия формул аналитики должна сохраняться в `AnalyticsSnapshot`;
- версия strategy schema должна сохраняться вместе с parsed strategy.

## Budget classes

- легкие запросы: низкий token cap, короткий timeout, без тяжелого пересчета по умолчанию;
- тяжелые запросы: больший token cap, больший timeout, возможен пересчет аналитики и визуализация;
- provider fallback должен уважать cost ceiling для каждого класса запроса.
