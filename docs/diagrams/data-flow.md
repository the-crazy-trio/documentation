# Data Flow

```mermaid
flowchart LR
    user[User message/command] --> gateway[Client Interface]
    gateway --> orchestrator[Orchestrator]

    orchestrator --> toolgw[Retriever / Tool Gateway]
    toolgw --> tinvest[T-Invest API]
    toolgw --> market[Market/Macro providers]

    tinvest --> normp[Normalize to PortfolioSnapshot]
    market --> norma[Normalize indicators/mappings]

    normp --> memory[(Memory Storage)]
    norma --> analytics[(Analytics Storage)]

    memory --> engine[Deterministic Analytics Engine]
    analytics --> engine
    memory --> strategy[StructuredStrategy]
    strategy --> engine

    engine --> ansnap[AnalyticsSnapshot]
    ansnap --> memory

    orchestrator --> ctx[Compact LLM context builder]
    memory --> ctx
    ansnap --> ctx
    ctx --> llm[LLM Provider]
    llm --> response[Grounded response]
    response --> gateway

    orchestrator --> ustore[(User Storage)]
    response --> ustore

    orchestrator --> obs[Logs/metrics/traces]
    toolgw --> obs
    llm --> obs
```

Поток отделяет три класса данных: операционные (`session/request`), доменные (`StructuredStrategy`, snapshots, indicators) и наблюдаемость (`logs/traces`). В storage сохраняются нормализованные сущности с TTL/freshness, а в telemetry пишутся только технические события без секретов.
