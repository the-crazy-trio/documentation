# Поток данных

```mermaid
flowchart TD
    user[Сообщение / команда пользователя] --> api[API Layer]
    api --> orch[Orchestrator]
    orch --> broker[T-Invest API]
    orch --> market[Market / Macro Sources]
    broker --> rawp[Временный broker payload]
    market --> rawm[Временный market payload]
    rawp --> mem[(Memory Storage)]
    rawm --> analytics[(Analytics Storage)]
    mem --> calc[Детерминированная аналитика]
    analytics --> calc
    calc --> snap[Analytics snapshot]
    snap --> mem
    mem --> llm[Ограниченный LLM context]
    snap --> llm
    llm --> answer[Grounded response]
    calc --> viz[Подготовленный dataset для графика]
    viz --> sandbox[Sandbox Renderer]
    sandbox --> artifacts[(Artifact refs в User Storage)]
    orch --> logs[Логи / метрики / трассы]
```

Хранение минимизируется: сырые payloads по возможности остаются временными, snapshots живут по TTL, артефакты хранятся по ссылке, а логи содержат технические события вместо секретов и полных чувствительных payloads.
