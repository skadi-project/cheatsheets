# Minecraft Java Edition в Docker — шпаргалка

## 1. Подготовка

Нужен Docker с Linux-контейнерами и, для основного рецепта, плагин Compose. См. [Docker](Docker.md) и [Docker Compose](Docker-Compose.md). Java на хост устанавливать не нужно: она входит в образ `itzg/minecraft-server`. Это сервер **Java Edition**, не Bedrock. Для первого запуска нужен интернет.

Примеры используют Minecraft `1.21.1` и тег образа `java21`. Это рабочая комбинация для примера, не рекомендация по свежести версии. Для модов сначала выберите версию Minecraft и загрузчик по требованиям модов, затем тег Java по [документации образа](https://docker-minecraft-server.readthedocs.io/). Например, `1.20.1` обычно используют с Java 17, а `1.20.5–1.21.1` требуют Java 21. Для старых сборок и новых релизов сверяйте требования отдельно.

## 2. Быстрый запуск через `docker run`

В пустой папке создайте `data`. В Bash выполните `mkdir -p data`, в PowerShell — `New-Item -ItemType Directory -Force data`. Указывайте `EULA=TRUE`, только если принимаете [лицензию Minecraft](https://www.minecraft.net/eula).

```bash
docker run -d --name minecraft --restart unless-stopped -p 127.0.0.1:25565:25565/tcp --mount "type=bind,source=${PWD}/data,target=/data" -e EULA=TRUE -e TYPE=VANILLA -e VERSION=1.21.1 -e MEMORY=2G itzg/minecraft-server:java21
```

Однострочная команда также подходит для PowerShell. Если Git Bash на Windows преобразует `/data` в путь Windows, запустите команду из PowerShell или добавьте `MSYS_NO_PATHCONV=1` перед `docker` в Git Bash.

```bash
docker logs -f --tail 100 minecraft
```

Дождитесь строки `Done (...)!`, затем добавьте `localhost:25565` в меню **Сетевая игра** клиента Java Edition. `Ctrl+C` прекращает просмотр логов, не останавливая сервер. Для остановки используйте `docker stop -t 120 minecraft`, для повторного запуска — `docker start minecraft`. Для изменения параметров `docker run` остановите и удалите контейнер (`docker rm minecraft`), затем создайте заново с тем же `data`. Не запускайте одновременно Compose и `docker run` с одним миром.

## 3. Постоянный сервер через Compose

В отдельной папке создайте `.env`:

```dotenv
MC_JAVA_TAG=java21
MC_VERSION=1.21.1
MC_TYPE=VANILLA
MC_MEMORY=2G
MC_BIND_IP=127.0.0.1
MC_PORT=25565
```

Рядом создайте `compose.yaml`:

```yaml
services:
  mc:
    image: itzg/minecraft-server:${MC_JAVA_TAG}
    restart: unless-stopped
    stop_grace_period: 2m
    ports:
      - "${MC_BIND_IP}:${MC_PORT}:25565/tcp"
    environment:
      EULA: "TRUE"
      VERSION: "${MC_VERSION}"
      TYPE: "${MC_TYPE}"
      MEMORY: "${MC_MEMORY}"
      ONLINE_MODE: "TRUE"
      ENABLE_RCON: "TRUE"
    volumes:
      - ./data:/data
```

```bash
docker compose config --quiet
docker compose up -d
docker compose ps
docker compose logs -f --tail 100 mc
```

Compose создаст `data`, если папки ещё нет. В неё попадут мир, `server.properties`, `ops.json`, `whitelist.json`, моды, настройки и логи. Применяйте изменения `.env` или `compose.yaml` через `docker compose up -d`: `restart` не перечитывает конфигурацию контейнера. Для ручных изменений файлов в `data` сначала остановите сервер; переменные образа могут переопределять одноимённые настройки при старте.

## 4. Загрузчики и моды

| `MC_TYPE` | Когда использовать | Переменная в `environment` для фиксации загрузчика |
|---|---|---|
| `VANILLA` | Оригинальный сервер; поддерживает datapack, не загружает моды `.jar` | — |
| `FABRIC` | Моды Fabric; Fabric API добавляйте отдельно, если его требуют моды | `FABRIC_LOADER_VERSION` |
| `FORGE` | Моды и сборки Forge | `FORGE_VERSION` |
| `NEOFORGE` | Моды и сборки NeoForge | `NEOFORGE_VERSION` |

Для нового сервера смените `MC_TYPE` в `.env` и выполните `docker compose up -d`. Чтобы не выбирать версию загрузчика заново при чистой установке, добавьте соответствующую переменную в `environment`, указав конкретную совместимую версию. Не считайте моды Forge и NeoForge взаимозаменяемыми. Для смены загрузчика на существующем мире сделайте копию и пробуйте на отдельной папке данных и другом порту.

**Ручная установка.** Остановите сервер (`docker compose stop mc`), положите серверные и общие `.jar` в `data/mods/`, затем запустите `docker compose up -d` и проверьте логи. Сверьте версию Minecraft, загрузчик и зависимости каждого мода. Моды с новыми блоками, предметами или сетевым протоколом обычно нужны и на клиенте; чисто серверные ставят только на сервер, чисто клиентские — только в клиент. Не складывайте в `mods/` ZIP модпака, Fabric/Forge installer или Bukkit-плагины.

**Модпак.** Используйте отдельную пустую папку данных. Вместо строки `TYPE` в `environment` задайте платформу и источник сборки:

- **Modrinth:** `MODPACK_PLATFORM: MODRINTH`, `MODRINTH_MODPACK: /packs/pack.mrpack`. Положите файл в `./packs/pack.mrpack` и добавьте к списку `volumes` строку `- ./packs:/packs:ro`.
- **CurseForge:** `MODPACK_PLATFORM: AUTO_CURSEFORGE`, `CF_PAGE_URL` со ссылкой на страницу конкретного файла сборки (`…/files/<ID>`). Выберите основной файл модпака, а не дополнительный server pack. Передайте API-ключ `CF_API_KEY` через окружение (`CF_API_KEY: "${CF_API_KEY}"`), не храните его в Git. При запрете автоматической загрузки отдельных модов следуйте указаниям в логах и документации образа.

Для модпака подберите тег Java по версии сборки; строку `VERSION` из базового примера уберите, чтобы не задавать конфликтующую версию. На клиентах установите тот же релиз сборки. Не обновляйте её отдельными `.jar` без проверки совместимости.

**Datapack:** поместите ZIP или папку пакета в `data/world/datapacks/` (при другом `level-name` замените `world`). Перезапустите сервер и проверьте `datapack list` через RCON. **Resource pack:** меняет клиентские ресурсы, не добавляет серверную логику. Для распространения укажите прямой HTTPS URL ZIP в `resource-pack` и его SHA-1 в `resource-pack-sha1` в `server.properties`; перезапустите сервер.

## 5. Администрирование

Команды Minecraft выполняются без начального `/`. RCON доступен через `exec`: его порт не нужно публиковать. С `docker run` замените `docker compose exec mc` на `docker exec -it minecraft`.

```bash
docker compose exec mc rcon-cli
docker compose exec mc rcon-cli list
docker compose exec mc rcon-cli whitelist add PlayerName
docker compose exec mc rcon-cli whitelist on
docker compose exec mc rcon-cli op PlayerName
```

`op` выдаёт административные права, `deop PlayerName` отзывает их. `whitelist list` показывает список, `whitelist remove PlayerName` удаляет запись. Для частного сервера включите whitelist до открытия игрового порта. `save-all flush` принудительно сохраняет мир, но не делает копирование живого каталога безопасным. Останавливайте экземпляр через `docker compose stop mc`, а не через RCON `stop`, чтобы Docker не запустил его снова по политике рестарта.

`data/server.properties` хранит настройки мира и подключения. Для декларативного управления используйте переменные образа, например `MOTD`, `MAX_PLAYERS`, `DIFFICULTY`, `VIEW_DISTANCE`, `SIMULATION_DISTANCE`. `LEVEL` задаёт имя каталога мира; его смена не переименовывает существующий мир. Для контроля нагрузки используйте `docker stats` и логи. `MEMORY` задаёт размер Java heap, не полный расход контейнера. Если задаёте `mem_limit`, оставьте запас для JVM вне heap и проверьте доступную память Docker Desktop. Снижайте дальность прорисовки и симуляции, если сервер не успевает обрабатывать тики.

## 6. Доступ из сети

- `127.0.0.1` оставляет игровой порт локальным. Для LAN задайте в `MC_BIND_IP` LAN-адрес хоста либо `0.0.0.0` для всех IPv4-интерфейсов. Игрокам передайте **адрес хоста** и `MC_PORT`, а не IP контейнера.
- Для друзей вне LAN используйте VPN либо настройте переадресацию **только игрового TCP-порта** на маршрутизаторе. При CGNAT обычный проброс с интернета не работает. На Linux проверьте доступ с другого устройства: опубликованные Docker-порты могут обходить обычные правила UFW.
- Оставьте `ONLINE_MODE: "TRUE"`. Не публикуйте RCON (`25575/tcp`), Docker API или сокет Docker. Ограничивайте доступ к Docker и резервным копиям; моды запускают сторонний код, скачивайте их из доверенных источников.

## 7. Бэкап, обновление и откат

Холодный бэкап: остановите сервер, заархивируйте **весь** `data` вместе с конфигурацией и только потом запустите сервер. Команды ниже для Bash из папки Compose; выбирайте новое имя архива для каждой копии.

```bash
docker compose stop mc
tar -czf minecraft-before-update.tar.gz data compose.yaml .env
tar -tzf minecraft-before-update.tar.gz
docker compose up -d
```

Если архивирование не удалось, не начинайте обновление. Храните проверенную копию вне хоста. `.env` и `data/server.properties` могут содержать секреты; ограничьте доступ к архиву. В Windows используйте `tar.exe` либо Bash/WSL. Для сервера на `docker run` сохраните команду запуска вместо `compose.yaml` и `.env`.

Для обновления игры или модпака после бэкапа проверьте совместимость Java, загрузчика и модов. Измените нужные версии, выполните `docker compose up -d`, проверьте логи и вход в игру. `docker compose pull mc` обновляет **образ**, а не значение `MC_VERSION`. Теги вроде `java21` обновляемые: для точного воспроизведения сохраните digest текущего образа и при необходимости используйте ссылку `itzg/minecraft-server@sha256:…` в Compose.

Для отката остановите сервер. Переименуйте текущий `data` и сохраните текущие настройки, затем распакуйте проверенный архив в пустую папку. Восстановите из него `data`, `.env`, `compose.yaml` и прежний образ, запустите проект и проверьте мир, whitelist и моды. Не накатывайте старый архив поверх обновлённого каталога и не открывайте уже обновлённый мир старой версией игры.

## 8. Частые ошибки

| Симптом | Что проверить |
|---|---|
| Контейнер запущен, клиент не входит | `docker compose logs --tail 100 mc`: первый запуск может скачивать сервер и моды; дождитесь `Done`. Затем проверьте адрес привязки и версию клиента. |
| `port is already allocated` | Освободите порт или измените `MC_PORT`; внутренний порт оставьте `25565`. |
| `UnsupportedClassVersionError` | Сопоставьте Java-тег с требованиями Minecraft, загрузчика и мода. |
| `Missing mandatory dependencies` или ошибка handshake | Проверьте зависимости, загрузчик и соответствие общего набора модов на клиенте и сервере. |
| `OOMKilled` | Проверьте память хоста и Docker Desktop, лимит контейнера и heap; не задавайте heap равным лимиту. |
| `Permission denied` в `/data` | Проверьте доступ Docker к папке, владельца и настройки `UID`/`GID` образа на Linux. Не ставьте `chmod 777`. |
| Появился новый мир | Проверьте путь bind mount и `LEVEL`/`level-name`. Остановите сервер до исправления. |

## 9. Документация

- [Документация `itzg/minecraft-server`](https://docker-minecraft-server.readthedocs.io/): теги Java, загрузчики, модпаки и переменные окружения.
- [Репозиторий образа](https://github.com/itzg/docker-minecraft-server): релизы и примеры Compose.
- [Minecraft Java Edition Server](https://www.minecraft.net/download/server).
