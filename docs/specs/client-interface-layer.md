# Спецификация: Client Interface Layer (Telegram/CLI)

## Назначение

Единая точка взаимодействия пользователя с системой: прием сообщений/команд, показ ответов, запрос подтверждений.

## Вход

- `user_id`, `chat_id`, `session_id`
- `message_text` или команда
- опционально `account_id`

## Выход

```json
{
  "request_id": "req_...",
  "status": "ok|partial|unavailable|error",
  "summary": "...",
  "details": ["..."],
  "evidence": {
    "as_of_ts": "...",
    "coverage": 0.0,
    "unsupported_criteria": []
  },
  "errors": []
}
```

## Правила

- `summary` обязателен всегда
- Для `partial|unavailable` обязательно указать причину фейла при ответе
- Язык ответа PoC по умолчанию: русский
