# nftables — шпаргалка

`nft` управляет правилами Netfilter. Семейство `inet` объединяет IPv4 и IPv6. В Debian/Ubuntu и Arch пакет называется `nftables`.

## Просмотр

```bash
sudo nft list ruleset
```

```bash
sudo nft -a list ruleset
```

Таблица содержит цепочки и наборы. Базовая цепочка регистрируется на `hook input`, `forward` или `output` и имеет приоритет. `-a` показывает идентификаторы правил (`handle`). Для удаления используйте актуальный идентификатор, например `sudo nft delete rule inet host_filter input handle 12`.

## Политика входящего трафика

Сохраните блок в `host-filter.nft`. Пример рассчитан на чистую учебную ВМ с консолью, без существующей таблицы `host_filter`. Проверьте порт SSH. Правило ниже разрешает TCP-порты 22 и 443 для обоих семейств адресов. Для маршрутизатора или Docker отдельно учитывайте пересылаемый трафик.

```nftables
table inet host_filter {
    chain input {
        type filter hook input priority filter; policy drop;
        iifname "lo" accept
        ct state invalid drop
        ct state established,related accept
        ip protocol icmp accept
        meta l4proto ipv6-icmp accept
        tcp dport { 22, 443 } counter accept
    }
}
```

```bash
sudo nft -c -f host-filter.nft
```

```bash
sudo nft -f host-filter.nft
```

`-c` проверяет конфигурацию без применения. После применения проверьте новый SSH-вход. Для отката учебного примера удалите добавленную таблицу через консоль.

```bash
sudo nft delete table inet host_filter
```

## Сохранение и диагностика

Снимок правил сохраняется в файл командой ниже. Повторная загрузка снимка поверх действующего набора может дублировать правила. Для постоянной конфигурации проверьте путь в выводе `systemctl cat nftables`. Часто используется `/etc/nftables.conf`.

```bash
sudo nft list ruleset > nft-backup.nft
```

| Симптом | Что проверить |
|---|---|
| `Operation not permitted` | Полномочия root или `CAP_NET_ADMIN` |
| Счётчик не растёт | Семейство, hook, приоритет и условия правила |
| Разрешённый трафик блокируется | Другую базовую цепочку с тем же hook, где может сработать `drop` |
| Правила пропадают после перезагрузки | Конфигурацию и статус службы `nftables` |

[Руководство nft](https://netfilter.org/projects/nftables/manpage.html) · [iptables](Iptables.md) · [UFW](Ufw.md)
