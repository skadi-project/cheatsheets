# Сети, DNS и HTTP — шпаргалка

## Что это

Сетевая диагностика проверяет путь от имени хоста до ответа приложения. DNS сопоставляет имена с адресами, IP доставляет пакеты, TCP/UDP переносят данные, а HTTP описывает запросы и ответы. Примеры рассчитаны на Linux, macOS и Windows с доступными аналогами команд.

## Быстрый порядок диагностики

```bash
getent hosts example.com
ip addr
ip route
ping -c 4 example.com
curl -vI --connect-timeout 5 https://example.com
```

Если имя не разрешается, проверяйте DNS. Если имя разрешается, но нет ответа, проверяйте маршрут и firewall. Если TCP-соединение установлено, изучайте TLS и HTTP-код.

## IP, интерфейсы и маршруты

| Команда | Назначение | Пример |
|---|---|---|
| `ip addr` | Адреса интерфейсов | `ip -br addr` |
| `ip link` | Состояние интерфейсов | `ip link show` |
| `ip route` | Таблица маршрутов | `ip route get 1.1.1.1` |
| `ss` | Слушающие порты и соединения | `ss -lntup` |
| `ping` | Проверка ICMP-доступности | `ping -c 4 192.0.2.1` |
| `traceroute` | Путь через маршрутизаторы | `traceroute example.com` |
| `tracepath` | Путь и MTU | `tracepath example.com` |
| `nc` | Проверка TCP/UDP-порта | `nc -vz example.com 443` |

В Windows используйте `ipconfig`, `route print`, `netstat -ano`, `ping`, `tracert` и `Test-NetConnection -ComputerName example.com -Port 443`. ICMP может блокироваться, поэтому отсутствие ответа `ping` само по себе не доказывает недоступность HTTP.

## DNS

Основные записи DNS — `A` и `AAAA` для адресов, `CNAME` для псевдонима, `MX` для почты, `NS` для серверов зоны, `TXT` для текстовых политик и проверок. TTL задаёт время кэширования ответа.

```bash
dig example.com A
dig example.com AAAA
dig example.com MX
dig +short example.com
dig @1.1.1.1 example.com
host example.com
```

В Windows PowerShell используйте `Resolve-DnsName example.com -Type A`.

| Симптом | Проверка | Возможная причина |
|---|---|---|
| `NXDOMAIN` | `dig example.com` | Имя отсутствует в DNS-зоне |
| `SERVFAIL` | `dig @1.1.1.1 example.com` | Ошибка рекурсии, делегации или DNSSEC |
| Имя разрешается в разные адреса | `dig +short example.com` | Балансировка, CDN или несколько записей |
| Один DNS работает, другой нет | Сравнить `dig @server name` | Локальный resolver, firewall или кэш |

Не проверяйте DNS только из одного места. Авторитетный ответ и кэш рекурсивного сервера могут отличаться до окончания TTL.

## HTTP и HTTPS

Методы `GET` читают ресурс, `POST` создаёт или запускает действие, `PUT` заменяет ресурс, `PATCH` изменяет часть, `DELETE` удаляет. Коды `2xx` означают успех, `3xx` перенаправление, `4xx` ошибку запроса или доступа, `5xx` ошибку сервера или upstream.

```bash
curl -I https://example.com
curl -v -- https://example.com/health
curl -L -o page.html https://example.com
curl -X POST -H 'Content-Type: application/json' \
  -d '{"name":"Ada"}' https://api.example.com/users
curl -w '\nstatus=%{http_code} time=%{time_total}\n' -o /dev/null -s https://example.com
```

Полезные заголовки — `Host`, `Content-Type`, `Accept`, `Authorization`, `Location`, `Cache-Control`, `ETag` и `Content-Length`. Не передавайте реальные секреты в командной строке и не публикуйте вывод `-v`.

| Код | Значение | Что проверить |
|---:|---|---|
| `200` | Успешный ответ | Тело и заголовки |
| `301`, `302` | Перенаправление | `Location`, `curl -L` |
| `400` | Неверный запрос | JSON, параметры и заголовки |
| `401` | Нужна аутентификация | Токен или учётные данные |
| `403` | Доступ запрещён | Права, ACL, policy и URL |
| `404` | Ресурс не найден | Маршрут, host и регистр пути |
| `429` | Слишком много запросов | Лимит и заголовок `Retry-After` |
| `500` | Ошибка приложения | Логи приложения |
| `502` | Некорректный ответ upstream | Прокси и состояние upstream |
| `503` | Сервис недоступен | Состояние службы и healthcheck |
| `504` | Тайм-аут upstream | Сеть, тайм-ауты и нагрузка |

## TLS

```bash
curl -vI https://example.com
openssl s_client -connect example.com:443 -servername example.com </dev/null
```

Проверяйте имя сертификата, срок действия, цепочку доверия и время системы. Не отключайте проверку сертификата в рабочем клиенте как способ исправить ошибку.

## Частые ошибки

| Ошибка | Причина | Решение |
|---|---|---|
| `Could not resolve host` | DNS или имя хоста | Сравните `dig`, resolver и `/etc/resolv.conf` |
| `Connection timed out` | Пакеты блокируются или маршрут недоступен | Проверьте `ip route`, firewall и порт назначения |
| `Connection refused` | Хост доступен, служба не слушает порт | Проверьте `ss -lntup` на сервере |
| `No route to host` | Нет маршрута или firewall отклоняет трафик | Проверьте маршрут и сетевые правила |
| `SSL certificate problem` | Сертификат, цепочка или время неверны | Проверьте сертификат и системное время |

## Документация

- [MDN HTTP](https://developer.mozilla.org/en-US/docs/Web/HTTP)
- [IANA service names and port numbers](https://www.iana.org/assignments/service-names-port-numbers/service-names-port-numbers.xhtml)
- [RFC 1034 DNS concepts](https://www.rfc-editor.org/rfc/rfc1034)
- [curl documentation](https://curl.se/docs/)
