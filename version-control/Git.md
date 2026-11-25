# Git — шпаргалка

## 1. Что это и зачем

Git хранит историю изменений файлов, позволяет работать в ветках и синхронизировать коммиты между копиями репозитория. Он удобен для исходного кода и текстовых документов. Для крупных бинарных файлов рассмотрите Git LFS, а для централизованного контроля версий — Subversion.

## 2. Установка и удаление

Используйте актуальный стабильный пакет для своей ОС. Версия пакета дистрибутива может отличаться от последнего выпуска Git. Ссылки в разделе 10 ведут на официальные загрузки.

### Windows 10/11 — PowerShell

```powershell
winget install --id Git.Git -e
```

Альтернатива — официальный установщик Git for Windows. Для PowerShell включите доступ к Git из командной строки в установщике. После установки откройте новый терминал.

```powershell
git --version
```

```powershell
winget uninstall --id Git.Git -e
```

### macOS — Intel и Apple Silicon

```bash
brew install git
```

Альтернатива — Git из Xcode Command Line Tools.

```bash
xcode-select --install
```

```bash
git --version
```

```bash
brew uninstall git
```

### Ubuntu / Debian

```bash
sudo apt update
```

```bash
sudo apt install git
```

```bash
git --version
```

```bash
sudo apt remove git
```

### Arch Linux

```bash
sudo pacman -Syu git
```

```bash
git --version
```

```bash
sudo pacman -R git
```

Для Linux альтернативой пакетному менеджеру служит сборка из официального архива исходников с отдельным префиксом установки. Команды с `sudo` меняют системные пакеты. На Arch `-Syu` выполняет обновление системы.

## 3. Особенности по ОС

- **Windows** — используйте PowerShell или Git Bash. В WSL работайте с Linux-версией Git и храните проект в файловой системе WSL при работе инструментами Linux.
- **macOS** — при наличии двух версий проверьте, какой `git` первым находится в `PATH`. Для Homebrew настройте оболочку по подсказке установщика.
- **Все ОС** — окончания строк удобнее закрепить в `.gitattributes` проекта. На нечувствительных к регистру файловых системах переименование с изменением только регистра делайте через промежуточное имя.

## 4. Быстрый старт

Из родительского каталога создайте новый учебный репозиторий. Команды подходят для PowerShell и Bash. Используйте свободное имя папки.

```bash
git init -b main git-practice
```

```bash
cd git-practice
```

Настройте имя и email только для этой папки. Замените учебные значения своими, если будете публиковать коммиты.

```bash
git config user.name "Student"
```

```bash
git config user.email "student@example.com"
```

```bash
echo '# Git practice' > README.md
```

```bash
git add README.md
```

```bash
git diff --staged
```

```bash
git commit -m "Add practice README"
```

```bash
git status
```

`add` записывает выбранную версию файла в индекс. `commit` сохраняет снимок индекса в локальной истории.

## 5. Основные команды

### Локальная работа

| Команда | Что делает | Пример |
|---|---|---|
| `init` | Создаёт репозиторий | `git init -b main` |
| `status` | Показывает состояние рабочей копии и индекса | `git status -sb` |
| `add` | Добавляет изменения в индекс | `git add README.md` |
| `diff` | Показывает изменения вне индекса | `git diff` |
| `diff --staged` | Показывает содержимое будущего коммита | `git diff --staged` |
| `commit` | Сохраняет изменения из индекса | `git commit -m "Update README"` |
| `log` | Показывает историю | `git log --oneline --graph --all` |
| `show` | Показывает содержимое объекта или коммита | `git show HEAD` |
| `branch` | Показывает ветки | `git branch -av` |
| `switch` | Переключает ветку, с `-c` создаёт новую | `git switch -c docs` |
| `merge` | Вливает указанную ветку в текущую | `git merge docs` |
| `rebase` | Переносит коммиты текущей ветки на новую базу | `git rebase main` |
| `tag` | Ставит метку на коммит | `git tag -a v1.0.0 -m "Version 1.0.0"` |

### Удалённый репозиторий и откат

| Команда | Что делает | Пример |
|---|---|---|
| `clone` | Создаёт локальную копию | `git clone https://github.com/git/git.git git-source` |
| `remote` | Показывает источники и адреса | `git remote -v` |
| `fetch` | Загружает историю без интеграции в текущую ветку | `git fetch origin` |
| `pull --ff-only` | Загружает и обновляет ветку только прямым продвижением | `git pull --ff-only` |
| `push -u` | Публикует ветку и назначает upstream | `git push -u origin docs` |
| `restore --staged` | Убирает файл из индекса, сохраняя изменения в рабочей копии | `git restore --staged README.md` |
| `restore` | Заменяет рабочую копию файла версией из индекса | `git restore -- README.md` |
| `revert` | Создаёт новый коммит, отменяющий выбранный | `git revert HEAD` |
| `stash` | Временно сохраняет локальные изменения | `git stash push -u -m "Draft"` |
| `stash pop` | Применяет и при успехе убирает сохранённый набор | `git stash pop` |
| `reflog` | Показывает локальные перемещения ссылок | `git reflog` |

