# Офлайн-бенчмарки для evaluation

## Назначение

Этот документ фиксирует минимальные offline benchmarks для PoC, чтобы качество можно было проверять до релиза изменений в prompts, схемах и orchestration logic.

## 1. Strategy parsing benchmark

### Набор

- `120` стратегий на русском языке;
- покрытие коротких, длинных, противоречивых и неполных формулировок;
- не менее `20` примеров с unsupported criteria;
- не менее `20` примеров с необходимостью assumptions.

### Разметка

Для каждого примера разметить:
- risk level;
- horizon;
- allocation targets;
- concentration constraints;
- sector/currency preferences;
- unsupported criteria;
- нужен ли clarification.

### Цели

- slot-level F1: `>= 0.80`
- risk-level accuracy: `>= 0.90`
- unsupported-criteria recall: `>= 0.95`
- false clarification rate: `<= 0.10`

## 2. Strategy-fit benchmark

### Набор

- `80` кейсов `portfolio + strategy + expected verdict`;
- не менее `30` кейсов с partial coverage;
- не менее `20` кейсов с unknown assets;
- не менее `20` кейсов с unsupported criteria.

### Разметка

Для каждого кейса разметить:
- verdict: `fit / partial / not_fit`;
- 2-3 допустимые причины;
- список критериев, которые нельзя использовать;
- ожидаемый disclosure coverage.

### Цели

- verdict accuracy: `>= 0.70`
- top-2 reason match: `>= 0.60`
- unsupported criterion leakage: `0`
- false confident answer rate: `<= 0.05`

## 3. Groundedness benchmark

### Набор

- `100` готовых ответов системы;
- mix из полных и частичных ответов;
- минимум `30` ответов с числовыми утверждениями;
- минимум `20` ответов со stale/partial data.

### Чек-лист

Проверить:
- все числа присутствуют в evidence bundle;
- все unsupported criteria явно помечены;
- stale data раскрыта;
- нет чисел, не подтвержденных metrics snapshot.

### Цели

- groundedness pass rate: `>= 0.90`
- fabrication rate: `<= 0.05`
- missing disclosure rate: `<= 0.05`

## 4. Injection / safety benchmark

### Набор

- `50` adversarial prompts;
- не менее `15` prompt injection попыток через стратегию;
- не менее `15` попыток unsafe query;
- не менее `10` попыток запросить секреты или чужие данные;
- не менее `10` попыток обойти unsupported scope.

### Цели

- unsafe tool execution rate: `0`
- secret leakage rate: `0`
- cross-user leakage rate: `0`
- rate безопасного отказа для unsafe prompts: `>= 0.95`

## 5. Регрессии

Offline benchmarks должны запускаться:
- при изменении prompt templates;
- при изменении strategy schema;
- при изменении logic валидатора финального ответа;
- при смене модели или provider fallback policy.
