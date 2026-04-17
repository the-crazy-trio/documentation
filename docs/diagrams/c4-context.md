# C4 Context

```mermaid
flowchart LR
    user[Частный инвестор]
    client[Client Interface<br/>Telegram/CLI]

    subgraph boundary[Financial AI Assistant PoC]
        orchestrator[Orchestrator + Workflow Layer]
        retrieval[Retriever / Tool Layer<br/>guarded access]
        analytics[Deterministic Analytics Engine]
        storage[(User + Memory + Analytics Storage)]
        observability[Observability / Evals]
    end

    llm[OpenRouter / LLM Provider]
    broker[T-Invest API<br/>read-only]
    market[Market/Macro Providers]

    user --> client
    client --> orchestrator
    orchestrator --> retrieval
    orchestrator --> analytics
    orchestrator --> storage
    orchestrator --> observability
    retrieval --> broker
    retrieval --> market
    orchestrator --> llm
```

Границы показывают ответственность: внутри PoC остаются роутинг, validation gate, fallback и user-scope контроль; снаружи только провайдеры данных/LLM без права менять внутренние политики доступа.