⚠️ `git restore -- файл` отбрасывает незаписанные в индекс правки файла. `rebase` меняет идентификаторы переносимых коммитов. Для отката опубликованного коммита обычно подходит `revert`.

## 6. Конфигурация

Локальная конфигурация обычно лежит в `.git/config`. Пользовательская — в `%USERPROFILE%\.gitconfig` на Windows и `~/.gitconfig` на macOS и Linux. Также возможен пользовательский файл `$XDG_CONFIG_HOME/git/config`. Пути системных файлов зависят от установки.

```bash
git config --show-origin --list
```

Минимальный фрагмент пользовательского `.gitconfig`.

```ini
[user]
    name = Student
    email = student@example.com
[init]
    defaultBranch = main
```

### Уровни и глобальные настройки

Git читает настройки из нескольких уровней. Более поздний уровень в списке переопределяет предыдущий.

| Уровень | Ключ | Где хранится | Область действия |
|---|---|---|---|
| Системный | `--system` | зависит от установки | все пользователи ОС |
| Глобальный | `--global` | `~/.gitconfig` | все репозитории пользователя |
| Локальный | `--local` (по умолчанию при записи) | `.git/config` | один репозиторий |
| Разовый | `git -c ключ=значение …` | нигде | одна команда |

Глобальные значения задают привычки для всех проектов, а локальные переопределяют их в конкретном репозитории. Например, `git config user.email "work@example.com"` без `--global` меняет email только в текущем проекте.

```bash
git config --global user.name "Student"
git config --global user.email "student@example.com"
git config --global init.defaultBranch main
git config --global pull.ff only
git config --global push.autoSetupRemote true
git config --global alias.st "status -sb"
```

`push.autoSetupRemote` требует Git 2.37 или новее. Чтение, удаление и редактирование:

```bash
git config --global --list
git config --show-origin --get user.email
git config --global --unset alias.st
git config --global --edit
```

`--show-origin` показывает, из какого файла пришло значение. Это помогает понять, какой уровень «победил».

Чтобы автоматически подставлять другой email и ключ для рабочих проектов, подключите отдельный файл через `includeIf` в `~/.gitconfig`. Правило действует для всех репозиториев внутри каталога. Слеш в конце обязателен.

```ini
[includeIf "gitdir:~/work/"]
    path = ~/.gitconfig-work
```

В `~/.gitconfig-work` задайте, например, `user.email` и `core.sshCommand`.

### Переменные окружения

Переменные окружения действуют в текущем сеансе оболочки и переопределяют значения из файлов конфигурации.

| Переменная | Назначение |
|---|---|
| `GIT_AUTHOR_NAME`, `GIT_AUTHOR_EMAIL` | Автор следующего коммита |
| `GIT_COMMITTER_NAME`, `GIT_COMMITTER_EMAIL` | Коммитер следующего коммита |
| `GIT_SSH_COMMAND` | Команда SSH для сетевых операций, например с другим ключом |
| `GIT_CONFIG_GLOBAL` | Путь к глобальному файлу вместо `~/.gitconfig` (Git 2.32+) |
| `GIT_CONFIG_NOSYSTEM` | Значение `1` отключает системный файл |
| `GIT_EDITOR` | Редактор для сообщений коммитов |
| `GIT_DIR`, `GIT_WORK_TREE` | Расположение каталога `.git` и рабочей копии |
| `GIT_TRACE` | Значение `1` включает отладочный вывод |

```bash
GIT_SSH_COMMAND="ssh -i ~/.ssh/id_ed25519_work -o IdentitiesOnly=yes" git fetch origin
```

```powershell
$env:GIT_TRACE = "1"; git status; Remove-Item Env:GIT_TRACE
```

Для постоянного значения добавьте `export ИМЯ=значение` в `~/.bashrc` или `~/.zshrc`. В Windows используйте `setx ИМЯ "значение"`: оно применится в новых терминалах. Не храните пароли и токены в `.gitconfig` и в переменных окружения, которые попадают в историю оболочки.

## 7. SSH: доступ, подпись коммитов и перенос ключей

SSH-ключ состоит из пары файлов: **закрытый** (`id_ed25519`) остаётся только у вас, **открытый** (`id_ed25519.pub`) загружается на хостинг. Одна и та же пара может служить и для доступа к репозиторию, и для подписи коммитов, но на хостинге её добавляют отдельно для каждой цели.

### Создание ключа

