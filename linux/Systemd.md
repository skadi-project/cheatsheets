# systemd — шпаргалка

## Что это и зачем

systemd управляет загрузкой Linux, службами, журналами, таймерами, сокетами и пользовательскими сессиями. Он используется в большинстве современных Debian, Ubuntu и Arch Linux. OpenRC — альтернатива с другим подходом к службам и зависимостям.

## Root и user

Есть два независимых менеджера systemd.

| Область | Команды | Где находятся юниты | Для чего |
|---|---|---|---|
| Система | `sudo systemctl ...` | `/etc/systemd/system`, `/usr/lib/systemd/system` | Общие службы сервера, сеть, SSH |
| Пользователь | `systemctl --user ...` | `~/.config/systemd/user` | Процессы конкретного пользователя без root |

`systemctl --user` не управляет системными службами. `sudo systemctl` не видит пользовательские юниты текущего пользователя. Для работы user-служб после выхода из сессии включите lingering, если политика системы это допускает.

```bash
loginctl enable-linger "$USER"
```

Команда выше требует root. Отменить можно через `sudo loginctl disable-linger "$USER"`.

## Быстрый старт для root

Создайте системную службу только для проверенного приложения. Редактируйте файл через `sudo` и проверяйте конфигурацию до запуска.

```ini
# /etc/systemd/system/example.service
[Unit]
Description=Example service
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
User=example
ExecStart=/usr/local/bin/example-server --listen 127.0.0.1:8080
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now example.service
systemctl status example.service --no-pager
sudo journalctl -u example.service -n 50 --no-pager
```

Сервис запускается от `example`, хотя управление выполняется root. Не указывайте `User=root`, если приложение не требует привилегий.

## Быстрый старт для user

Создайте пользовательский юнит в домашнем каталоге.

```ini
# ~/.config/systemd/user/example-worker.service
[Unit]
Description=Example user worker

[Service]
ExecStart=%h/bin/example-worker
Restart=on-failure

[Install]
WantedBy=default.target
```

```bash
systemctl --user daemon-reload
systemctl --user enable --now example-worker.service
systemctl --user status example-worker.service --no-pager
journalctl --user -u example-worker.service -n 50 --no-pager
```

Без lingering user-служба обычно работает только пока активна пользовательская сессия.

## Основные команды

| Действие | Root, система | User, текущий пользователь |
|---|---|---|
| Список служб | `sudo systemctl list-units --type=service` | `systemctl --user list-units --type=service` |
| Состояние | `sudo systemctl status nginx` | `systemctl --user status app.service` |
| Запуск | `sudo systemctl start nginx` | `systemctl --user start app.service` |
| Остановка | `sudo systemctl stop nginx` | `systemctl --user stop app.service` |
| Перезапуск | `sudo systemctl restart nginx` | `systemctl --user restart app.service` |
| Автозапуск | `sudo systemctl enable nginx` | `systemctl --user enable app.service` |
| Запуск и автозапуск | `sudo systemctl enable --now nginx` | `systemctl --user enable --now app.service` |
| Отключить автозапуск | `sudo systemctl disable nginx` | `systemctl --user disable app.service` |
| Перечитать юниты | `sudo systemctl daemon-reload` | `systemctl --user daemon-reload` |
| Логи | `sudo journalctl -u nginx` | `journalctl --user -u app.service` |
| Показать юнит | `systemctl cat nginx` | `systemctl --user cat app.service` |

`enable` меняет ссылки автозапуска, но сам по себе не запускает службу. `enable --now` выполняет оба действия.

## Юниты и зависимости

```bash
systemctl list-unit-files --state=enabled
systemctl list-dependencies nginx.service
systemctl show nginx.service -p ActiveState,SubState,MainPID
systemctl is-active nginx.service
systemctl is-enabled nginx.service
```

Типы юнитов включают `.service`, `.socket`, `.timer`, `.mount`, `.path` и `.target`. Локальные изменения делайте через drop-in, а не редактированием файла пакета.

```bash
sudo systemctl edit nginx.service
sudo systemctl revert nginx.service
```

## Таймер вместо cron

```ini
# ~/.config/systemd/user/backup.timer
[Unit]
Description=Run backup every day

[Timer]
OnCalendar=*-*-* 03:00:00
Persistent=true

[Install]
WantedBy=timers.target
```

```ini
# ~/.config/systemd/user/backup.service
[Service]
Type=oneshot
ExecStart=%h/bin/backup
```

```bash
systemctl --user daemon-reload
systemctl --user enable --now backup.timer
systemctl --user list-timers
```

## Частые ошибки

| Ошибка | Причина | Решение |
|---|---|---|
| `Unit not found` | Неверное имя или юнит не установлен | Проверьте `systemctl list-unit-files` и `systemctl cat` |
| `Job failed` | Ошибка запуска или зависимости | Смотрите `systemctl status` и `journalctl -u` |
| `Changed on disk` | Файл юнита изменён после загрузки менеджером | Выполните `daemon-reload` |
| User unit не стартует после выхода | Не включён lingering | Используйте `loginctl enable-linger` с root при необходимости |
| `203/EXEC` | `ExecStart` не найден или не исполняемый | Проверьте абсолютный путь и права `x` |
| `217/USER` | Указанный пользователь отсутствует | Проверьте `getent passwd example` |

## Документация

- [systemd](https://systemd.io/)
- [systemctl](https://www.freedesktop.org/software/systemd/man/latest/systemctl.html)
- [systemd.service](https://www.freedesktop.org/software/systemd/man/latest/systemd.service.html)
- [systemd user services](https://www.freedesktop.org/software/systemd/man/latest/systemd.special.html)

- [Администрирование серверов Linux](Linux-Server-Admin.md)
- [OpenRC](OpenRC.md)
