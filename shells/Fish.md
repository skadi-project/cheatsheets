# Fish — шпаргалка

## 1. Что это и зачем

Fish — интерактивная оболочка со встроенными подсказками и собственным скриптовым синтаксисом. Подходит для терминала и личной автоматизации. Для переносимых системных сценариев используйте [POSIX sh](Sh.md).

## 2. Установка

| ОС | Установка | Удаление |
|---|---|---|
| Ubuntu/Debian | `sudo apt install fish` | `sudo apt remove fish` |
| Arch | `sudo pacman -Syu fish` | `sudo pacman -R fish` |
| macOS | `brew install fish` или официальный установщик | `brew uninstall fish` для Homebrew |
| Windows | WSL и пакет внутри дистрибутива | Удаление пакета внутри WSL |

```bash
fish --version
```

Сверяйте версию с официальной страницей загрузки. `sudo` меняет системные пакеты, а `-Syu` обновляет систему. Если Fish был login-оболочкой, смените её перед удалением.

## 3. Особенности ОС

Пути на macOS зависят от префикса Homebrew. Fish можно запускать из текущего терминала без изменения login-оболочки. Команды настройки для Bash используйте в Bash, а скрипты Fish запускайте через `fish`.

## 4. Быстрый старт

Сохраните `hello.fish`.

```fish
#!/usr/bin/env fish
set -l name World
if set -q argv[1]
    set name "$argv[1]"
end
printf 'Hello, %s!\n' "$name"
```

```bash
fish hello.fish 'Ada Lovelace'
```

## 5. Основной синтаксис

| Конструкция | Назначение | Пример |
|---|---|---|
| `set -l` | Создаёт локальную переменную | `set -l name Ada` |
| `set -gx` | Экспортирует глобальную для текущего процесса переменную | `set -gx EDITOR vi` |
| `set -q` | Проверяет наличие переменной или элемента | `set -q argv[1]` |
| `$argv` | Аргументы скрипта или функции | `printf '%s\n' $argv` |
| `set` со списком | Хранит несколько элементов | `set -l files 'a b.txt' c.txt` |
| Индекс | Выбирает элемент, отсчёт с 1 | `printf '%s\n' $files[1]` |
| `(…)` | Подставляет вывод, разбивая его по строкам | `set -l dir (pwd)` |
| `math` | Вычисляет выражение | `math '2 + 3'` |
| `test` | Проверяет условие | `test -f "$path"` |
| `if … end` | Выполняет ветку при успехе | `if test -f "$path"; printf 'Found\n'; end` |
| `for … end` | Перебирает список | `for path in $files; printf '%s\n' "$path"; end` |
| `function … end` | Создаёт функцию | `function greet; printf 'Hello, %s\n' $argv; end` |
| `and` / `or` | Условно запускает следующую команду | `test -f "$path"; and cat -- "$path"` |
| `$status` | Код выхода последней команды | `echo $status` |
| `$pipestatus` | Коды команд последнего конвейера | `echo $pipestatus` |
| `read` | Читает строку | `read -l line` |

## 6. Конфигурация

В Linux, macOS и WSL обычный путь `~/.config/fish/config.fish`. Если задан `XDG_CONFIG_HOME`, Fish использует подкаталог `fish` в нём. Файлы автозагружаемых функций хранятся в `functions/`.

```fish
set -gx EDITOR vi
if status is-interactive
    alias ll 'ls -l'
end
```

## 7. Полезные приёмы

- Принимайте интерактивную подсказку стрелкой вправо.
- Проверяйте сценарий через `fish -n hello.fish`.
- Для добавления каталогов в поиск команд используйте `fish_add_path`.
- Печатайте каждый элемент списка отдельным аргументом через `printf '%s\n' $files`.
- После конвейера сохраните `$pipestatus` сразу, до следующей команды.

## 8. Частые ошибки

| Симптом | Решение |
|---|---|
| Не работает `export`-пример со сложным синтаксисом Bash | Используйте Fish-конструкцию `set -gx` |
| Ошибка при `then` или `fi` | Используйте `if … end` |
| Переменная из скрипта не сохранилась в терминале | Процесс скрипта имеет собственное окружение |
| Не найден первый аргумент | Убедитесь, что аргумент передан, через `set -q argv[1]` |

## 9. Документация

- [Загрузка Fish](https://fishshell.com/)
- [Руководство Fish](https://fishshell.com/docs/current/)
- [Репозиторий Fish](https://github.com/fish-shell/fish-shell)
