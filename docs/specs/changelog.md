# Specs Changelog

Формат: Keep a Changelog + SemVer.

## [Unreleased]

### Changed
- Consolidated spec structure aligned with `docs/system-design.md` layers.

## [1.0.0] - 2026-04-17

### Added
- `client-interface-layer.md`
- `orchestrator-layer.md`
- `workflow-layer.md`
- `deterministic-analytics-layer.md`
- `retriever-tool-layer.md`
- `storage-layer.md`
- `observability-evals-layer.md`
- `serving-config.md`

### Changed
- Removed visualization from PoC scope.
- Introduced runtime-config policy for limits (tokens/timeouts/retries/freshness).

### Removed
- Legacy split specs replaced by consolidated layer-based specs.

## Template (for future entries)

```md
## [X.Y.Z] - YYYY-MM-DD

### Added
- ...

### Changed
- ...

### Fixed
- ...

### Removed
- ...

### Migration notes
- ...

### Eval impact
- baseline: ...
- current: ...
```
