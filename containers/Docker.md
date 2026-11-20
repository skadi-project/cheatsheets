# Docker — шпаргалка

## 1. Что это и зачем

Docker запускает приложения в контейнерах. Образ содержит файлы и инструкции запуска, контейнер — экземпляр образа, а том хранит данные независимо от контейнера. Для задач, требующих отдельного ядра ОС, используйте виртуальную машину. Альтернатива для контейнеров — Podman.

## 2. Установка и удаление

Используйте последний стабильный выпуск для поддерживаемой версии ОС. Доступность и требования сверяйте по ссылкам в разделе 9.

### Windows 10/11 — PowerShell

Включите аппаратную виртуализацию и настройте WSL 2 по требованиям Docker Desktop. Установите Docker Desktop через WinGet или официальный установщик, затем запустите приложение.

```powershell
winget install --id Docker.DockerDesktop -e
```

```powershell
docker version
```

```powershell
winget uninstall --id Docker.DockerDesktop -e
```

### macOS — Intel и Apple Silicon

Используйте Homebrew или официальный DMG для своей архитектуры. После установки откройте Docker Desktop.

```bash
brew install --cask docker-desktop
```

```bash
docker version
```

```bash
brew uninstall --cask docker-desktop
```

### Ubuntu / Debian

Настройте официальный репозиторий Docker для своего дистрибутива по разделу 9. Альтернатива — пакет `docker.io` из репозитория дистрибутива, версия которого может отличаться.

```bash
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

```bash
sudo docker version
```

```bash
sudo apt remove docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

### Arch Linux

```bash
sudo pacman -Syu docker
```

```bash
sudo systemctl enable --now docker.service
```

```bash
sudo docker version
```

```bash
sudo systemctl disable --now docker.service
```

```bash
sudo pacman -R docker
```

Альтернатива — установка статических бинарников по инструкции Docker, с ручной настройкой службы и обновлений. Перед удалением Docker сделайте копию нужных данных. Команда `pacman -Syu` обновляет систему.

## 3. Особенности по ОС

- **Windows** — для примеров включите Linux containers. При запуске из WSL включите интеграцию для своего дистрибутива.
- **macOS** — если CLI не найден, проверьте установку CLI в Docker Desktop и наличие выбранного каталога в `PATH`. На Apple Silicon предпочитайте образы с поддержкой `arm64`.
- **Linux с Docker Engine** — примеры предполагают доступ к демону. На стандартной установке используйте `sudo` или настройте rootless Docker. Членство в группе `docker` даёт права уровня root.

## 4. Быстрый старт

Опубликуйте контейнерный порт 80 на локальном порту 8080.

```bash
docker run -d --name cheat-web -p 127.0.0.1:8080:80 nginx:alpine
```

```bash
docker ps --filter name=cheat-web
```

Откройте `http://localhost:8080`. После проверки остановите и удалите учебный контейнер.

```bash
docker stop cheat-web
```

```bash
docker rm cheat-web
```

## 5. Основные команды

### Образы

| Команда | Что делает | Пример |
|---|---|---|
| `pull` | Скачивает образ | `docker pull nginx:alpine` |
| `image ls` | Показывает локальные образы | `docker image ls` |
| `build` | Собирает образ из контекста | `docker build -t cheat-site:1 .` |
| `tag` | Добавляет имя и тег образу | `docker tag cheat-site:1 cheat-site:local` |
| `image rm` | Удаляет локальный тег или образ | `docker image rm cheat-site:1` |

### Контейнеры, тома и сети

| Команда | Что делает | Пример |
|---|---|---|
| `run` | Создаёт и запускает контейнер | `docker run --rm nginx:alpine nginx -t` |
| `ps -a` | Показывает и остановленные контейнеры | `docker ps -a` |
| `logs` | Читает вывод контейнера | `docker logs --tail 50 -f cheat-web` |
| `exec` | Запускает команду внутри работающего контейнера | `docker exec -it cheat-web sh` |
| `inspect` | Показывает настройки контейнера | `docker inspect cheat-web` |
| `stats` | Показывает потребление ресурсов | `docker stats cheat-web` |
| `port` | Показывает опубликованные порты | `docker port cheat-web` |
| `cp` | Копирует файл между хостом и контейнером | `docker cp cheat-web:/etc/nginx/nginx.conf ./nginx.conf` |
| `stop` | Останавливает контейнер | `docker stop cheat-web` |
| `start` | Запускает уже созданный контейнер | `docker start cheat-web` |
| `rm` | Удаляет остановленный контейнер и его записываемый слой | `docker rm cheat-web` |
| `volume create` | Создаёт именованный том | `docker volume create cheat-data` |
| `volume ls` | Показывает тома | `docker volume ls` |
| `network create` | Создаёт сеть | `docker network create cheat-net` |
| `network ls` | Показывает сети | `docker network ls` |
| `system df` | Показывает использование диска | `docker system df` |
| `context ls` | Показывает доступные подключения к демонам | `docker context ls` |

## 6. Конфигурация и Dockerfile

Для обычного Docker Engine настройки демона находятся в `/etc/docker/daemon.json` на Linux и `%ProgramData%\docker\config\daemon.json` на Windows. В Docker Desktop на Windows и macOS используйте **Settings → Docker Engine**. Пример ограничивает размер файлов логов для новых контейнеров с драйвером `json-file`. Объедините параметры с существующими и примените конфигурацию через интерфейс Desktop или перезапуск Engine.

```json
{
  "log-driver": "json-file",
  "log-opts": { "max-size": "10m", "max-file": "3" }
}
```

Минимальный `Dockerfile` для каталога с `index.html`.

```dockerfile
FROM nginx:alpine
COPY index.html /usr/share/nginx/html/index.html
```

```bash
docker build -t cheat-site:1 .
```

## 7. Полезные приёмы

- Добавляйте `.git`, локальные зависимости и файлы секретов в `.dockerignore`.
- Для данных приложения используйте именованный том. Для исходников при разработке подойдёт bind mount.
- Используйте `--rm` для одноразовых команд, чтобы контейнер удалялся после выхода.
- Фиксируйте digest образа, когда нужна воспроизводимая сборка.
- Перед `docker system prune` проверьте, какие остановленные контейнеры и другие ресурсы ещё нужны. Опция `--volumes` расширяет очистку.

## 8. Частые ошибки

| Ошибка | Причина | Решение |
|---|---|---|
| `Cannot connect to the Docker daemon` | Desktop или служба не запущены либо выбран другой контекст | Проверьте службу и `docker context ls` |
| `permission denied` для сокета | CLI без доступа к демону | Используйте `sudo` для стандартного Linux Engine или настроенный rootless-контекст |
| `port is already allocated` | Порт хоста занят | Замените `8080` на свободный порт |
| `container name ... is already in use` | Контейнер с таким именем уже существует | Проверьте `docker ps -a`, затем выберите другое имя или удалите старый учебный контейнер |
| `no matching manifest` | Тег не поддерживает архитектуру | Выберите тег с нужной платформой |

## 9. Документация

- [Docker Desktop](https://docs.docker.com/desktop/)
- [Docker Engine для Ubuntu](https://docs.docker.com/engine/install/ubuntu/), [Debian](https://docs.docker.com/engine/install/debian/) и [статические бинарники](https://docs.docker.com/engine/install/binaries/)
- [Docker в ArchWiki](https://wiki.archlinux.org/title/Docker)
- [CLI](https://docs.docker.com/reference/cli/docker/) и [репозиторий](https://github.com/docker/cli)
