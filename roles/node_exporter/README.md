# node_exporter

Ставит Prometheus node_exporter из официальных релизов GitHub.

- скачивание с проверкой sha256 по `sha256sums.txt` релиза
- системный пользователь `node_exporter` без shell и домашнего каталога
- systemd-юнит с `NoNewPrivileges`
- архитектура определяется по фактам: amd64, arm64, armv7

## Переменные

| Переменная                     | По умолчанию   |
|--------------------------------|----------------|
| `node_exporter_version`        | `1.8.2`        |
| `node_exporter_user`           | `node_exporter`|
| `node_exporter_listen_address` | `0.0.0.0:9100` |
