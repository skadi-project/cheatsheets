# APT — шпаргалка

## 1. Что это и зачем

APT — семейство инструментов управления пакетами в Debian, Ubuntu и производных системах. Он скачивает пакеты из настроенных репозиториев, устанавливает зависимости и обновляет систему. В Arch Linux используйте `pacman`, а для изолированных приложений рассмотрите Flatpak или контейнер.

## 2. Установка

APT уже входит в Debian и Ubuntu. Проверка версии клиента не требует прав администратора.

```bash
apt --version
```

Вместо ручной установки APT обновляется как часть базовой системы. Не удаляйте пакет `apt` из рабочей системы. На Windows запускайте APT внутри WSL-дистрибутива, а не в PowerShell.

## 3. Особенности по ОС

- **Debian** — стабильность важнее самых новых версий. Для сторонних репозиториев проверяйте совместимость с выпуском Debian.
- **Ubuntu** — источники могут включать `main`, `universe`, `restricted` и `multiverse`. Компоненты зависят от редакции и выпуска.
- **WSL** — обновляйте индексы внутри конкретного дистрибутива. Окружение Windows и Linux имеют отдельные пакеты.
- `sudo apt update` обновляет индексы, но не сами пакеты. `sudo apt upgrade` устанавливает доступные обновления.

## 4. Быстрый старт

Обновите индекс, найдите пакет `curl`, установите его, проверьте и удалите. Выполняйте удаление только если пакет больше не нужен.

```bash
sudo apt update
apt search '^curl$'
sudo apt install curl
curl --version
sudo apt remove curl
```

## 5. Основные команды

### Поиск и сведения

| Команда | Что делает | Пример |
|---|---|---|
| `apt search` | Ищет пакет по имени и описанию | `apt search nginx` |
| `apt show` | Показывает сведения о пакете | `apt show curl` |
| `apt policy` | Показывает версии и источники | `apt policy nginx` |
| `apt list --installed` | Показывает установленные пакеты | `apt list --installed` |
| `dpkg -l` | Показывает состояние пакетов dpkg | `dpkg -l \| grep nginx` |
| `apt-mark showhold` | Показывает удержанные пакеты | `apt-mark showhold` |

### Установка и удаление

| Команда | Что делает | Пример |
|---|---|---|
| `apt update` | Обновляет индексы репозиториев | `sudo apt update` |
| `apt install` | Устанавливает пакет и зависимости | `sudo apt install git` |
| `apt reinstall` | Переустанавливает пакет | `sudo apt reinstall ca-certificates` |
| `apt remove` | Удаляет пакет, оставляя конфигурацию | `sudo apt remove nginx` |
| `apt purge` | Удаляет пакет и системную конфигурацию | `sudo apt purge nginx` |
| `apt autoremove` | Удаляет ненужные зависимости | `sudo apt autoremove` |
| `apt download` | Скачивает `.deb` без установки | `apt download curl` |
| `dpkg -i` | Устанавливает локальный `.deb` | `sudo dpkg -i ./package.deb` |
| `apt install ./file.deb` | Устанавливает локальный `.deb` и разрешает зависимости | `sudo apt install ./package.deb` |

### Обновление

| Команда | Что делает | Пример |
|---|---|---|
| `apt upgrade` | Обновляет пакеты без удаления установленных | `sudo apt upgrade` |
| `apt full-upgrade` | Разрешает изменения зависимостей с удалением пакетов при необходимости | `sudo apt full-upgrade` |
| `apt-mark hold` | Удерживает пакет от автоматического обновления | `sudo apt-mark hold linux-image-generic` |
| `apt-mark unhold` | Снимает удержание | `sudo apt-mark unhold linux-image-generic` |
| `apt changelog` | Показывает журнал изменений | `apt changelog curl` |

## 6. Конфигурация

Репозитории находятся в `/etc/apt/sources.list` и файлах `/etc/apt/sources.list.d/*.list` или `*.sources`. Настройки APT — в `/etc/apt/apt.conf.d/`. Не смешивайте репозитории от разных выпусков Debian или Ubuntu без понимания зависимостей.

```bash
grep -Rhv '^\s*#' /etc/apt/sources.list /etc/apt/sources.list.d/ 2>/dev/null
```

Пример современной записи формата `.sources` требует заменить кодовое имя выпуска на своё.

```text
Types: deb
URIs: https://deb.debian.org/debian
Suites: bookworm
Components: main
Architectures: amd64
```

После изменения источников выполните `sudo apt update`. Добавляйте сторонний репозиторий только по официальной инструкции поставщика и проверяйте ключ подписи.

## 7. Полезные приёмы

- Перед крупным обновлением прочитайте список изменений и освободите место в `/boot` и корневой файловой системе.
- Используйте `apt-get` в автоматизированных сценариях, когда нужна стабильная CLI-интерфейсная форма. Для ручной работы подходит `apt`.
- `apt list --upgradable` показывает доступные обновления.
- После прерванной установки попробуйте `sudo dpkg --configure -a`, затем `sudo apt -f install`.
- Не добавляйте в `sources.list` репозитории от другого выпуска ради одной новой версии пакета.

## 8. Частые ошибки

| Ошибка | Причина | Решение |
|---|---|---|
| `Could not get lock` | Другая операция APT или dpkg уже выполняется | Дождитесь её завершения. Не удаляйте lock-файлы вручную |
| `Unable to locate package` | Индексы устарели или пакет отсутствует в источниках | Выполните `sudo apt update` и проверьте имя и репозитории |
| `404 Not Found` | Выпуск или зеркало больше не содержит пакет | Проверьте codename, зеркало и поддержку выпуска |
| `dpkg was interrupted` | Предыдущая установка завершилась аварийно | Выполните `sudo dpkg --configure -a` |
| `held broken packages` | Несовместимый набор версий или удержание пакета | Проверьте `apt policy`, удержания и источники |
| `NO_PUBKEY` | Не найдена подпись репозитория | Установите официальный ключ способом поставщика. Не отключайте проверку подписей |

## 9. Документация

- [Debian APT User Manual](https://www.debian.org/doc/manuals/apt-guide/)
- [Ubuntu — Managing packages](https://ubuntu.com/server/docs/package-management/)
- [Справка apt](https://manpages.debian.org/bookworm/apt/apt.8.en.html)
