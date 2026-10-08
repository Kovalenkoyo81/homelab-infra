# alertmanager

Ставит Alertmanager с отправкой уведомлений в Telegram.

- скачивание с проверкой sha256
- конфиг проверяется `amtool check-config` до записи; задача под `no_log`,
  потому что в конфиге токен бота
- группировка по `alertname` + `instance`, повтор раз в 4 часа
- critical-алерт подавляет warning того же хоста
- слушает только `127.0.0.1` — наружу не торчит

## Переменные

| Переменная                        | По умолчанию     |
|-----------------------------------|------------------|
| `alertmanager_version`            | `0.27.0`         |
| `alertmanager_listen_address`     | `127.0.0.1:9093` |
| `alertmanager_resolve_timeout`    | `5m`             |
| `alertmanager_telegram_bot_token` | — из vault       |
| `alertmanager_telegram_chat_id`   | — из vault       |