Тип `ed25519` короткий и поддерживается основными хостингами. Задайте парольную фразу, когда её запросят. Если файл с таким именем уже есть, не перезаписывайте его: укажите другое имя через `-f`.

```bash
ssh-keygen -t ed25519 -C "student@example.com" -f ~/.ssh/id_ed25519
```

```powershell
ssh-keygen -t ed25519 -C "student@example.com" -f "$env:USERPROFILE\.ssh\id_ed25519"
```

### Агент ключей

Агент хранит расшифрованный ключ в памяти, чтобы не вводить парольную фразу при каждой операции.

```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
```

На macOS можно сохранить парольную фразу в связке ключей (в более старых выпусках вместо `--apple-use-keychain` используется `-K`).

```bash
ssh-add --apple-use-keychain ~/.ssh/id_ed25519
```

В Windows служба `ssh-agent` по умолчанию отключена. Включите её в PowerShell, запущенном от имени администратора, затем добавьте ключ из обычного сеанса.

```powershell
Get-Service ssh-agent | Set-Service -StartupType Automatic
Start-Service ssh-agent
```

```powershell
ssh-add "$env:USERPROFILE\.ssh\id_ed25519"
```

Если Git for Windows использует встроенный SSH, он не видит службу Windows. Либо выберите внешний OpenSSH в установщике, либо укажите его явно:

```powershell
git config --global core.sshCommand "C:/Windows/System32/OpenSSH/ssh.exe"
```

### Доступ к репозиторию по SSH

Выведите открытый ключ и добавьте его на хостинге в разделе SSH-ключей аккаунта (на GitHub и GitLab — в настройках профиля). Никогда не отправляйте файл без `.pub`.

```bash
cat ~/.ssh/id_ed25519.pub
```

```powershell
Get-Content "$env:USERPROFILE\.ssh\id_ed25519.pub"
```

Проверьте соединение. При первом подключении ssh спросит подтверждение отпечатка хоста: сверьте его с документацией хостинга. Успешный ответ GitHub содержит приветствие с вашим именем и сообщение, что shell-доступа нет.

```bash
ssh -T git@github.com
```

Переключите существующий репозиторий с HTTPS на SSH:

```bash
git remote set-url origin git@github.com:USER/REPO.git
```

Для нескольких ключей или хостингов опишите их в `~/.ssh/config`. `IdentitiesOnly yes` запрещает ssh перебирать лишние ключи из агента. На macOS можно добавить строку `UseKeychain yes`: её понимает только SSH от Apple.

```text
Host github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519
    IdentitiesOnly yes
    AddKeysToAgent yes
```

### Подпись коммитов по SSH

Нужен Git 2.34 или новее и OpenSSH 8.0 или новее. Укажите формат подписи и открытый ключ. Закрытый ключ при этом должен быть доступен через `ssh-agent` или запрашивать парольную фразу.

```bash
git config --global gpg.format ssh
git config --global user.signingkey ~/.ssh/id_ed25519.pub
git config --global commit.gpgsign true
git config --global tag.gpgsign true
```

С `commit.gpgsign` подписываются все новые коммиты. Без неё подписывайте вручную: `git commit -S -m "..."` и `git tag -s v1.0.0 -m "..."`.

Для локальной проверки подписей создайте файл допустимых подписантов. Строка состоит из email и открытого ключа. Email должен совпадать с адресом коммитера.

```bash
git config --global gpg.ssh.allowedSignersFile ~/.ssh/allowed_signers
echo "student@example.com $(cat ~/.ssh/id_ed25519.pub)" >> ~/.ssh/allowed_signers
```

```powershell
"student@example.com $(Get-Content "$env:USERPROFILE\.ssh\id_ed25519.pub")" | Add-Content "$env:USERPROFILE\.ssh\allowed_signers"
```

```bash
git log --show-signature -1
```

```bash
git verify-commit HEAD
```

Для корректной подписи вывод содержит строку `Good "git" signature` с вашим email. Чтобы рядом с коммитом на хостинге появилась отметка о проверенной подписи, добавьте тот же открытый ключ ещё раз с типом **Signing** (на GitHub — «Signing Key», на GitLab — использование «Signing» либо «Authentication & Signing»). Email коммитера должен быть подтверждён в аккаунте.

### Перенос ключей на другое устройство

Безопаснее создать **новую пару на каждом устройстве** и добавить новый открытый ключ на хостинг, а ключ потерянного или проданного устройства удалить. Если нужен тот же ключ, копируйте оба файла безопасным способом: `scp`, зашифрованная флешка или менеджер паролей. Не отправляйте закрытый ключ почтой, в мессенджерах или в облачных папках без шифрования. Ключи, созданные как `ed25519-sk` на аппаратном токене, перенести нельзя: для каждого токена нужна своя пара.

На новом устройстве выполните копирование со старого (замените имя и адрес):

