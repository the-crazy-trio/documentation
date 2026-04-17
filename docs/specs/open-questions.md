# Open Questions / Decisions

## Закрыто

1. **Request classes**
- Решение: `request_class` 1:1 соответствует workflow.
- Зафиксировано: `docs/specs/orchestrator-layer.md`.

2. **Timeout/retry/token/freshness policy**
- Решение: лимиты задаются через runtime-конфиг; в workflow не хардкодятся.
- Зафиксировано: `docs/specs/orchestrator-layer.md`, `docs/specs/retriever-tool-layer.md`, `docs/specs/serving-config.md`, `docs/specs/storage-layer.md`.

3. **Must-have стратегия**
- Решение: используется расширенная структура `StructuredStrategy`.
- Зафиксировано: `docs/specs/workflow-layer.md`.

4. **Аналитический каркас strategy-fit / asset analysis**
- Решение: top-down + bottom-up, техника/сентимент как overlay, hard business rules обязательны.
- Зафиксировано: `docs/specs/deterministic-analytics-layer.md`, `docs/specs/observability-evals-layer.md`.

5. **Secret management в PoC**
- Решение: env-only.
- Зафиксировано: `docs/specs/serving-config.md`.

6. **Язык ответа**
- Решение: русский в приоритете.
- Зафиксировано: `docs/specs/client-interface-layer.md`.

7. **Визуализация**
- Решение: вне scope PoC и не планируется post-PoC.
- Зафиксировано: `docs/system-design.md` и актуальная структура `docs/specs/`.

8. **T-Invest error mapping table**
- Решение: введена базовая нормализующая таблица upstream -> internal codes.
- Основание: `https://developer.tbank.ru/invest/api`
- Зафиксировано: `docs/specs/retriever-tool-layer.md`.

9. **Политика хранения логов/трейсов**
- Решение: введен retention по окружениям (`dev/staging/prod`) + redaction policy.
- Зафиксировано: `docs/specs/observability-evals-layer.md`.

10. **Versioning changelog format**
- Решение: Keep a Changelog + SemVer, единый файл changelog.
- Зафиксировано: `docs/specs/serving-config.md`, `docs/specs/changelog.md`.

## Остается уточнить (не блокирует PoC)

1. Конкретные model IDs для `DEFAULT_MODEL_INTENT` и `DEFAULT_MODEL_SYNTHESIS` по окружениям.
