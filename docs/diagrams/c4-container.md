# C4 Container

```mermaid
flowchart LR
    user[Investor]
    frontend[Frontend<br/>Telegram/CLI Gateway]

    subgraph backend[Backend PoC]
        orchestrator[Orchestrator Service]
        workflows[Workflow Layer<br/>fixed workflows]
        analytics[Deterministic Analytics Engine]
        retriever[Retriever Gateway]
        tools[Tool Layer<br/>portfolio_collect/show,<br/>strategy_save/fit,<br/>guarded query]
        response[Response Synthesis Adapter]
        validation[Validation Gate]
    end

    subgraph data[Storage Layer]
        userdb[(User Storage)]
        memorydb[(Memory Storage)]
        analyticsdb[(Analytics Storage)]
    end

    obs[Observability / Evals]
    llm[OpenRouter]
    tinvest[T-Invest API<br/>read-only]
    market[Market/Macro Providers]

    user --> frontend
    frontend --> orchestrator
    orchestrator --> workflows
    workflows --> analytics
    workflows --> retriever
    retriever --> tools
    tools --> tinvest
    tools --> market
    tools --> memorydb
    tools --> analyticsdb
    orchestrator --> validation
    validation --> response
    response --> llm
    orchestrator --> userdb
    workflows --> memorydb
    analytics --> analyticsdb
    orchestrator --> obs
    tools --> obs
    response --> obs
```

Диаграмма разделяет ответственность контейнеров: workflow и аналитика принимают решения детерминированно, tool layer изолирует внешние источники, а LLM используется только на этапах классификации/синтеза через контролируемые адаптеры.
