# Samba: шпаргалка по настройке и подключению

*Сервер: Linux (Ubuntu/Debian, Arch). Клиенты: Windows 10/11, Linux, macOS, iOS, Android. Уровень: новичок. Версия: Samba 4.x (⚠️ актуальную версию сверьте на samba.org).*

## 1. Что это и зачем

Samba — реализация протокола SMB/CIFS для Linux. Она открывает папки по сети для Windows, macOS, Linux, iOS и Android. Подходит для домашнего файлового сервера и общих папок в локальной сети. **Не подходит** для доступа напрямую из интернета (порт 445 наружу открывать нельзя) и для синхронизации файлов между устройствами. Альтернативы: SFTP/SSH (проще и безопаснее для одного пользователя), NFS (Linux/macOS), Syncthing или Nextcloud (синхронизация).

> Samba ставится на **сервер (Linux)**. На Windows и macOS клиент SMB уже встроен. Чтобы раздать папку с них, используйте встроенный общий доступ (Windows: свойства папки → «Доступ»; macOS: Настройки → Основные → Общий доступ → Общий доступ к файлам).

## 2. Установка сервера

### Ubuntu / Debian

```bash
sudo apt update
sudo apt install samba smbclient cifs-utils
sudo systemctl enable --now smbd nmbd
```

Проверка:

```bash
smbd --version
testparm -s
```

Удаление (⚠️ `purge` удаляет и конфиги, сделайте копию `/etc/samba/smb.conf`):

```bash
sudo apt remove --purge samba
```

### Arch

```bash
sudo pacman -S samba smbclient cifs-utils
sudo systemctl enable --now smb nmb
```

⚠️ В Arch **нет готового** `/etc/samba/smb.conf`. Создайте его (см. раздел 6), иначе служба не запустится.

Удаление:

```bash
sudo pacman -Rns samba
```

### Другие способы

Docker-образы Samba существуют, но новичку проще пакет из репозитория (⚠️ проверять документацию конкретного образа). Сборка из исходников описана на wiki.samba.org, для обычных задач не нужна.

### Windows / macOS

Устанавливать ничего не нужно, клиент SMB встроен. PATH на Linux настраивать не нужно: бинарники лежат в `/usr/bin` и `/usr/sbin`. ⚠️ Не включайте SMB1 в Windows: протокол устарел и небезопасен.

## 3. Особенности и подводные камни

**Linux-сервер**

- Действуют **два уровня прав**: права Linux на папку и настройки шары в `smb.conf`. Доступ есть, только если разрешено обоими.
- Пользователь Samba должен существовать в Linux, а пароль для Samba задаётся отдельно (`smbpasswd`).
- Файрвол: порт 445/TCP (`sudo ufw allow samba` или firewalld).
- SELinux (Fedora/RHEL; в Ubuntu и Arch по умолчанию не включён): нужна метка `samba_share_t` на папке.
- После правки `smb.conf` перезапускайте службу.
- Шара на NTFS/exFAT-разделе: монтируйте раздел с опциями `uid=` и `gid=`.
- WSL2: сеть за NAT, шару оттуда раздавать неудобно (⚠️ проверить в документации WSL). Лучше нативный Linux или виртуальная машина с мостовой сетью.

**Windows**

- Гостевой доступ без пароля в Windows 10/11 по умолчанию заблокирован. Используйте логин и пароль.
- Windows кэширует учётные данные: при ошибке 1219 сбросьте подключения командой `net use * /delete`.
- Сервер может не показываться в разделе «Сеть» (старое обнаружение отключено). Подключайтесь по IP.
- Windows 11 24H2 строже к гостевому доступу и подписи SMB (⚠️ проверить в документации Microsoft).

**macOS**

- Intel и Apple Silicon ведут себя одинаково. Finder создаёт на шаре служебные файлы `.DS_Store` и `._*`, это нормально.

**iOS / Android**

- Работают по IP в той же Wi-Fi сети. Гостевой Wi-Fi с изоляцией клиентов и включённый VPN блокируют доступ.
- На Android приложение «Files» от Google SMB не поддерживает, нужен другой файловый менеджер.

## 4. Быстрый старт (5 минут)

Замените `$USER` на своё имя, если делаете от другого пользователя. На Arch сначала создайте `smb.conf` по разделу 6.

