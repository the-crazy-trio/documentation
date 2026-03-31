# C4 Context Diagram

```mermaid
flowchart TB
    user[Частный инвестор]
    ui[Telegram Bot / Mini App]
    system[Financial AI Assistant PoC]
    llm[OpenRouter LLMs]
    broker[T-Invest API]
    market[Источники market / macro data]
    sandbox[Sandbox Renderer]
    storage[(User / Memory / Analytics Storage)]
    obs[Observability / Evals]

    user --> ui
    ui --> system
    system --> llm
    system --> broker
    system --> market
    system --> sandbox
    system --> storage
    system --> obs
```

Граница системы включает backend PoC, его оркестратор, хранилища и execution control logic. Внешние сервисы дают инференс, брокерские данные, рыночные данные и рендеринг, но ответственность за permissioning, validation, fallback и безопасный пользовательский вывод остается внутри системы.
