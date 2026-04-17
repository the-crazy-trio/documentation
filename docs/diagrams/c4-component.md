# C4 Component (Core)

```mermaid
flowchart TD
    intake[Intake Handler<br/>user/session/payload checks]
    corr[Correlation ID Manager]
    classifier[Intent Classifier Adapter<br/>LLM bounded role]
    policy[Policy Engine<br/>scope + max-age + limits]
    context[Context Loader<br/>strategy + snapshots]
    router[Workflow Router<br/>1 request -> 1 workflow]
    wfexec[Workflow Executor]
    tools[Tool Orchestrator<br/>allowlist + retry/fallback]
    calc[Deterministic Analytics Module]
    vgate[Validation Gate<br/>evidence/freshness/user-scope]
    synth[Response Synthesizer Adapter<br/>LLM bounded role]
    persist[Persistence Coordinator]
    fallback[Fallback Manager<br/>partial/unavailable path]
    out[Response Presenter]

    intake --> corr
    corr --> classifier
    classifier --> policy
    policy --> context
    context --> router
    router --> wfexec
    wfexec --> tools
    wfexec --> calc
    tools --> vgate
    calc --> vgate
    vgate -->|pass| synth
    vgate -->|fail| fallback
    synth --> persist
    fallback --> persist
    persist --> out
```

Компонентная схема показывает, что генерация текста отделена от вычислений: до `Response Synthesizer` проходят только валидационные результаты workflow, а при провале validation система принудительно идет в fallback-ветку.
