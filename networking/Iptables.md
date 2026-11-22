# iptables — шпаргалка

`iptables` управляет правилами IPv4, `ip6tables` управляет IPv6. В Debian/Ubuntu пакет называется `iptables`. Вывод `iptables --version` показывает бэкенд `nf_tables` или `legacy`.

## Просмотр

```bash
sudo iptables -L -n -v --line-numbers
```

```bash
sudo iptables -S
```

```bash
sudo iptables -t nat -S
```

В таблице `filter` цепочка `INPUT` обслуживает входящий трафик к хосту, `OUTPUT` исходящий, `FORWARD` пересылаемый. Правила проверяются по порядку. Если в базовой цепочке решение не принято, действует её политика.

## Ограничение новых SSH-подключений

В учебной машине с консольным доступом сначала сохраните правила. Замените `192.0.2.10` адресом доверенного клиента и проверьте порт SSH. Учитывайте отдельные правила IPv6.

```bash
sudo iptables-save > rules.v4
```

```bash
sudo iptables -I INPUT 1 -p tcp -s 192.0.2.10 --dport 22 -m conntrack --ctstate NEW -j ACCEPT
```

```bash
sudo iptables -I INPUT 2 -p tcp --dport 22 -m conntrack --ctstate NEW -j DROP
```

Проверьте вход в новой сессии. `-A` добавляет правило в конец, `-I` вставляет по позиции, `-C` проверяет наличие, `-D` удаляет. Перед удалением по номеру обновите вывод `-L`.

## Восстановление и сохранение

```bash
sudo iptables-restore --test < rules.v4
```

```bash
sudo iptables-restore < rules.v4
```

В Debian/Ubuntu `iptables-persistent` загружает `/etc/iptables/rules.v4` и `rules.v6`, если настроен соответствующий сервис. Для снимков IPv6 используйте `ip6tables-save`. Перед изменениями проверьте, кто управляет правилами на хосте с Docker или UFW.

| Симптом | Что проверить |
|---|---|
| Правило есть, трафик проходит | IPv4/IPv6, цепочку и более ранние правила |
| Счётчик равен нулю | Порт, адрес, интерфейс и направление |
| Правила исчезли после перезагрузки | Загрузку сохранённой конфигурации |

[Справка iptables](https://man7.org/linux/man-pages/man8/iptables.8.html) · [UFW](Ufw.md) · [nftables](Nftables.md)
