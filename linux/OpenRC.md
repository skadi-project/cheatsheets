# OpenRC — шпаргалка

## Что это и зачем

OpenRC — менеджер служб и зависимостей, используемый в том числе в Gentoo, Alpine и некоторых других Linux-системах. Он управляет системными службами через `rc-service` и уровнями запуска через `rc-update`. В отличие от systemd, OpenRC не является полноценным менеджером журналов и пользовательских служб.

## Root и user

OpenRC в основном управляет системными службами и требует root для изменения состояния.

| Действие | Root | Обычный пользователь |
|---|---|---|
| Состояние службы | `sudo rc-service nginx status` | Обычно нет доступа к управлению системной службой |
| Запуск или остановка | `sudo rc-service nginx start` | Не выполняется без делегированных прав |
| Автозапуск | `sudo rc-update add nginx default` | Не выполняется без root |
| Просмотр уровня запуска | `rc-status` или `sudo rc-status` | `rc-status` может показать доступные сведения |
| Свой процесс пользователя | обычный shell, `nohup`, cron пользователя | Запускается без OpenRC user-manager |

Не выдавайте пользователю полный root без необходимости. Для ограниченного управления используйте тщательно составленное правило `sudoers` через `visudo`. OpenRC не предоставляет аналог `systemctl --user` с отдельным менеджером user-служб.

## Установка и проверка

OpenRC обычно уже является частью системы. Проверка init и команд:

```bash
ps -p 1 -o pid,comm,args
rc-status
rc-update show
```

Если OpenRC установлен как пакет, точная команда установки зависит от дистрибутива. В Alpine используется `apk`, в Gentoo — `emerge`. Не заменяйте работающую init-систему на сервере без плана восстановления и консольного доступа.

## Быстрый старт для root

Скрипты служб обычно находятся в `/etc/init.d`. Создайте службу только для проверенного приложения.

```sh
#!/sbin/openrc-run

name="example"
description="Example service"
command="/usr/local/bin/example-server"
command_args="--listen 127.0.0.1:8080"
command_user="example:example"
pidfile="/run/${RC_SVCNAME}.pid"

depend() {
    need net
    after firewall
}
```

Сохраните файл как `/etc/init.d/example`, затем назначьте права и проверьте запуск.

```bash
sudo chmod +x /etc/init.d/example
sudo rc-service example status
sudo rc-service example start
sudo rc-service example status
sudo rc-update add example default
```

`command_user` запускает приложение от непривилегированного пользователя. Директория для PID и каталоги данных должны быть ему доступны.

## Основные команды

| Действие | Команда | Права |
|---|---|---|
| Состояние всех служб | `rc-status` | Обычно root для полного вывода |
| Подробный статус | `rc-status --servicelist` | Root или доступные сведения |
| Состояние службы | `rc-service nginx status` | Обычно root |
| Запуск | `rc-service nginx start` | Root |
| Остановка | `rc-service nginx stop` | Root |
| Перезапуск | `rc-service nginx restart` | Root |
| Перечитать конфигурацию | `rc-service nginx reload` | Root, если скрипт поддерживает |
| Добавить в уровень | `rc-update add nginx default` | Root |
| Удалить из уровня | `rc-update del nginx default` | Root |
| Список уровней | `rc-update show` | Обычно root |
| Службы уровня | `rc-status --runlevel default` | Root или доступные сведения |
| Ручной запуск скрипта | `rc-service nginx start` | Root |

Уровни обычно включают `sysinit`, `boot`, `default`, `nonetwork`, `shutdown` и `reboot`, но набор зависит от системы. Узнавайте доступные уровни через `rc-update show`.

## Управление от обычного пользователя

Обычный пользователь может запускать собственное приложение напрямую, через cron пользователя или отдельный пользовательский supervisor. Это не делает приложение OpenRC-службой.

```bash
nohup "$HOME/bin/example-worker" >"$HOME/example.log" 2>&1 &
echo $!
```

Проверьте процесс и завершите его по PID.

```bash
pgrep -af example-worker
kill 1234
```

Не используйте `sudo` для `nohup` без понимания владельца процесса и окружения. Если приложение должно переживать выход пользователя и автоматически перезапускаться, оформите системную OpenRC-службу с `command_user`.

## Логи и диагностика

OpenRC запускает сервисы, но формат логирования зависит от службы и системы. Ищите вывод в journald, syslog, `logread` на Alpine или файлах приложения.

```bash
sudo rc-service example status
sudo tail -n 100 /var/log/messages
sudo logread | tail -n 100
ps aux | grep '[e]xample-server'
```

`logread` доступен не на всех системах. Сначала проверьте `command -v logread`. Для ошибок запуска изучите сам init-скрипт и переменные `command`, `command_args`, `command_user`, `pidfile` и зависимости `depend()`.

## Частые ошибки

| Ошибка | Причина | Решение |
|---|---|---|
| `command not found: rc-service` | OpenRC не установлен или система использует другую init-систему | Проверьте `ps -p 1 -o comm=` и установку дистрибутива |
| `Permission denied` | Команда управления выполняется без root | Используйте `sudo` только для нужной операции |
| `service does not exist` | Нет скрипта в `/etc/init.d` | Проверьте имя и установите пакет службы |
| Служба стартует и сразу завершается | Ошибка команды или PID-файла | Запустите приложение вручную от `command_user`, проверьте логи и `pidfile` |
| `depend()` не работает ожидаемо | Ошибка в названиях зависимостей или уровне запуска | Проверьте доступные службы и `rc-update show` |
| Нет лога в ожидаемом файле | Логирование настроено syslog, logread или приложением | Проверьте конфигурацию logger и вывод команды вручную |

## Документация

- [OpenRC repository](https://github.com/OpenRC/openrc)
- [OpenRC manual pages](https://github.com/OpenRC/openrc/tree/master/man)
- [Alpine OpenRC](https://wiki.alpinelinux.org/wiki/OpenRC)
- [Gentoo OpenRC](https://wiki.gentoo.org/wiki/OpenRC)

- [Администрирование серверов Linux](Linux-Server-Admin.md)
- [systemd](Systemd.md)