```bash
# 1. Папка для шары
sudo mkdir -p /srv/share
sudo chown $USER:$USER /srv/share

# 2. Резервная копия и добавление шары в конец конфига
sudo cp /etc/samba/smb.conf /etc/samba/smb.conf.bak
sudo tee -a /etc/samba/smb.conf > /dev/null <<EOF

[share]
   path = /srv/share
   read only = no
   valid users = $USER
EOF

# 3. Проверка конфига
testparm -s

# 4. Пароль Samba (отдельный от системного)
sudo smbpasswd -a $USER

# 5. Перезапуск (на Arch: smb)
sudo systemctl restart smbd

# 6. Файрвол, если включён ufw
sudo ufw allow samba

# 7. Локальный тест и узнать IP
smbclient -L localhost -U $USER
hostname -I
```

Подключение с другого устройства: `\\IP_сервера\share` (Windows) или `smb://IP_сервера/share` (остальные). Подробно в разделе 7.

## 5. Основные команды

### Сервер

| Команда | Что делает | Пример |
| --- | --- | --- |
| `testparm` | Проверяет `smb.conf` на ошибки | `testparm -s` |
| `smbpasswd -a` | Добавляет пользователя Samba | `sudo smbpasswd -a alice` |
| `smbpasswd -d` / `-e` | Отключает / включает пользователя | `sudo smbpasswd -d alice` |
| `smbpasswd -x` | Удаляет пользователя из Samba | `sudo smbpasswd -x alice` |
| `pdbedit -L` | Список пользователей Samba | `sudo pdbedit -L` |
| `smbstatus` | Кто подключён, какие файлы открыты | `sudo smbstatus` |
| `systemctl restart` | Перезапуск после правки конфига | `sudo systemctl restart smbd` |
| `journalctl -u` | Логи службы | `sudo journalctl -u smbd -e` |

### Клиент Linux / macOS

| Команда | Что делает | Пример |
| --- | --- | --- |
| `smbclient -L` | Список шар на сервере | `smbclient -L //192.168.1.10 -U alice` |
| `smbclient //host/share` | Консольный клиент (`ls`, `get`, `put`) | `smbclient //192.168.1.10/share -U alice` |
| `mount -t cifs` | Монтирует шару (Linux) | `sudo mount -t cifs //192.168.1.10/share /mnt/smb -o username=alice,uid=$(id -u),gid=$(id -g)` |
| `umount` | Отмонтирует | `sudo umount /mnt/smb` |
| `mount_smbfs` | Монтирует шару (macOS) | `mount_smbfs //alice@192.168.1.10/share ~/smb` |
| `open smb://` | Открывает шару в Finder (macOS) | `open smb://192.168.1.10/share` |

### Клиент Windows (cmd / PowerShell)

| Команда | Что делает | Пример |
| --- | --- | --- |
| `net use` | Подключает сетевой диск, `*` запросит пароль | `net use Z: \\192.168.1.10\share /user:alice * /persistent:yes` |
| `net use` | Показывает текущие подключения | `net use` |
| `net use /delete` | Отключает диск | `net use Z: /delete` |
| `net use * /delete` | Сбрасывает все подключения и кэш учёток | `net use * /delete` |

## 6. Конфигурация

| Где | Путь |
| --- | --- |
| Конфиг сервера (Ubuntu/Debian/Arch) | `/etc/samba/smb.conf` |
| Логи | `/var/log/samba/` и `journalctl -u smbd` |
| Клиенты Windows / macOS / iOS / Android | Конфиг не нужен, учётные данные хранит система или приложение |
| Клиент Linux (автомонтирование) | `/etc/fstab` и файл с логином и паролем |

**Минимальный рабочий конфиг** (для Arch целиком, в Ubuntu/Debian `[global]` уже есть):

```ini
[global]
   workgroup = WORKGROUP
   server string = Samba Server
   security = user
   server min protocol = SMB2

[share]
   path = /srv/share
   browseable = yes
   read only = no
   valid users = alice
   create mask = 0664
   directory mask = 0775
```

**Важнейшие параметры шары**

