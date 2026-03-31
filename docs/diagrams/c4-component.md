# C4 Component Diagram

```mermaid
flowchart TB
    intake[Прием запроса]
    classifier[Классификатор intent]
    policy[Policy / Guardrail Check]
    context[Сборщик контекста]
    planner[Планировщик инструментов]
    strategy[Парсер стратегии]
    exec[Координатор исполнения]
    validator[Валидатор финального ответа]
    composer[Генератор ответа]
    fallback[Обработчик fallback]
    response[Ответ пользователю]

    intake --> classifier
    classifier --> policy
    policy --> context
    context --> planner
    planner --> strategy
    planner --> exec
    strategy --> exec
    exec --> validator
    validator --> composer
    composer --> response
    validator --> fallback
    fallback --> response
```

Ядро системы организовано как последовательность явных контрольных ворот. Каждый запрос проходит через классификацию, policy checks, сбор контекста, контролируемое исполнение и synthesis, проверяемый валидатором финального ответа. Если данные неполные или итоговый текст невалиден, система уходит в fallback вместо того, чтобы позволить LLM импровизировать.
