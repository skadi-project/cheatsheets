# Docker Compose — шпаргалка

## 1. Что это и зачем

Docker Compose описывает сервисы, сети и тома в YAML и управляет ими как одним проектом. Используйте его для локальной разработки и запуска нескольких сервисов на одном хосте. Для одного контейнера достаточно `docker run`, а для оркестрации на нескольких хостах подойдёт Kubernetes.

## 2. Установка и удаление

Используйте актуальный CLI с командой `docker compose`. Версию сверяйте по официальной странице установки.

### Windows 10/11 и macOS

Установите и запустите Docker Desktop по [шпаргалке Docker](Docker.md). Compose входит в Desktop и удаляется вместе с ним. На Windows через WSL можно установить Linux-плагин внутри дистрибутива с собственным Docker Engine. На macOS для альтернативной установки используйте Homebrew-плагин с уже настроенным Docker CLI и Engine.

```bash
brew install docker-compose
```

При необходимости добавьте путь к каталогу плагина по подсказке `brew info docker-compose`. Перед удалением Homebrew-плагина удалите добавленный путь из `cliPluginsExtraDirs`, если он больше не нужен.

```bash
brew uninstall docker-compose
```

### Ubuntu / Debian

Используйте официальный репозиторий Docker для своего дистрибутива. Альтернатива — ручная установка CLI-плагина из релизов GitHub в `~/.docker/cli-plugins/docker-compose` с правом на выполнение. Для удаления ручной установки удалите этот файл.

```bash
sudo apt install docker-compose-plugin
```

```bash
sudo apt remove docker-compose-plugin
```

### Arch Linux

```bash
sudo pacman -Syu docker-compose
```

```bash
sudo pacman -R docker-compose
```

Альтернатива — ручная установка плагина, как для Ubuntu/Debian. Команды с `sudo` меняют системные пакеты. `pacman -Syu` обновляет систему.

### Проверка на любой ОС

```bash
docker compose version
```

## 3. Особенности по ОС

- **Windows** — запускайте команды из каталога проекта. При работе через WSL храните исходники в файловой системе Linux, если важна скорость bind mount.
- **macOS** — проверьте наличие образов для `arm64` на Apple Silicon и доступ Docker Desktop к каталогам, которые монтируете.
- **Linux** — для запуска сервисов требуются работающий Docker Engine и доступ к нему. Используйте настроенный rootless-контекст или `sudo` при стандартной установке. При `sudo` может меняться пользовательская конфигурация CLI и набор переменных среды.

## 4. Быстрый старт

Создайте в отдельной папке файл `compose.yaml`.

```yaml
services:
  web:
    image: nginx:alpine
    ports:
      - "127.0.0.1:${WEB_PORT:-8080}:80"
```

```bash
docker compose config --quiet
```

```bash
docker compose up -d
```

```bash
docker compose ps
```

Откройте `http://localhost:8080` при настройках по умолчанию. Чтобы убрать учебный контейнер и сеть проекта, выполните команду из той же папки.

```bash
docker compose down
```

## 5. Основные команды

| Команда | Что делает | Пример |
|---|---|---|
| `config` | Собирает конфигурацию и подставляет переменные | `docker compose config` |
| `up -d` | Создаёт и запускает сервисы в фоне | `docker compose up -d` |
| `up --build` | Собирает локальные образы перед запуском | `docker compose up -d --build` |
| `up --scale` | Задаёт количество экземпляров сервиса | `docker compose up -d --scale worker=3` |
| `ps` | Показывает состояние контейнеров | `docker compose ps -a` |
| `ls` | Показывает проекты | `docker compose ls` |
| `logs` | Читает вывод сервиса | `docker compose logs --tail 50 -f web` |
| `exec` | Выполняет команду в работающем сервисе | `docker compose exec web sh` |
| `run --rm` | Запускает одноразовый контейнер сервиса | `docker compose run --rm web nginx -t` |
| `port` | Показывает адрес и порт хоста | `docker compose port web 80` |
| `top` | Показывает процессы | `docker compose top web` |
| `images` | Показывает образы контейнеров проекта | `docker compose images` |
| `pull` | Скачивает образы сервисов | `docker compose pull web` |
| `build` | Собирает образы сервисов с `build:` | `docker compose build` |
| `stop` | Останавливает контейнеры | `docker compose stop web` |
| `start` | Запускает существующие контейнеры | `docker compose start web` |
| `restart` | Перезапускает существующие контейнеры | `docker compose restart web` |
| `down` | Удаляет контейнеры и сети проекта | `docker compose down` |
| `-f` | Выбирает файл конфигурации | `docker compose -f compose.yaml config` |
| `-p` | Задаёт имя проекта | `docker compose -p training up -d` |

`worker` в примере масштабирования — имя сервиса другого проекта. Для нескольких экземпляров одного сервиса избегайте фиксированного порта хоста на каждом экземпляре.

## 6. Конфигурация

На всех ОС храните `compose.yaml` в каталоге проекта. Поле `version` можно опустить. Определяйте сервисы в `services`, а именованные тома и сети — в верхнеуровневых `volumes` и `networks`.

Минимальный `.env` рядом с `compose.yaml` для примера выше.

```dotenv
WEB_PORT=8081
```

Compose использует `.env` при подстановке значений. Чтобы передать переменные внутрь контейнера, задайте `environment` или `env_file` у сервиса. Для явного выбора файла подстановки используйте глобальный флаг `--env-file`.

```bash
docker compose --env-file .env config --quiet
```

## 7. Полезные приёмы

- После правки YAML используйте `up -d`, чтобы Compose применил изменения. `restart` перезапускает контейнеры с прежней конфигурацией.
- Сервисы общей сети обращаются друг к другу по имени сервиса и внутреннему порту, например `http://web:80`.
- Для ожидания готовности зависимости задавайте `healthcheck` и условие `depends_on: ... condition: service_healthy`.
- Используйте именованный том для данных базы. `down -v` удаляет тома проекта, кроме внешних, и может уничтожить данные.
- До публикации логов или вывода `docker compose config` проверьте, есть ли в них подставленные секреты.

## 8. Частые ошибки

| Ошибка | Причина | Решение |
|---|---|---|
| `docker: 'compose' is not a docker command` | CLI не видит плагин | Установите плагин или проверьте Docker Desktop |
| `no configuration file provided` | Compose не нашёл файл | Перейдите в каталог проекта или передайте `-f` |
| `port is already allocated` | Порт хоста занят другим процессом | Измените `WEB_PORT` и примените настройки через `up -d` |
| `service ... is not running` | Контейнер не запустился или завершился | Проверьте `ps -a` и `logs` |
| Приложение не подключается к базе | Неверный адрес или база ещё не готова | Проверьте имя сервиса, внутренний порт и `healthcheck` |

## 9. Документация

- [Установка Docker Compose](https://docs.docker.com/compose/install/)
- [Compose CLI](https://docs.docker.com/reference/cli/docker/compose/) и [спецификация](https://docs.docker.com/reference/compose-file/)
- [Репозиторий](https://github.com/docker/compose)
