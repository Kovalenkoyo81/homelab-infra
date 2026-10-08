# admin_user

Пользователь для администрирования и закрытие входа под root.

- создаёт `admin_user_name` (по умолчанию `devops`) в группе `sudo`;
  sudo — по паролю, хэш пароля хранится в vault
- добавляет публичные ключи из `admin_user_authorized_keys` (к существующим,
  не заменяя их — чтобы не отрезать доступ с других машин)
- `/etc/ssh/sshd_config.d/10-hardening.conf`: вход по паролю запрещён,
  root по SSH запрещён; конфиг проверяется `sshd -t` до записи
- после reload проверяется новое подключение

## Защита от самоблокировки

Пока Ansible подключён под root, полностью запретить root нельзя — роль
отрезала бы сама себя. В этом случае ставится `PermitRootLogin prohibit-password`
(root только по ключу). Полный запрет — при следующем запуске уже под
`admin_user_name`.

Первый запуск на хосте, где есть только root:

```bash
ansible-playbook playbooks/access.yml --limit <host> -e ansible_user=root -K
ansible-playbook playbooks/access.yml --limit <host> -K   # уже под devops
```

## Переменные

| Переменная                   | По умолчанию     |
|------------------------------|------------------|
| `admin_user_name`            | `devops`         |
| `admin_user_groups`          | `[sudo]`         |
| `admin_user_authorized_keys` | `[]`             |
| `admin_user_password_hash`   | не задан — пароль не меняется |
