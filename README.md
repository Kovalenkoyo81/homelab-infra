# homelab-infra

[![CI](https://github.com/Kovalenkoyo81/homelab-infra/actions/workflows/ci.yml/badge.svg)](https://github.com/Kovalenkoyo81/homelab-infra/actions/workflows/ci.yml)

Ansible-роли для мониторинга двух серверов (VPS + одноплатник), связанных
через WireGuard.

## Что разворачивает

- **node_exporter** — на все хосты, бинарник из релизов с проверкой sha256
- **prometheus** — сервер мониторинга, правила алертов
- **alertmanager** — группировка, подавление, уведомления в Telegram
- **admin_user** — пользователь `devops` с sudo по паролю, вход только по ключу,
  root по SSH запрещён
- **firewall** — iptables на VPS: SSH только через WireGuard, правила k3s не
  затрагиваются, откат при потере доступа

```
playbooks/site.yml              — полная раскатка
├── access.yml
│   └── nodes       → admin_user
├── firewall.yml
│   └── firewall    → firewall  (+ пересборка цепочек k3s при изменении)
└── monitoring.yml
    ├── nodes       → node_exporter
    └── monitoring  → prometheus, alertmanager
```

Firewall раскатывается первым, остальное ставится уже за ним. Каждый плейбук
можно запустить и отдельно: ошибка в мониторинге стоит пропущенных метрик,
ошибка в firewall — потерянного доступа.

Если правила firewall изменились, а на хосте есть k3s, он останавливается
(`k3s-killall.sh`) и создаёт свои цепочки заново — тот же порядок, что при
загрузке системы. Роль firewall про k3s не знает: она только уведомляет
`firewall changed`, а перезапуск описан handler'ом в `firewall.yml`.

## Решения и их причины

- **Отдельный системный пользователь на каждый сервис** вместо общего `nobody`:
  компрометация одного сервиса не даёт доступа к файлам другого.
  Плюс `NoNewPrivileges` во всех юнитах и `ProtectHome` у prometheus и
  alertmanager.
- **Права 750 на данные, 640 на конфиги.** Сервис читает свой конфиг, но не
  может его переписать. Каталог с метриками закрыт от остальных пользователей —
  в метриках имена хостов, адреса, топология.
- **Валидация до записи файла.** `promtool check config`, `promtool check rules`,
  `amtool check-config` и `systemd-analyze verify` через параметр `validate`
  модуля `template`: битый конфиг физически не попадает на сервер, старый
  остаётся рабочим.
- **Reload по SIGHUP вместо рестарта** при смене конфига — метрики не теряются.
  Рестарт только при смене бинарника или юнита, для этого два разных handler'а.
- **Цели Prometheus генерируются из inventory.** Новый хост в группе `nodes`
  автоматически попадает в мониторинг.
- **Секреты в ansible-vault**, задача с токеном под `no_log`.
- **Контрольные суммы при скачивании** — на сервер не попадает бинарник,
  который не совпал с официальной суммой.

## Алерты

| Алерт          | Условие                         | Severity |
|----------------|---------------------------------|----------|
| `HostDown`     | `up == 0` дольше 1 мин          | critical |
| `DiskSpaceLow` | свободно < 15% дольше 10 мин    | warning  |
| `MemoryLow`    | доступно < 10% RAM дольше 5 мин | warning  |
| `HighCPU`      | загрузка > 90% дольше 10 мин    | warning  |

`HostDown` подавляет warning-алерты того же хоста (`inhibit_rules`) — если хост
лежит, сообщения про его диск и память только мешают.

## Запуск

Пароль от vault — в `~/.vault_pass_homelab`, вне репозитория. Пароли sudo
у каждого хоста свои и лежат в vault (`ansible_become_password` в `host_vars`),
поэтому `-K` не нужен.

```bash
ansible-playbook playbooks/site.yml --check --diff   # посмотреть, что изменится
ansible-playbook playbooks/site.yml                  # применить всё

ansible-playbook playbooks/firewall.yml              # только firewall
ansible-playbook playbooks/monitoring.yml            # только мониторинг
```

## Проверки

Те же версии, что в CI (`requirements.txt`):

```bash
pip install -r requirements.txt
yamllint .
ansible-lint
ansible-playbook playbooks/site.yml --syntax-check
```

CI запускает их на каждый PR и push в `main`. Vault в CI не расшифровывается:
линтерам и `syntax-check` пароль не нужен, используется фиктивный.

## Проверено

Полный цикл алертинга: остановка node_exporter → алерт в Telegram →
восстановление → уведомление resolved.
