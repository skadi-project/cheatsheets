# UFW — шпаргалка

UFW упрощает настройку файрвола Linux. В Debian/Ubuntu установите пакет `ufw` через APT. Команды управления требуют `sudo`.

## Начальная политика

```bash
sudo ufw status verbose
```

На новом хосте добавьте разрешение до включения файрвола. Подставьте адрес доверенного клиента вместо `192.0.2.10` и реальный SSH-порт вместо `22`. Держите текущую сессию открытой и проверьте повторный вход.

```bash
sudo ufw allow from 192.0.2.10 to any port 22 proto tcp
```

Если хост обслуживает HTTPS, откройте порт веб-службы.

```bash
sudo ufw allow 443/tcp
```

```bash
sudo ufw default deny incoming
```

```bash
sudo ufw default allow outgoing
```

```bash
sudo ufw enable
```

## Управление правилами

| Задача | Команда |
|---|---|
| Показать номера правил | `sudo ufw status numbered` |
| Предварительно просмотреть изменения | `sudo ufw --dry-run allow 443/tcp` |
| Удалить разрешение по спецификации | `sudo ufw delete allow 443/tcp` |
| Удалить правило по номеру | `sudo ufw delete 3` |
| Показать локальные профили | `sudo ufw app list` |
| Включить журналирование | `sudo ufw logging low` |
| Перечитать правила | `sudo ufw reload` |

После удаления по номеру заново выведите список. При включённом IPv6 общее правило может создавать две записи. Настройку IPv6 проверяйте в `/etc/default/ufw`. После `enable` правила сохраняются между перезагрузками.

| Симптом | Что проверить |
|---|---|
| `Status: inactive` | Состояние UFW и был ли выполнен `enable` |
| Сервис недоступен | Адрес и порт через `ss`, правило UFW и сетевые правила провайдера |
| Опубликованный порт Docker доступен | Правила Docker и путь пересылаемого трафика. Для локального доступа публикуйте порт на `127.0.0.1` |

[Ubuntu Server — Firewall](https://ubuntu.com/server/docs/how-to/security/firewalls/) · [iptables](Iptables.md) · [nftables](Nftables.md)
