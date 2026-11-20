# PowerShell — шпаргалка

## 1. Что это и зачем

PowerShell автоматизирует администрирование и обработку данных. Его конвейер передаёт объекты со свойствами. На Windows доступны встроенный Windows PowerShell 5.1 и отдельно устанавливаемый PowerShell 7. PowerShell 7 также работает в Linux и macOS.

## 2. Установка

| ОС | Установка PowerShell 7 | Удаление |
|---|---|---|
| Windows 10/11 | `winget install --id Microsoft.PowerShell -e` или MSI Microsoft | `winget uninstall --id Microsoft.PowerShell -e` для WinGet |
| macOS | `brew install --cask powershell` или официальный PKG для своей архитектуры | `brew uninstall --cask powershell` для Homebrew |
| Ubuntu/Debian | Настройте репозиторий Microsoft по официальной инструкции и выполните `sudo apt install powershell` | `sudo apt remove powershell` |
| Arch | Официальный архив Linux для подходящей архитектуры. Распакуйте в отдельную папку и добавьте её в `PATH` | Уберите запись из `PATH` и удалите папку распаковки |

```bash
pwsh --version
```

Для Windows PowerShell 5.1 запустите `powershell.exe` и проверьте версию через `$PSVersionTable.PSVersion`. Сверяйте стабильный релиз со страницей Microsoft. Архив Linux также можно использовать на поддерживаемых Ubuntu/Debian.

## 3. Особенности ОС

- **Windows** — для совместимости с 5.1 сохраняйте `.ps1` с не-ASCII текстом в UTF-8 с BOM. PowerShell 7 читает UTF-8 без BOM.
- **Linux и macOS** — файловая система может различать регистр. Для путей используйте `Join-Path`. Команды управления Windows доступны только при наличии подходящей платформы или модуля.
- **Все ОС** — для запуска PowerShell 7 используйте `pwsh`. Проверьте требования модулей перед переносом сценария из 5.1.

## 4. Быстрый старт

Сохраните `hello.ps1`.

```powershell
param([string]$Name = 'World')

Write-Output "Hello, $Name!"
```

```powershell
pwsh -NoProfile -File ./hello.ps1 -Name 'Ada Lovelace'
```

В открытом PowerShell путь к скрипту в текущей папке указывайте как `./hello.ps1`.

## 5. Основной синтаксис

| Конструкция | Назначение | Пример |
|---|---|---|
| `$name = …` | Присваивает значение | `$name = 'Ada'` |
| `$env:NAME` | Читает или задаёт переменную окружения | `$env:APP_MODE = 'dev'` |
| `param` | Объявляет параметры сценария | `param([string]$Name = 'World')` |
| `@()` | Создаёт массив | `$names = @('Ada', 'Grace')` |
| `@{}` | Создаёт хеш-таблицу | `$settings = @{ Port = 8080 }` |
| `if` | Проверяет условие | `if ($count -gt 0) { 'Ready' }` |
| `foreach` | Перебирает элементы | `foreach ($name in $names) { $name }` |
| `function` | Объявляет функцию | `function Get-Greeting { param($Name) "Hello, $Name" }` |
| `Get-ChildItem` | Показывает дочерние элементы | `Get-ChildItem -File` |
| `Where-Object` | Фильтрует объекты конвейера | `Where-Object { $_.Length -gt 1000 }` |
| `Select-Object` | Выбирает свойства | `Select-Object Name, Length` |
| `Sort-Object` | Сортирует объекты | `Sort-Object Length -Descending` |
| `Test-Path` | Проверяет существование пути | `Test-Path -LiteralPath './notes.txt'` |
| `Get-Content` | Читает содержимое | `Get-Content -Raw -LiteralPath './notes.txt'` |
| `ConvertFrom-Json` | Разбирает JSON | `ConvertFrom-Json -InputObject '{"port":8080}'` |
| `try / catch` | Обрабатывает завершающие ошибки | `try { Get-Item -LiteralPath './notes.txt' -ErrorAction Stop } catch { Write-Warning $_.Exception.Message }` |
| `$LASTEXITCODE` | Код выхода последней внешней программы | `git --version; $LASTEXITCODE` |

```powershell
Get-ChildItem -File | Where-Object { $_.Length -gt 1000 } | Select-Object Name, Length
```

## 6. Конфигурация

`$PROFILE` показывает путь к профилю текущего пользователя и хоста. Windows PowerShell 5.1 и PowerShell 7 обычно используют разные папки в Documents. PowerShell 7 на Linux и macOS обычно использует `~/.config/powershell/`. Минимальная строка профиля.

```powershell
Set-Alias -Name ll -Value Get-ChildItem
```

## 7. Полезные приёмы

- Используйте `Get-Help Get-ChildItem -Examples` для примеров команд.
- Передавайте путь через `-LiteralPath`, если в имени есть скобки или звёздочки.
- Перед `Remove-Item` или другими изменяющими командами применяйте `-WhatIf`, когда команда его поддерживает.
- Вызывайте `Format-Table` в конце конвейера. Для CSV передавайте исходные объекты в `Export-Csv`.
- Используйте `-ErrorAction Stop`, когда ошибка cmdlet должна попасть в `catch`. Для внешних программ проверяйте `$LASTEXITCODE` сразу после вызова.

## 8. Частые ошибки

| Симптом | Причина | Решение |
|---|---|---|
| `pwsh` не найден | PowerShell 7 отсутствует или не в `PATH` | Проверьте установку и откройте новый терминал |
| `running scripts is disabled` | Скрипт блокирует действующая политика Windows | Проверьте `Get-ExecutionPolicy -List` и используйте правила подписания или запуска своей организации |
| Ошибка осталась вне `catch` | Cmdlet выдал незавершающую ошибку | Укажите `-ErrorAction Stop` |
| Переменная не подставилась | Строка в одинарных кавычках | Используйте двойные кавычки для интерполяции |

## 9. Документация

- [Установка PowerShell](https://learn.microsoft.com/powershell/scripting/install/installing-powershell)
- [Руководство PowerShell](https://learn.microsoft.com/powershell/)
- [Репозиторий PowerShell](https://github.com/PowerShell/PowerShell)