| Параметр | Назначение |
| --- | --- |
| `path` | Папка на сервере |
| `valid users` | Кто имеет доступ (`alice` или `@группа`) |
| `read only` | `no` разрешает запись |
| `browseable` | `no` скрывает шару из списка (имя нужно знать) |
| `guest ok` | `yes` разрешает вход без пароля |
| `force user` / `force group` | Все файлы создаются от этого пользователя / группы |
| `create mask` / `directory mask` | Права новых файлов / папок |
| `hosts allow` | Ограничение по IP, например `192.168.1. 127.0.0.1` |

**Гостевая шара** (⚠️ только в доверенной сети, Windows 10/11 такой вход по умолчанию блокирует):

```ini
[global]
   map to guest = Bad User

[public]
   path = /srv/public
   guest ok = yes
   read only = no
   force user = nobody
```

Папке `/srv/public` нужны права на запись для `nobody`, например `sudo chown nobody:nogroup /srv/public` (на Arch группа может называться иначе).

## 7. Подключение с разных устройств

Везде нужны: **IP сервера** (`hostname -I` на сервере), **имя шары** (в примерах `share`), **логин и пароль Samba**.

### Windows 10/11

1. `Win+R`, введите `\\192.168.1.10\share`, нажмите Enter.
2. Введите логин и пароль Samba, отметьте «Запомнить учётные данные».
3. Чтобы получить диск с буквой: Проводник → «Этот компьютер» → «Подключить сетевой диск» (в Windows 11 пункт в меню «…»), укажите тот же путь.

Логотип: если имя пользователя Windows и пароль совпадают с Samba, вход произойдёт автоматически. Иначе укажите логин вручную, при необходимости в виде `192.168.1.10\alice`.

### Linux

**Файловый менеджер:** `Ctrl+L` и введите `smb://192.168.1.10/share` (Nautilus: «Другие места» → «Подключиться к серверу»; Dolphin и Thunar тоже понимают `smb://`).

**Консоль** (нужен `cifs-utils`):

```bash
sudo mkdir -p /mnt/smb
sudo mount -t cifs //192.168.1.10/share /mnt/smb -o username=alice,uid=$(id -u),gid=$(id -g)
```

**Автомонтирование при загрузке.** Сначала файл с учётными данными:

```bash
nano ~/.smbcred
# внутри:
# username=alice
# password=ВАШ_ПАРОЛЬ
chmod 600 ~/.smbcred
```

Затем строка в `/etc/fstab` (⚠️ ошибка в этом файле может помешать загрузке, поэтому `nofail`; пути и `uid` подставьте свои):

```text
//192.168.1.10/share  /mnt/smb  cifs  credentials=/home/alice/.smbcred,uid=1000,gid=1000,iocharset=utf8,_netdev,nofail  0  0
```

Проверка без перезагрузки: `sudo mount -a`.

### macOS (Intel и Apple Silicon)

Finder → меню «Переход» → «Подключиться к серверу» (`Cmd+K`) → `smb://192.168.1.10/share` → логин и пароль. Для автоподключения при входе добавьте шару в «Объекты входа» (Настройки → Основные).

### iPhone / iPad (iOS, iPadOS)

1. Приложение «Файлы» → вкладка «Обзор» → кнопка «⋯» → «Подключиться к серверу».
2. Введите `smb://192.168.1.10` (или сразу `smb://192.168.1.10/share`) и нажмите «Далее».
3. Выберите «Зарегистрированный пользователь», введите логин и пароль Samba.
4. Сервер появится в «Обзор» в разделе «Общие».

⚠️ Названия пунктов меню зависят от версии iOS.

### Android

Приложение «Files» от Google SMB не поддерживает. Подойдёт один из менеджеров с поддержкой SMB: Material Files (открытый код), Solid Explorer, CX File Explorer, FX File Explorer, Total Commander с плагином LAN (⚠️ набор функций и интерфейс зависят от приложения и версии). В Samsung «Мои файлы» есть пункт «Сетевое хранилище» (⚠️ проверить на вашей прошивке). Для просмотра видео и музыки подойдёт VLC.

Общий порядок: «Добавить сетевое хранилище» → тип **SMB/Windows** → поля:

- Host: `192.168.1.10`
- Port: `445`
- Share (путь): `share`
- Domain/Workgroup: пусто или `WORKGROUP`
- Логин и пароль Samba

## 8. Полезные приёмы

