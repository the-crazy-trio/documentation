# Дизайн PoC-системы

## Назначение

Документ фиксирует архитектуру PoC `Financial AI Assistant` и границы реализации.

PoC покрывает:
- анализ портфеля и Q&A по портфелю;
- онбординг/обновление инвестиционной стратегии;
- детерминированную проверку соответствия портфеля стратегии;
- анализ отдельного актива в рамках доступного data coverage.

## Ключевые архитектурные решения

1. **Один оркестратор + фиксированные workflow**  
   Оркестратор классифицирует запрос и запускает заранее определенный workflow вместо свободного agent-loop.

2. **LLM только в bounded-ролях**  
   LLM используется для intent parsing, strategy parsing и grounded synthesis; расчеты и вердикты — детерминированные.

3. **Snapshot-first подход**  
   Рабочая единица данных: `PortfolioSnapshot` и `AnalyticsSnapshot` с freshness metadata и TTL.

4. **Guarded retrieval**  
   Любой доступ к данным проходит через allowlist-инструменты с `user_id` фильтрацией, row/time limits и schema validation.

5. **Read-only внешние интеграции**  
   T-Invest API используется только в read-only режиме; write side effects отсутствуют.

## Основные модули

1. `Client Interface (Telegram/CLI)`  
   Вход сообщений/команд, подтверждения чувствительных операций, доставка ответа

2. `Orchestrator`  
   Классификация запроса, сбор минимального контекста, запуск workflow, финальная валидация ответа

3. `Workflow Layer`  
   Набор предсказуемых потоков:
   - `Strategy Parsing Workflow`
   - `Portfolio Analysis Workflow`
   - `Strategy-Fit Workflow`
   - `Asset Analysis Workflow`
   - `Portfolio Q&A Workflow`

4. `Deterministic Analytics Engine`
   Предустановленная логика оценки активов на основе показателей

5. `Retriever / Tool Layer`
   Адаптеры `T-Invest`, макро и микро данных, память, аналитика

6. `Storage Layer`
   - `User Storage`: user/session/broker connection metadata
   - `Memory Storage`: `StructuredStrategy`, стратегии, снапшоты, саммари предыдущих воркфлоу
   - `Analytics Storage`: показатели, их изменение во времени и прочая информация используемая при анализе

7. `Observability / Evals`
   > Трассировки, latency/cost/retry метрики, error logs
   > Эвалы на небольших golden set по каждому из воркфлоу

## Основной workflow выполнения

1. **Intake**
   Валидация пользователя и payload, назначение `correlation_id`

2. **Intent classification**
   Роутинг по воркфлоу от orchestartor агента

3. **Context load**
   Загрузка релевантного минимума: `StructuredStrategy`, `PortfolioSnapshot`, `AnalyticsSnapshot`

4. **Execution**
   Запуск выбранного workflow: сбор данных, пересчет аналитики, парсинг стратегии и т.д.

5. **Validation**
   Проверка цепочки рассуждений из ворфлоу

6. **Response synthesis**
   LLM синтезирует ответ только после валидации заключения воркфлоу по заданному списку критериев

7. **Persist**
   Сохранение необходимых сущностей: стратегия, snapshots, assessment summary

## State / Memory / Context handling

### Session state (short-lived)

- `user_id`, `chat_id`, `session_id`;
- Request class;
- Confirmation flags;
- `correlation_id`.

### Long-lived memory

- Broker connection metadata
- Strategy slots и история версий
- Последние `PortfolioSnapshot` и `AnalyticsSnapshot`
- Саммари прошлых strategy-fit оценок

### Политика памяти

- `StructuredStrategy` хранится до обновления/удаления
- Снапшоты портфеля имеют TTL
- Частичные результаты при нехватке данных помечаются `partial=true`

### Политика контекста для LLM

В prompt включается только компактный релевантный контекст:
- Тулы
- Инструкции
- Саммари диалога / последние N запросов

## Retrieval-контур

1. `Portfolio retrieval`  
   Источник: T-Invest API и/или кэш

2. `Market/reference retrieval`  
   Источник: analytics storage + внешние провайдеры из whitelist

3. `Memory retrieval`  
   Источник: memory storage

4. `Analytical retrieval`  
   Источник: `AnalyticsSnapshot` и детерминированные метрики

## Tool/API-интеграции

### OpenRouter / LLM Provider

- Вся работа с LLM

### T-Invest API

- Портфель/счета/операции - исключительно в read-only

### Market/Macro Providers

- Цены, классификации, индикаторы

### Guarded Query Tool (Text2SQL/Text2API)

- Безопасные запросы к данным портфеля
- Объединение логически связанных запросов (например тул для макро показателей с возможностью указания списка метрик или keyword по воркфлоу)

## Failure modes, fallback и guardrails

1. **Неполные портфельные данные**  
   Ответ исключительно по доступной части

2. **Неопределенная стратегия**  
   Один уточняющий вопрос или фиксация допущени - неподдержанные критерии помечаются `unsupported`.

3. **Сбой внешних API**  
   Ограниченные ретраи, затем fallback на последнее валидное состояние портфеля (с ограничением по окну времени), иначе `partial/unavailable` ответ.

4. **Hallucination risk**  
   Прорабатывается на уровне валидации ответа и оффлайн бенчмарке

5. **Prompt injection risk**  
   Пользовательский текст не меняет права доступа тулов

6. **Базовые guardrails PoC**  
   Read-only токены, изоляция пользователей, запрет хранения секретов в prompt/логах, обязательный дисклеймер «не инвестиционная рекомендация».

## Технические и операционные ограничения

### Reliability
- Доля успешных ответов: `>= 80%`

### Latency
- Легкие запросы: `<= 1-2 минуты`
- Тяжелые запросы (обновление портфеля + аналитика): `<= 7-8 минут`

### Cost
- Минимизация числа LLM-вызовов
- Приоритет кэша показателей, снапшотов и переиспользования аналитики
- Эффективная работа с контекстом

### Data freshness
- Котировки могут отставать на `15-60 минут`
- Макро данные берутся по последней доступной публикации
- В ответах показываются timestamps/source dates, если критично для интерпретации.
