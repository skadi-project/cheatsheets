# ZeroTier: сеть через CLI и Docker

> Для Windows 10/11, macOS Intel/Apple Silicon, Ubuntu/Debian и Arch.
> Версии и команды ZeroTier меняются. Перед рабочим внедрением сверяйте их с [официальной документацией](https://docs.zerotier.com/).

## 1. Что это и зачем

ZeroTier создаёт виртуальную сеть поверх Интернета. Компьютеры получают адреса, похожие на адреса в одной локальной сети, даже если находятся за разными роутерами. Это удобно для удалённого доступа, лабораторий и совместной игры без проброса порта на домашнем роутере.

Не используйте ZeroTier как замену полноценному VPN для всего трафика без понимания маршрутизации. Альтернативы: Tailscale и WireGuard.

## 2. Термины и схема

- **ZeroTier Central** — веб-панель и API управления сетями.
- **Network ID** — 16-значный идентификатор сети.
- **Member** — подключённое устройство.
- **Managed IP** — адрес устройства внутри ZeroTier-сети.
- **API token** — секрет для API. Не публикуйте его.

Схема:

```text
ZeroTier Central
       |
   Network ID
   /        \
PC-1       PC-2
 10.x.x.1   10.x.x.2
       \
   Minecraft server
```

## 3. Установка

### Windows 10/11

Рекомендуемый способ — официальный MSI-инсталлятор с [страницы загрузки](https://www.zerotier.com/download/). В PowerShell после установки:

```powershell
zerotier-cli -v
zerotier-cli info
```

Альтернатива — Chocolatey:

```powershell
choco install zerotier-one
```

Удаление: **Параметры → Приложения → Установленные приложения → ZeroTier One → Удалить**. Через Chocolatey:

```powershell
choco uninstall zerotier-one
```

Обычно `zerotier-cli.exe` добавляется в PATH. Если команда не найдена, перезапустите терминал или добавьте каталог установки ZeroTier в PATH.

### macOS Intel и Apple Silicon

Рекомендуемый способ — официальный `.pkg` с [страницы загрузки](https://www.zerotier.com/download/). Проверка:

```bash
zerotier-cli -v
sudo zerotier-cli info
```

Альтернатива — Homebrew:

```bash
brew install --cask zerotier-one
```

Удаление:

```bash
brew uninstall --cask zerotier-one
```

При проблемах Apple Silicon проверьте, что скачан актуальный универсальный пакет. Не запускайте Intel-бинарник через Rosetta без необходимости.

### Ubuntu/Debian

Официальный установочный скрипт удобен, но перед запуском проверьте его содержимое и источник:

```bash
curl -s https://install.zerotier.com | sudo bash
```

Проверка и запуск службы:

```bash
sudo zerotier-cli info
sudo systemctl enable --now zerotier-one
```

Удаление:

```bash
sudo apt remove zerotier-one
```

Альтернатива — скачать пакет `.deb` с официальной страницы и установить его через `apt`:

```bash
sudo apt install ./zerotier-one-<version>.deb
```

### Arch Linux

Пакет доступен в репозитории Arch:

```bash
sudo pacman -S zerotier-one
sudo systemctl enable --now zerotier-one
```

Проверка:

```bash
sudo zerotier-cli info
```

Удаление:

```bash
sudo pacman -R zerotier-one
```

## 4. Создать сеть через API

Самый безопасный путь для новичка — создать сеть в [ZeroTier Central](https://central.zerotier.com/), затем скопировать `Network ID`. CLI/API нужен для автоматизации.

Создайте API-токен в профиле ZeroTier Central. Не вставляйте токен прямо в команду: он попадёт в историю shell.

### Bash (Linux, macOS или Git Bash на Windows)

Введите токен по скрытому запросу. Не вставляйте его в строку команды.

```bash
read -rsp 'ZeroTier API token: ' ZEROTIER_API_TOKEN
printf '
'
```

Примеры ниже используют Classic Central API v1. Для New Central используйте соответствующую документацию или создайте сеть через веб-панель. После создания проверьте в Central, что настроен пул автоматической выдачи IPv4 и он не пересекается с домашними подсетями участников.

Создание сети через API:

```bash
curl -X POST "https://api.zerotier.com/api/v1/network" \
  -H "Authorization: token $ZEROTIER_API_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"config":{"name":"minecraft-lan","private":true}}'
```

В ответе найдите `id` и сохраните его как `NETWORK_ID`. Если API отвергло формат запроса, сверяйтесь с [API-документацией](https://docs.zerotier.com/api/).

## 5. Подключить компьютеры

На каждом компьютере выполните:

### Windows PowerShell

```powershell
zerotier-cli join NETWORK_ID
zerotier-cli listnetworks
```

### macOS/Linux

```bash
sudo zerotier-cli join NETWORK_ID
sudo zerotier-cli listnetworks
```

Статус `ACCESS_DENIED` означает: устройство вошло в сеть, но ещё не авторизовано владельцем.

### Авторизовать участников

Откройте ZeroTier Central → сеть → **Members** → включите **Auth** для нужного устройства. Не включайте неизвестные устройства.

Для автоматизации сначала получите список участников:

```bash
curl -H "Authorization: token $ZEROTIER_API_TOKEN" \
  "https://api.zerotier.com/api/v1/network/NETWORK_ID/member"
```

Затем обновите конкретного участника. Поля API могут меняться; сверяйте запрос с официальной документацией:

```bash
curl -X POST "https://api.zerotier.com/api/v1/network/NETWORK_ID/member/MEMBER_ID" \
  -H "Authorization: token $ZEROTIER_API_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"config":{"authorized":true}}'
```

После работы удалите токен из текущей оболочки:

```bash
unset ZEROTIER_API_TOKEN
```

## 6. Проверка сети и диагностика

```bash
sudo zerotier-cli info
sudo zerotier-cli listnetworks
sudo zerotier-cli -j listnetworks
sudo zerotier-cli listpeers
```

На Windows уберите `sudo`.

Ищите в `listnetworks` состояние `OK` и managed IP. Проверка связи:

```bash
ping ZEROTIER_IP
```

Проверка маршрута:

```bash
# Linux
ip route

# macOS
route -n get ZEROTIER_IP

# Windows
route print
```

Если `ping` запрещён firewall, это не всегда означает, что TCP-сервис недоступен. Проверяйте конкретный порт.

## 7. ZeroTier в Docker

Контейнеру нужен доступ к TUN-устройству и сетевым capability. Пример для Linux-хоста:

```yaml
services:
  zerotier:
    image: zerotier/zerotier:latest
    container_name: zerotier
    restart: unless-stopped
    devices:
      - /dev/net/tun:/dev/net/tun
    cap_add:
      - NET_ADMIN
      - SYS_ADMIN
    volumes:
      - zerotier-data:/var/lib/zerotier-one

volumes:
  zerotier-data:
```

Запуск и подключение:

```bash
docker compose up -d
docker exec zerotier zerotier-cli info
docker exec zerotier zerotier-cli join NETWORK_ID
docker exec zerotier zerotier-cli listnetworks
```

⚠️ Не удаляйте `zerotier-data`, если хотите сохранить identity контейнера. Команда ниже удалит данные сети и потребует повторной авторизации:

```bash
docker compose down -v
```

На Docker Desktop для Windows/macOS сетевое поведение отличается от Linux. Для игрового сервера проще запускать ZeroTier-клиент на хосте, а Minecraft — в контейнере.

## 8. Minecraft Java Edition через Docker

Для этого примера запустите ZeroTier **на хосте**, не в отдельном контейнере из предыдущего раздела: его виртуальный интерфейс находится внутри контейнера. Сначала авторизуйте хост, затем укажите его фактический ZeroTier IPv4 в `.env` рядом с `compose.yml`:

```dotenv
ZEROTIER_IP=10.147.20.5
```

`EULA: "TRUE"` указывайте только после принятия [Minecraft EULA](https://www.minecraft.net/eula). Образ `itzg` поддерживается сообществом.

```yaml
services:
  minecraft:
    image: itzg/minecraft-server:latest
    container_name: minecraft
    tty: true
    stdin_open: true
    ports:
      - "${ZEROTIER_IP:?Set the host ZeroTier IPv4 in .env}:25565:25565/tcp"
    environment:
      EULA: "TRUE"
      TYPE: "VANILLA"
      VERSION: "LATEST"
      MEMORY: "2G"
    volumes:
      - ./data:/data
    restart: unless-stopped
```

Запуск:

```bash
docker compose up -d

docker compose logs -f minecraft
```

На компьютере-хосте найдите ZeroTier-IP:

```bash
sudo zerotier-cli listnetworks
```

Подключение клиента Minecraft:

```text
Multiplayer → Add Server → Server Address: ZEROTIER_IP:25565
```

Например:

```text
10.147.20.5:25565
```

Если используется уже запущенный одиночный мир, откройте его через **Open to LAN**. Minecraft покажет случайный порт. Игроки подключаются по `ZEROTIER_IP:СЛУЧАЙНЫЙ_ПОРТ`, а не обязательно по `25565`.

Проверьте:

- все устройства авторизованы в одной ZeroTier-сети;
- Minecraft запущен;
- TCP-порт разрешён firewall;
- используется ZeroTier-IP хоста, а не обычный `192.168.x.x`;
- `server-ip` в `server.properties` обычно оставлен пустым, чтобы сервер слушал интерфейсы контейнера; адресом публикации управляет Compose.

Проверка порта с Linux/macOS:

```bash
nc -vz ZEROTIER_IP 25565
```

Проверка из PowerShell:

```powershell
Test-NetConnection ZEROTIER_IP -Port 25565
```

## 9. Файлы и конфигурация

| ОС | Данные ZeroTier |
|---|---|
| Windows | Обычно `C:\ProgramData\ZeroTier\One` |
| macOS | Обычно `/Library/Application Support/ZeroTier/One` |
| Linux | Обычно `/var/lib/zerotier-one` |
| Docker | Volume, смонтированный в `/var/lib/zerotier-one` |

Не редактируйте identity-файлы вручную без резервной копии. Для изменения сети используйте Central и `zerotier-cli`.

Полезные настройки:

- private network — участники требуют авторизации;
- managed IP — адреса внутри сети;
- flow rules — фильтрация трафика, если нужна;
- `server.properties` — параметры Minecraft-сервера;
- Docker volume — сохранение мира и данных сервера.

## 10. Полезные приёмы

1. Дайте сети понятное имя: `minecraft-lan`.
2. Храните `NETWORK_ID` в документации, API-токен — только в менеджере секретов или переменной окружения.
3. Для каждого компьютера записывайте hostname и ZeroTier-IP.
4. Сначала проверяйте `listnetworks`, потом `ping`, потом порт Minecraft.
5. Не публикуйте `25565` на роутере, если игроки подключаются через ZeroTier.
6. Закрепите DHCP/managed IP только при необходимости; лишняя фиксация усложняет сеть.
7. Обновляйте Docker image осознанно; для стабильной игры лучше зафиксировать версию вместо `latest`.
8. При смене Docker-хоста сохраните volume ZeroTier и Minecraft.
9. На Linux проверьте `ufw`/`firewalld`; на Windows — правило Windows Defender Firewall.
10. Удаляйте участников, которыми больше не пользуетесь.

## 11. Частые ошибки

| Ошибка | Причина | Решение |
|---|---|---|
| `ACCESS_DENIED` | Устройство не авторизовано | В Central включите `Auth` для участника |
| `NETWORK_NOT_FOUND` | Неверный `NETWORK_ID` | Скопируйте ID заново из Central; в CLI статус может называться `NOT_FOUND` |
| `zerotier-cli: command not found` | CLI не в PATH | Перезапустите терминал или добавьте каталог в PATH |
| Сеть имеет статус не `OK` | Служба остановлена или участник не подтверждён | Запустите службу, проверьте Central и `listnetworks` |
| Docker: нет `/dev/net/tun` | TUN не передан контейнеру | Добавьте `devices` и capability из примера |
| `ping` не проходит | Firewall блокирует ICMP или нет маршрута | Проверьте managed IP, firewall и `listpeers` |
| Minecraft `Connection refused` | Сервер не запущен или порт закрыт | Проверьте `docker compose logs`, порт `25565`, firewall |
| Minecraft не видит LAN-мир | Используется не тот адрес/порт | Используйте ZeroTier-IP и порт из `Open to LAN` |
| Работает в обычной LAN, но не через ZeroTier | Сервер привязан к другому интерфейсу | Оставьте `server-ip` пустым и разрешите трафик firewall |
| После пересоздания контейнера нужна новая авторизация | Удалён volume identity | Восстановите volume или повторно авторизуйте новый member |

## 12. Официальные ссылки

- [ZeroTier](https://www.zerotier.com/)
- [Документация ZeroTier](https://docs.zerotier.com/)
- [Загрузка ZeroTier](https://www.zerotier.com/download/)
- [ZeroTier Central](https://central.zerotier.com/)
- [ZeroTier API](https://docs.zerotier.com/api/)
- [Docker image ZeroTier](https://hub.docker.com/r/zerotier/zerotier)
- [Minecraft Java Server](https://www.minecraft.net/en-us/download/server)
- [itzg Minecraft Docker image](https://github.com/itzg/docker-minecraft-server)