- **Зафиксируйте IP сервера** (резервация DHCP на роутере), иначе подключения «потеряются» после смены адреса.
- **Общая папка для нескольких пользователей** через группу. ⚠️ Рекурсивные `chgrp` и `chmod` применяйте только к своей папке шары:

```bash
sudo groupadd smbusers
sudo usermod -aG smbusers alice
sudo chgrp -R smbusers /srv/share
sudo chmod -R g+rwX /srv/share
sudo chmod g+s /srv/share
```

В шаре укажите: `valid users = @smbusers`, `force group = smbusers`, `create mask = 0660`, `directory mask = 0770`.

Ещё советы:

- **Ограничьте доступ по сети:** в `[global]` добавьте `hosts allow = 192.168.1. 127.0.0.1` и `hosts deny = 0.0.0.0/0`.
- **Никогда не используйте `chmod 777`** на шаре: корректные права решают проблему безопаснее.
- **Доступ из интернета только через VPN** (WireGuard, Tailscale и т.п.), порт 445 не пробрасывайте.
- **Чтобы сервер был виден в «Сети» Windows**, часто ставят службу `wsdd` (⚠️ название пакета зависит от дистрибутива и версии, проверьте в репозитории).
- **Диагностика:** временно добавьте `log level = 2` в `[global]`, смотрите `/var/log/samba/`, затем уберите.
- **Проверить, слушает ли сервер порт:** `sudo ss -tlnp | grep 445`.
- **Временно закрыть доступ пользователю:** `sudo smbpasswd -d alice`, не удаляя его.
- **Time Machine и macOS:** используется модуль `vfs_fruit` (`vfs objects = fruit streams_xattr`), ⚠️ проверить в `man vfs_fruit`.

## 9. Частые ошибки и их решение

| Ошибка | Причина | Решение |
| --- | --- | --- |
| `NT_STATUS_LOGON_FAILURE` | Неверный пароль или пользователь не добавлен в Samba | `sudo smbpasswd -a пользователь` |
| `NT_STATUS_ACCESS_DENIED`, «Отказано в доступе» | Нет прав Linux на папку или пользователь не в `valid users` | Проверьте `ls -ld /srv/share`, `valid users`, права и владельца |
| `NT_STATUS_BAD_NETWORK_NAME` | Неверное имя шары | Имя в `[скобках]` конфига, список: `smbclient -L //IP -U user` |
| Windows: `0x80070035` «Не найден сетевой путь» | Служба не запущена, закрыт порт 445 или неверный IP | `systemctl status smbd`, `sudo ss -tlnp` и проверка файрвола, `ping` |
| Windows: «Организация блокирует гостевой доступ» | Windows запрещает вход без пароля | Задайте пользователя и пароль (`smbpasswd -a`) и входите с ними |
| Windows: ошибка 1219 | Уже есть подключение к серверу под другой учёткой | `net use * /delete`, затем подключитесь заново |
| Шара только для чтения | Не указано `read only = no` или нет прав на запись | Добавьте параметр и проверьте права папки |
| Не виден в «Сети» Windows | Отключено обнаружение по NetBIOS | Подключайтесь по IP или поставьте `wsdd` |
| Служба не стартует, Arch | Нет `/etc/samba/smb.conf` | Создайте файл по разделу 6, затем `testparm` |
| Изменения в конфиге не работают | Служба не перезапущена или ошибка в конфиге | `testparm`, затем `sudo systemctl restart smbd` |
| `mount error(13): Permission denied` | Неверный логин/пароль или версия протокола | Проверьте учётку, добавьте `vers=3.0` в опции монтирования |
| `cifs filesystem not supported` | Не установлен `cifs-utils` или нет модуля ядра | Установите `cifs-utils`, перезагрузитесь после обновления ядра |
| Кракозябры в именах файлов | Неверная кодировка при монтировании | Добавьте `iocharset=utf8` |

## 10. Ссылки

- Сайт Samba: https://www.samba.org
- Wiki проекта: https://wiki.samba.org
- Справка по `smb.conf`: `man smb.conf` или https://www.samba.org/samba/docs/current/man-html/smb.conf.5.html
- Arch Wiki: https://wiki.archlinux.org/title/Samba

⚠️ Версии, названия пакетов и пункты меню в мобильных приложениях меняются, при расхождениях сверяйтесь с официальной документацией.
