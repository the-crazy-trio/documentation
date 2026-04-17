# Спецификация: Storage Layer

## Назначение

Хранение пользовательского состояния, стратегии и аналитических snapshot-данных для переиспользования.

## Компоненты

1. `User Storage`
- `user_id`, session metadata
- Broker connection metadata
- Служебные ссылки на сохраненные сущности

2. `Memory Storage`
- `StructuredStrategy` + история версий
- `PortfolioSnapshot`
- `AnalyticsSnapshot`
- Саммари после воркфлоу (запрос юзера <-> ответ ассистента)

3. `Analytics Storage`
- Макро и микро показатели с изменением во времени
- instrument mapping table
- Данные показателей

## Политика хранения

- `StructuredStrategy` хранится до обновления/удаления
- Снапшоты имеют TTL и freshness metadata (TTL задается в едином runtime-конфиге)

## Состояние сессии

Short-lived state:
- `user_id`, `chat_id`, `session_id`
- `request_class`
- `correlation_id`
- `confirmation_flags`
