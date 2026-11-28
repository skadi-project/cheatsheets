# Технические шпаргалки

Краткие справочники на русском языке с примерами команд, синтаксиса и типичных ошибок. Подборка охватывает Linux, командные оболочки, языки программирования, базы данных, контейнеры и Git.

## Содержание

| Раздел | Материалы |
|---|---|
| Linux | [Основные команды](linux/Linux-Commands.md), [виртуализация QEMU/KVM](linux/Linux-Virtualization.md), [dd](linux/Dd.md), [администрирование серверов](linux/Linux-Server-Admin.md), [systemd](linux/Systemd.md), [OpenRC](linux/OpenRC.md), [очистка диска](linux/Linux-Cleanup.md), [APT](linux/Apt.md), [pacman](linux/Pacman.md) |
| Командные оболочки | [POSIX sh](shells/Sh.md), [Bash](shells/Bash.md), [Fish](shells/Fish.md), [Zsh](shells/Zsh.md), [Windows Batch](shells/Batch.md), [PowerShell](shells/PowerShell.md) |
| Программирование | [C](programming/C.md), [C++](programming/Cpp.md), [Rust](programming/Rust.md), [Python](programming/Python.md) |
| Базы данных | [SQL](databases/SQL.md), [Oracle SQL](databases/Oracle-SQL.md), [PostgreSQL](databases/PostgreSQL.md) |
| Контейнеры | [Docker](containers/Docker.md), [Docker Compose](containers/Docker-Compose.md), [Minecraft-сервер в Docker](containers/Minecraft-Docker.md) |
| Сети и веб-серверы | [Сети, DNS и HTTP](networking/Networks-DNS-HTTP.md), [Nginx](networking/Nginx.md), [iptables](networking/Iptables.md), [UFW](networking/Ufw.md), [nftables](networking/Nftables.md), [Samba](networking/Samba.md) |
| Инструменты сборки | [GNU Make](build-tools/Make.md), [CMake](build-tools/CMake.md), [Cargo](build-tools/Cargo.md) |
| Контроль версий | [Git](version-control/Git.md) |
| Оформление | [Шаблон для новых шпаргалок](docs/PROMT.md) |

## Дерево репозитория

```text
.
├── README.md
├── build-tools/
│   ├── CMake.md
│   ├── Cargo.md
│   └── Make.md
├── containers/
│   ├── Docker-Compose.md
│   ├── Docker.md
│   └── Minecraft-Docker.md
├── databases/
│   ├── Oracle-SQL.md
│   ├── PostgreSQL.md
│   └── SQL.md
├── docs/
│   └── PROMT.md
├── linux/
│   ├── Apt.md
│   ├── Dd.md
│   ├── Linux-Cleanup.md
│   ├── Linux-Commands.md
│   ├── Linux-Server-Admin.md
│   ├── OpenRC.md
│   ├── Pacman.md
│   ├── Linux-Virtualization.md
│   └── Systemd.md
├── networking/
│   ├── Iptables.md
│   ├── Networks-DNS-HTTP.md
│   ├── Nftables.md
│   ├── Nginx.md
│   ├── Samba.md
│   └── Ufw.md
├── programming/
│   ├── C.md
│   ├── Cpp.md
│   ├── Python.md
│   └── Rust.md
├── shells/
│   ├── Bash.md
│   ├── Batch.md
│   ├── Fish.md
│   ├── PowerShell.md
│   ├── Sh.md
│   └── Zsh.md
└── version-control/
    └── Git.md
```
