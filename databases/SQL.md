# Чистый SQL — шпаргалка для новичка

## 1. Что это и зачем

SQL — декларативный язык для чтения, изменения и описания данных в реляционных базах. Он подходит для связей между таблицами, отчётов и транзакций. Для хранения файлов или простого доступа по ключу обычно удобнее файловое хранилище или Redis.

Основа шпаргалки — распространённый SQL-синтаксис. SQLite используется для локальной практики, а его особенности отмечены отдельно. Стандарт языка и версии конкретных СУБД выпускаются независимо.

## 2. Установка среды для практики

Устанавливается CLI SQLite. Версия из пакетного менеджера зависит от ОС и репозитория. Для последней стабильной версии сверьте вывод команды проверки с [официальной страницей загрузки](https://www.sqlite.org/download.html).

### Windows 10/11 — PowerShell

```powershell
winget install --id SQLite.SQLite -e
```

```powershell
sqlite3 --version
```

Альтернатива — распакуйте архив `sqlite-tools` для своей архитектуры и добавьте папку с `sqlite3.exe` в пользовательский `Path`. Затем откройте новый терминал.

```powershell
winget uninstall --id SQLite.SQLite -e
```

При ручной установке уберите папку из `Path` и удалите распакованные инструменты.

### macOS — Intel и Apple Silicon

```bash
brew install sqlite
```

```bash
"$(brew --prefix sqlite)/bin/sqlite3" --version
```

Альтернатива — системный `/usr/bin/sqlite3`, если он доступен. Чтобы интерактивный Zsh выбирал Homebrew-версию, добавьте строку в `~/.zshrc`, затем откройте новый терминал.

```bash
export PATH="$(brew --prefix sqlite)/bin:$PATH"
```

```bash
brew uninstall sqlite
```

### Ubuntu / Debian

```bash
sudo apt update
```

```bash
sudo apt install sqlite3
```

```bash
sqlite3 --version
```

Альтернатива — официальный CLI-архив для Linux, если он доступен для вашей архитектуры. Распакованный бинарник можно запускать по полному пути или положить в `~/.local/bin`, проверив, что каталог есть в `PATH`.

```bash
sudo apt remove sqlite3
```

### Arch Linux

```bash
sudo pacman -Syu sqlite
```

```bash
sqlite3 --version
```

Альтернатива — официальный CLI-архив для вашей архитектуры, с теми же вариантами запуска, что и на Ubuntu / Debian.

```bash
sudo pacman -R sqlite
```

⚠️ На Linux команды с `sudo` меняют системные пакеты. В Arch `-Syu` также обновляет систему. Перед удалением проверьте список пакетов и зависимости. Свои файлы баз храните отдельно от папки установки.

## 3. Особенности по ОС

- **Windows.** Если команда недоступна после установки, откройте новый терминал и проверьте пользовательский `Path`. Для пути к базе с пробелами используйте кавычки.
- **macOS.** Системная и Homebrew-версии могут отличаться. Путь через префикс Homebrew работает без привязки к Intel или Apple Silicon.
- **Linux / WSL.** В WSL устанавливайте пакет внутри дистрибутива. Для локальной практики храните базу в домашней папке Linux. Для записи SQLite могут требоваться права на файл базы и её родительский каталог.

## 4. Быстрый старт

Запустите CLI без имени файла. Он откроет временную базу в памяти. Для сохранения данных передайте имя файла, например `practice.db`.

```bash
sqlite3
```

Вставьте SQL целиком в приглашение `sqlite>` в новом сеансе. Остальные примеры используют эти же таблицы.

```sql
SELECT 'Hello, SQL!' AS message;

CREATE TABLE customers (
    id INTEGER PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    city VARCHAR(100)
);

CREATE TABLE orders (
    id INTEGER PRIMARY KEY,
    customer_id INTEGER NOT NULL REFERENCES customers (id),
    total_cents INTEGER NOT NULL CHECK (total_cents >= 0)
);

INSERT INTO customers (id, name, city) VALUES
    (1, 'Anna', 'Kazan'), (2, 'Ivan', NULL), (3, 'Olga', 'Kazan');
INSERT INTO orders (id, customer_id, total_cents) VALUES
    (101, 1, 1500), (102, 1, 2500), (103, 2, 3000);

SELECT name FROM customers WHERE city = 'Kazan' ORDER BY id;
```

Последний запрос вернёт `Anna` и `Olga`. Команда выхода из CLI — `.quit` без точки с запятой.

## 5. Основной синтаксис

В таблицах — короткие фрагменты. Полные запросы для копирования находятся в блоках SQL.

| Команда | Что делает | Пример |
|---|---|---|
| `SELECT … FROM` | Выбирает столбцы | `SELECT name FROM customers` |
| `WHERE` | Фильтрует строки | `WHERE total_cents >= 2000` |
| `AND / OR / NOT` | Объединяет условия | `WHERE (id = 1 OR id = 2) AND city IS NOT NULL` |
| `IN / BETWEEN / LIKE` | Проверяет список, диапазон с границами или шаблон | `WHERE name LIKE 'A%'` |
| `IS NULL` | Проверяет отсутствие значения | `WHERE city IS NULL` |
| `DISTINCT` | Убирает дубли по всем выбранным столбцам | `SELECT DISTINCT city FROM customers` |
| `ORDER BY` | Сортирует результат | `ORDER BY total_cents DESC, id` |
| `COALESCE` | Возвращает первое значение, отличное от `NULL` | `COALESCE(city, 'Unknown')` |
| `CASE` | Вычисляет значение по условию | `CASE WHEN total_cents >= 2000 THEN 'large' ELSE 'small' END` |

| Команда | Что делает | Пример |
|---|---|---|
| `INNER JOIN` | Оставляет совпавшие пары строк | `JOIN orders AS o ON o.customer_id = c.id` |
| `LEFT JOIN` | Сохраняет все строки слева | `LEFT JOIN orders AS o ON o.customer_id = c.id` |
| `GROUP BY` | Объединяет строки в группы | `GROUP BY c.id, c.name` |
| `COUNT / SUM / AVG / MIN / MAX` | Считает показатели по группе | `COUNT(o.id)` |
| `HAVING` | Фильтрует группы | `HAVING COUNT(o.id) >= 2` |
| `EXISTS` | Проверяет наличие строк в подзапросе | `WHERE EXISTS (SELECT 1 FROM orders AS o WHERE o.customer_id = c.id)` |
| `UNION / UNION ALL` | Складывает результаты с удалением дублей или с их сохранением | `SELECT city FROM customers UNION ALL SELECT city FROM customers` |
| `WITH` | Даёт имя промежуточному результату в одном запросе | `WITH big_orders AS (SELECT id FROM orders WHERE total_cents >= 2000) SELECT id FROM big_orders` |
| `ROW_NUMBER() OVER` | Нумерует строки внутри окна | `ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY id)` |

```sql
SELECT c.id, c.name, COUNT(o.id) AS order_count,
       COALESCE(SUM(o.total_cents), 0) AS total_cents
FROM customers AS c
LEFT JOIN orders AS o ON o.customer_id = c.id
GROUP BY c.id, c.name
ORDER BY order_count DESC, c.id;
```

`COUNT(o.id)` вернёт `0` для Olga. `COUNT(*)` посчитает сохранённую соединением строку и вернёт `1`.

| Команда | Что делает | Пример |
|---|---|---|
| `INSERT` | Добавляет строки | `INSERT INTO customers (id, name) VALUES (4, 'Maria')` |
| `UPDATE` | Меняет строки | `UPDATE customers SET city = 'Perm' WHERE id = 2` |
| `DELETE` | Удаляет строки | `DELETE FROM orders WHERE id = 101` |
| `CREATE TABLE` | Создаёт таблицу и ограничения | `CREATE TABLE tags (id INTEGER PRIMARY KEY)` |
| `ALTER TABLE` | Меняет структуру | `ALTER TABLE customers ADD COLUMN email VARCHAR(200)` |
| `CREATE INDEX` | Добавляет индекс | `CREATE INDEX idx_orders_customer ON orders (customer_id)` |
| `DROP TABLE` | Удаляет таблицу вместе с данными | `DROP TABLE tags` |
| `BEGIN / COMMIT / ROLLBACK` | Начинает, фиксирует или откатывает транзакцию | `BEGIN` в SQLite; `START TRANSACTION` в стандартном SQL |

⚠️ `UPDATE` и `DELETE` без `WHERE` затрагивают все строки. Перед изменением проверьте тот же фильтр через `SELECT`. `DROP TABLE` удаляет структуру и данные, а возможность отката зависит от СУБД.

```sql
-- SQLite. После проверки можно выбрать COMMIT вместо ROLLBACK.
BEGIN;
SELECT id, city FROM customers WHERE id = 2;
UPDATE customers SET city = 'Perm' WHERE id = 2;
SELECT id, city FROM customers WHERE id = 2;
ROLLBACK;
```

## 6. Конфигурация

У SQL как языка нет общего файла конфигурации. У SQLite CLI есть необязательный пользовательский `.sqliterc`. На macOS и Linux это обычно `~/.sqliterc`, на Windows — `.sqliterc` в домашнем каталоге, который определяет CLI. Явный путь к файлу можно передать через `sqlite3 -init PATH`.

Минимальный `.sqliterc` для CLI SQLite. Точечные команды настраивают вывод, а `PRAGMA` включает проверку внешних ключей для текущего подключения. В приложении настройте её для каждого соединения, до начала транзакции.

```text
.headers on
.mode column
PRAGMA foreign_keys = ON;
```

## 7. Полезные приёмы

- Для чтения запроса держите в уме упрощённый порядок `FROM / JOIN → WHERE → GROUP BY → HAVING → SELECT → ORDER BY`.
- Указывайте нужные столбцы явно и добавляйте стабильный критерий сортировки, например `id`, когда значения совпадают.
- Для первых N строк стандарт SQL использует `FETCH FIRST 10 ROWS ONLY`. В SQLite используйте `LIMIT 10`. В обоих случаях задайте `ORDER BY`, чтобы результат был предсказуем.
- В `LIKE` символ `%` означает любую последовательность символов, `_` — один символ. Чувствительность к регистру зависит от СУБД и настроек.
- Для суммы денег в примере используются целые сотые доли валюты. SQLite допускает гибкую типизацию и не ограничивает длину по `VARCHAR(n)`.
- Передавайте пользовательский ввод параметрами драйвера. Формат заполнителей зависит от драйвера и СУБД.
- В SQLite используйте `.schema` для просмотра структуры и `EXPLAIN QUERY PLAN` перед запросом для проверки плана. Эти команды специфичны для SQLite.

## 8. Частые ошибки

| Ошибка или симптом | Причина | Решение |
|---|---|---|
| `sqlite3` не найден | Каталога с CLI нет в `PATH` или терминал открыт до установки | Откройте новый терминал или запустите CLI по полному пути |
| `no such table: customers` | Открыта другая база или новая сессия в памяти | Выполните быстрый старт в текущей сессии или откройте нужный файл |
| Приглашение `...>` в CLI | SQLite ждёт завершения инструкции | Закройте кавычки и скобки, затем поставьте `;` |
| `WHERE city = NULL` ничего не возвращает | Сравнение с `NULL` даёт неопределённый результат | Используйте `IS NULL` или `IS NOT NULL` |
| После `LEFT JOIN` исчезли строки без пары | `WHERE` отфильтровал строки по правой таблице | Перенесите условие на правую таблицу в `ON`, чтобы сохранить строки слева |
| Лишние строки после `JOIN` | Одной строке соответствуют несколько строк в другой таблице | Проверьте ключи соединения. Для проверки наличия используйте `EXISTS` |
| `FOREIGN KEY constraint failed` | Добавляется ссылка на отсутствующую запись или удаляется запись с зависимостями | Проверьте родительские и дочерние строки перед изменением |
| `database is locked` | Другое подключение удерживает блокировку | Завершите открытую транзакцию в другом подключении и повторите операцию |

## 9. Документация

- [Литералы, выражения и команды SQLite](https://www.sqlite.org/lang.html)
- [SQLite CLI](https://www.sqlite.org/cli.html)
- [Официальный репозиторий SQLite](https://sqlite.org/src/)
- [Соответствие PostgreSQL стандарту SQL](https://www.postgresql.org/docs/current/features.html)
