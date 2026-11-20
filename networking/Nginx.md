# Nginx — шпаргалка

## Что это и зачем

Nginx — веб-сервер и reverse proxy для статики, HTTP-запросов и проксирования приложений. Он подходит для TLS-терминации, раздачи файлов и балансировки. Для простого локального сервера может хватить встроенного сервера приложения.

## Установка

### Debian / Ubuntu

```bash
sudo apt update
sudo apt install nginx
sudo nginx -t
sudo systemctl enable --now nginx
```

### Arch Linux

```bash
sudo pacman -Syu nginx
sudo nginx -t
sudo systemctl enable --now nginx
```

### Windows и macOS

Используйте официальный архив Nginx или Docker. На macOS доступна формула Homebrew `brew install nginx`. На Windows для рабочей службы обычно удобнее Docker, WSL или отдельный Windows reverse proxy.

```bash
nginx -v
```

## Быстрый старт

```bash
curl -I http://localhost
sudo systemctl status nginx --no-pager
sudo nginx -t
```

Основной конфигурационный файл обычно `/etc/nginx/nginx.conf`. Сайты могут находиться в `/etc/nginx/sites-available` и `sites-enabled` на Debian/Ubuntu или в `/etc/nginx/conf.d` на других установках.

## Раздача статики

```nginx
server {
    listen 80;
    server_name example.com;
    root /srv/www/example;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

После изменения:

```bash
sudo nginx -t && sudo systemctl reload nginx
```

Каталог и файлы должны быть доступны пользователю, под которым работает Nginx. Проверьте права всех родительских каталогов через `namei -l /srv/www/example`.

## Reverse proxy

```nginx
server {
    listen 80;
    server_name app.example.com;

    location / {
        proxy_pass http://127.0.0.1:3000;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

В Docker Compose адресом upstream обычно является имя сервиса и внутренний порт, например `http://app:3000`, а не `localhost`.

### Локальный пример в Docker Compose

Создайте `compose.yaml` в пустой папке. Сервис `app` отдаёт стандартную страницу Nginx, а `proxy` передаёт ему запросы.

```yaml
services:
  app:
    image: nginx:alpine
  proxy:
    image: nginx:alpine
    depends_on:
      - app
    ports:
      - "127.0.0.1:8080:80"
    volumes:
      - ./proxy.conf:/etc/nginx/conf.d/default.conf:ro
```

Сохраните `proxy.conf` рядом с `compose.yaml`.

```nginx
server {
    listen 80;
    location / {
        proxy_pass http://app:80;
    }
}
```

```bash
docker compose up -d
```

```bash
docker compose exec proxy nginx -t
```

```bash
curl -I http://127.0.0.1:8080
```

Ожидаемый ответ — `200 OK`. При `502` проверьте состояние `app` через `docker compose ps` и журналы через `docker compose logs proxy app`. После проверки остановите учебный проект.

```bash
docker compose down
```

## Основные директивы

| Директива | Назначение | Пример |
|---|---|---|
| `listen` | Адрес и порт | `listen 80;` |
| `server_name` | Имена виртуального сервера | `server_name example.com;` |
| `root` | Корень статических файлов | `root /srv/www/example;` |
| `location` | Правила для URL | `location /api/ { ... }` |
| `try_files` | Проверка файлов и fallback | `try_files $uri =404;` |
| `proxy_pass` | Передача запроса upstream | `proxy_pass http://127.0.0.1:3000;` |
| `add_header` | Добавление заголовка | `add_header X-Content-Type-Options nosniff always;` |
| `client_max_body_size` | Максимальный размер тела | `client_max_body_size 10m;` |
| `access_log` / `error_log` | Файлы журналов | `error_log /var/log/nginx/error.log warn;` |

## TLS

Для рабочей системы используйте сертификат от доверенного CA и автоматическое обновление. После добавления сертификата проверьте конфигурацию и открытие HTTPS из внешней сети.

```nginx
server {
    listen 443 ssl;
    server_name example.com;
    ssl_certificate /etc/ssl/example/fullchain.pem;
    ssl_certificate_key /etc/ssl/example/privkey.pem;

    location / {
        proxy_pass http://127.0.0.1:3000;
    }
}
```

## Диагностика

```bash
sudo nginx -t
sudo journalctl -u nginx -n 100 --no-pager
sudo tail -n 100 /var/log/nginx/error.log
curl -vI http://127.0.0.1
ss -lntp | grep ':80\|:443'
```

| Ошибка | Причина | Решение |
|---|---|---|
| `nginx: [emerg]` | Ошибка синтаксиса или дубликат директивы | Исправьте файл по выводу `nginx -t` |
| `403 Forbidden` | Нет права чтения или отсутствует index | Проверьте `root`, `index` и права каталогов |
| `404 Not Found` | Файл или маршрут не найден | Проверьте `root`, `location` и `try_files` |
| `502 Bad Gateway` | Upstream не запущен или неверный адрес | Проверьте `ss`, имя сервиса и порт |
| `504 Gateway Timeout` | Upstream отвечает слишком долго | Проверьте приложение, сеть и тайм-ауты |
| `bind() failed` | Порт занят или нет прав | Найдите процесс через `ss` и проверьте порт |

## Документация

- [Официальная документация Nginx](https://nginx.org/en/docs/)
- [Nginx Beginner's Guide](https://nginx.org/en/docs/beginners_guide.html)
- [Сетевая диагностика](Networks-DNS-HTTP.md)
- [Docker Compose](../containers/Docker-Compose.md)
