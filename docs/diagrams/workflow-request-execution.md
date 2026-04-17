# Workflow / Request Graph

```mermaid
flowchart TD
    start([1. User request]) --> intake[2. Intake: validate user/payload<br/>assign correlation_id]
    intake --> intakeok{Intake valid?}
    intakeok -->|No| e1[Return error: VALIDATION_ERROR]
    intakeok -->|Yes| classify[3. Intent classification]

    classify --> route{4. Route primary workflow}
    route -->|strategy_parsing| w1[Run Strategy Parsing Workflow]
    route -->|portfolio_analysis / strategy_fit / asset_analysis| w2[Run Portfolio/Analytics Workflow]
    route -->|portfolio_qa| w3[Run Guarded Q&A Workflow]

    w1 --> svalid{StructuredStrategy complete?}
    svalid -->|No| clarify[Ask one clarification OR mark unsupported]
    svalid -->|Yes| save[Persist strategy version]

    w2 --> fresh{Fresh snapshots exist?}
    fresh -->|Yes| usecache[Use cached snapshots]
    fresh -->|No| collect[Collect portfolio via read-only tools]
    collect --> apiok{Tool/API call succeeded?}
    apiok -->|Retryable| retry[Retry with backoff (limited)]
    retry --> apiok
    apiok -->|Failed| f1[Fallback to latest valid snapshot]
    apiok -->|Succeeded| enrich[Load market/macro + mappings]
    enrich --> cov{Coverage above threshold?}
    cov -->|No| markpartial[Mark partial + unsupported criteria]
    cov -->|Yes| compute[Run deterministic analytics/strategy-fit]
    markpartial --> compute

    w3 --> safe{Guarded query policy pass?}
    safe -->|No| f2[Return partial/unavailable with reason]
    safe -->|Yes| qexec[Execute allowlisted retrieval]

    usecache --> validate
    f1 --> validate
    compute --> validate
    qexec --> validate
    clarify --> validate
    save --> validate

    validate{5. Validation gate pass?<br/>evidence + freshness + user-scope}
    validate -->|No| f3[Safe degraded response]
    validate -->|Yes| synth[6. Response synthesis]
    synth --> persist[7. Persist snapshots/summary/log refs]
    f2 --> persist
    f3 --> persist
    e1 --> done
    persist --> done([Return status: ok/partial/unavailable])
```

Диаграмма фокусируется на управлении рисками: где разрешены ретраи, где обязателен fallback, и где система должна вернуть `partial|unavailable`, а не продолжать генерацию без данных.