```bash
mkdir -p ~/.ssh && chmod 700 ~/.ssh
scp olduser@oldhost:~/.ssh/id_ed25519 olduser@oldhost:~/.ssh/id_ed25519.pub ~/.ssh/
chmod 600 ~/.ssh/id_ed25519
chmod 644 ~/.ssh/id_ed25519.pub
```

Не забудьте перенести и сопутствующие файлы: `~/.ssh/config`, `~/.ssh/allowed_signers` и настройки Git из `~/.gitconfig` (или повторите команды `git config --global`). На Windows ограничьте доступ к закрытому ключу, иначе ssh откажется им пользоваться:

```powershell
$key = "$env:USERPROFILE\.ssh\id_ed25519"
icacls $key /inheritance:r /grant:r "$($env:USERNAME):F"
```

Проверьте, что ключ не повреждён: отпечаток должен совпасть со старым устройством. Если потерялся файл `.pub`, его можно восстановить из закрытого ключа.

```bash
ssh-keygen -lf ~/.ssh/id_ed25519.pub
ssh-keygen -y -f ~/.ssh/id_ed25519 > ~/.ssh/id_ed25519.pub
```

Затем загрузите ключ в агент и проверьте соединение: `ssh-add ~/.ssh/id_ed25519` и `ssh -T git@github.com`. Сменить парольную фразу позволяет `ssh-keygen -p -f ~/.ssh/id_ed25519`.

⚠️ В WSL свой каталог `~/.ssh`: ключи Windows и WSL не совпадают, пока вы не скопируете их явно. Если закрытый ключ мог попасть в чужие руки, удалите его открытую часть на хостинге и создайте новую пару.

## 8. Полезные приёмы

- Перед коммитом проверьте `status` и `diff --staged`. Через `git add -p` можно добавить только часть правок.
- Используйте `.gitignore` для локальных файлов окружения и артефактов сборки. Уже отслеживаемые файлы продолжат отслеживаться.
- Для просмотра изменений перед интеграцией сначала сделайте `fetch`, затем сравните `HEAD` с `origin/main`.
- При конфликте проверьте `status`, исправьте конфликтные файлы и добавьте их через `add`. Затем продолжите операцию с помощью `git merge --continue` или `git rebase --continue`.
- Если передумали продолжать конфликтный merge или rebase, используйте соответствующую команду `--abort`. До начала операции лучше сохранить текущие правки.
- При случайной публикации секрета сначала отзовите или замените его. Удаление файла новым коммитом сохраняет старую версию в истории.

## 9. Частые ошибки

| Ошибка | Причина | Решение |
|---|---|---|
| `not a git repository` | Открыта папка вне репозитория | Перейдите в корень проекта |
| `Author identity unknown` | Не задано имя или email | Настройте `user.name` и `user.email` |
| `nothing to commit` | Индекс пуст или изменений нет | Проверьте `git status` и добавьте нужные файлы |
| `non-fast-forward` | На сервере есть новые коммиты | Выполните `fetch`, проверьте историю и согласуйте способ интеграции |
| `Permission denied (publickey)` | Сервер не принял SSH-ключ | Проверьте адрес `origin`, SSH-агент и регистрацию открытого ключа на хостинге |
| `CONFLICT` | Git не смог автоматически объединить правки | Исправьте конфликтные файлы и продолжите операцию либо отмените её |
| `UNPROTECTED PRIVATE KEY FILE` | У закрытого ключа слишком широкие права доступа | На macOS и Linux выполните `chmod 600`, в Windows ограничьте доступ через `icacls` |
| `Could not open a connection to your authentication agent` | `ssh-agent` не запущен в текущем сеансе | Запустите агент и выполните `ssh-add`. В Windows включите службу `ssh-agent` |
| `Load key ...: error in libcrypto` или `invalid format` | Файл ключа повреждён при копировании | Скопируйте файл заново без изменений, включая окончания строк |
| `gpg.ssh.allowedSignersFile needs to be configured` | Для проверки SSH-подписи не задан файл допустимых подписантов | Задайте `gpg.ssh.allowedSignersFile` и добавьте в файл email и открытый ключ |

## 10. Документация

- [Установка Git](https://git-scm.com/install/)
- [Справочник команд](https://git-scm.com/docs) и [Pro Git на русском](https://git-scm.com/book/ru/v2)
- [Репозиторий Git](https://github.com/git/git)
- [git config](https://git-scm.com/docs/git-config) и [переменные окружения Git](https://git-scm.com/docs/git#_git_commits)
- [GitHub: подключение по SSH](https://docs.github.com/en/authentication/connecting-to-github-with-ssh)
- [GitHub: подпись коммитов ключом SSH](https://docs.github.com/en/authentication/managing-commit-signature-verification/telling-git-about-your-signing-key)
