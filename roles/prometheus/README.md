# prometheus

Ставит Prometheus, генерирует цели из inventory и раскладывает правила алертов.

- скачивание с проверкой sha256
- конфиг и правила проверяются `promtool` до записи на сервер
- юнит проверяется `systemd-analyze verify`
- смена конфига → reload (SIGHUP), смена бинарника или юнита → restart
- цели `node`: все хосты группы `nodes`, адрес из `ansible_host`

## Переменные

| Переменная                        | По умолчанию          |
|-----------------------------------|-----------------------|
| `prometheus_version`              | `2.53.0`              |
| `prometheus_listen_address`       | `0.0.0.0:9090`        |
| `prometheus_retention_time`       | `30d`                 |
| `prometheus_scrape_interval`      | `15s`                 |
| `prometheus_node_exporter_port`   | `9100`                |
| `prometheus_alertmanager_address` | `""` — без alerting   |
| `prometheus_config_dir`           | `/etc/prometheus`     |
| `prometheus_data_dir`             | `/var/lib/prometheus` |
