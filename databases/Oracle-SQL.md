# Oracle SQL — шпаргалка для новичка

## 1. Что это и зачем

Oracle SQL — диалект SQL для Oracle Database. Подходит для запросов и транзакций в Oracle, включая отчёты и обработку больших наборов данных. Для небольшого приложения с локальным файлом можно выбрать SQLite. Примеры ориентированы на Oracle Database 19c и новее. Актуальный выпуск клиента сверяйте на странице загрузки.

## 2. Установка

SQLcl — клиент командной строки. Для выполнения запросов нужны адрес, порт, имя сервиса и учётная запись в Oracle Database. Для локальной практики можно использовать Oracle Database Free и официальный образ из Oracle Container Registry.

### Windows 10/11

Скачайте ZIP SQLcl с сайта Oracle и распакуйте в пользовательский каталог. Установите версию Java, указанную в требованиях к выпуску, и добавьте `bin` из SQLcl в пользовательский `Path`. Альтернатива — Instant Client с пакетами Basic и SQL*Plus для одной архитектуры и версии.

```powershell
sql -version
```

Для удаления уберите каталог из `Path` и удалите распакованный архивный комплект.

### macOS — Intel и Apple Silicon

```bash
brew install --cask sqlcl
```

```bash
sql -version
```

Альтернатива — архив Oracle и Java для своей архитектуры. Добавьте каталог `bin` SQLcl в `PATH`, если используете архив.

```bash
brew uninstall --cask sqlcl
```

### Ubuntu / Debian

Установите Java из репозитория или у поставщика JDK согласно требованиям SQLcl. Распакуйте официальный ZIP SQLcl и добавьте его `bin` в `PATH` через профиль оболочки. Альтернатива — ZIP Instant Client Basic и SQL*Plus из одной версии для вашей архитектуры. Для Instant Client сверяйте системные зависимости в инструкции Oracle.

```bash
sql -version
```

Для удаления уберите добавленный путь из профиля и удалите каталог SQLcl.

### Arch Linux

Используйте официальный ZIP SQLcl и совместимую Java. При нескольких JDK выберите поддерживаемый выпуск через `JAVA_HOME` и проверьте порядок каталогов в `PATH`. Альтернатива — графический Oracle SQL Developer, если текущий выпуск поддерживает вашу платформу.

```bash
sql -version
```

Для удаления уберите добавленный путь из профиля и удалите каталог SQLcl.

### SQL*Plus — установка и работа

SQL*Plus запускается командой `sqlplus`. Он входит в состав Oracle Database и доступен отдельно через Instant Client. Скачайте пакеты **Basic** и **SQL*Plus** одной версии и архитектуры, распакуйте их в один каталог и добавьте его в `Path` на Windows или `PATH` на macOS и Linux. На Linux при ошибке загрузки библиотек проверьте настройку `ldconfig` или `LD_LIBRARY_PATH` по инструкции для выбранного выпуска. Для удаления ZIP-установки уберите добавленные пути и удалите каталог Instant Client, если его не используют другие приложения. [Загрузки и инструкции Oracle Instant Client](https://www.oracle.com/database/technologies/instant-client/downloads.html)

```bash
sqlplus -V
```

Подключитесь к учебной схеме через Easy Connect. Флаг `-L` отключает повторные попытки входа после ошибки. Пароль вводится по запросу.

```bash
sqlplus -L student@//localhost:1521/FREEPDB1
```

| Команда SQL*Plus | Что делает |
|---|---|
| `SHOW USER` | Показывает текущего пользователя |
| `DESC demo_orders` | Показывает столбцы и типы таблицы |
| `SET LINESIZE 120` | Задаёт ширину вывода |
| `SET SERVEROUTPUT ON` | Показывает вывод `DBMS_OUTPUT` из PL/SQL |
| `@report.sql` | Выполняет SQL-скрипт |
| `SPOOL report.txt` / `SPOOL OFF` | Начинает и завершает запись вывода в файл |
| `EXIT ROLLBACK` | Выходит с откатом незавершённой транзакции |

Для пакетного запуска сохраните пример в `report.sql`. Таблица `demo_orders` создаётся в разделе быстрого старта.

```sql
WHENEVER OSERROR EXIT FAILURE ROLLBACK
WHENEVER SQLERROR EXIT FAILURE ROLLBACK
SET PAGESIZE 50
SET LINESIZE 120
SELECT order_code, customer, total_cents FROM demo_orders ORDER BY id;
EXIT SUCCESS ROLLBACK
```

```bash
sqlplus -L student@//localhost:1521/FREEPDB1 @report.sql
```

Команды SQL*Plus вводятся без `;`. Запрос SQL завершайте `;`. Для блока PL/SQL добавляйте `/` на отдельной строке после `END;`. Не добавляйте `/` после уже выполненного SQL-запроса, поскольку он повторно запустит содержимое буфера. `WHENEVER SQLERROR` обрабатывает ошибки SQL и PL/SQL. Для ошибок собственных команд SQL*Plus это правило не действует. [Руководство SQL*Plus](https://docs.oracle.com/en/database/oracle/oracle-database/19/sqpug/)

## 3. Особенности по ОС

- **Windows.** После изменения `Path` откройте новый PowerShell. В WSL используйте Linux-версию Java и SQLcl внутри дистрибутива.
- **macOS.** При ошибке запуска проверьте требования к Java и соответствие архитектуры JDK вашему окружению.
- **Linux.** При нескольких версиях Java проверьте, какую находит оболочка. SQLcl можно распаковать в домашнюю папку без `sudo`.
- **Все ОС.** Для контейнера Oracle Database Free проверьте, доступен ли тег для архитектуры хоста. Сервис учебной PDB обычно называется `FREEPDB1`.

## 4. Быстрый старт

Подключитесь к учебному пользователю с правом `CREATE TABLE` и квотой на табличное пространство. Пример использует локальную Database Free. SQLcl запросит пароль.

```bash
sql student@//localhost:1521/FREEPDB1
```

Выполните блок один раз в учебной схеме.

```sql
SELECT 'Hello, Oracle!' AS message FROM dual;

CREATE TABLE demo_orders (
    id NUMBER GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
    order_code VARCHAR2(20 CHAR) NOT NULL UNIQUE,
    customer VARCHAR2(100 CHAR) NOT NULL,
    total_cents NUMBER(12, 0) NOT NULL CHECK (total_cents >= 0),
    ordered_on DATE DEFAULT TRUNC(SYSDATE) NOT NULL
);

INSERT INTO demo_orders (order_code, customer, total_cents)
VALUES ('A101', 'Anna', 1500);
INSERT INTO demo_orders (order_code, customer, total_cents)
VALUES ('A102', 'Ivan', 3000);
COMMIT;

SELECT customer, total_cents FROM demo_orders ORDER BY id;
```

## 5. Основной синтаксис

| Команда | Что делает | Пример |
|---|---|---|
| `SELECT` | Выбирает столбцы | `SELECT customer FROM demo_orders` |
| `WHERE` | Фильтрует строки | `WHERE total_cents >= 2000` |
| `IS NULL` | Проверяет отсутствие значения | `WHERE customer IS NULL` |
| `ORDER BY` | Сортирует | `ORDER BY total_cents DESC, id` |
| `FETCH FIRST` | Ограничивает выдачу | `FETCH FIRST 5 ROWS ONLY` |
| `OFFSET` | Пропускает строки | `OFFSET 5 ROWS FETCH NEXT 5 ROWS ONLY` |
| `JOIN` | Соединяет совпавшие строки | `JOIN demo_orders b ON b.customer = a.customer` |
| `LEFT JOIN` | Сохраняет строки слева без пары | `LEFT JOIN demo_orders b ON b.id = a.id + 1` |
| `GROUP BY` | Группирует строки | `GROUP BY customer` |
| `COUNT / SUM` | Считает строки и суммы | `COUNT(*), SUM(total_cents)` |
| `HAVING` | Фильтрует группы | `HAVING COUNT(*) > 1` |
| `WITH` | Даёт имя подзапросу | `WITH totals AS (SELECT SUM(total_cents) AS amount FROM demo_orders) SELECT amount FROM totals` |
| `ROW_NUMBER` | Нумерует строки в группе | `ROW_NUMBER() OVER (PARTITION BY customer ORDER BY id)` |
| `NVL` | Подставляет значение вместо `NULL` | `NVL(SUM(total_cents), 0)` |
| `CASE` | Вычисляет значение по условию | `CASE WHEN total_cents >= 2000 THEN 'large' ELSE 'small' END` |
| `TRUNC` | Для `DATE` убирает время при вызове без формата | `TRUNC(ordered_on)` |
| `INSERT` | Добавляет строку | `INSERT INTO demo_orders (order_code, customer, total_cents) VALUES ('A103', 'Anna', 2000)` |
| `UPDATE` | Изменяет строки | `UPDATE demo_orders SET total_cents = 2500 WHERE order_code = 'A101'` |
| `DELETE` | Удаляет строки | `DELETE FROM demo_orders WHERE order_code = 'A102'` |
| `MERGE` | Обновляет совпавшие строки и добавляет новые | Полный пример ниже |
| `CREATE INDEX` | Создаёт индекс | `CREATE INDEX demo_orders_customer_ix ON demo_orders (customer)` |
| `COMMIT` | Фиксирует изменения | `COMMIT` |
| `ROLLBACK` | Откатывает незакреплённые изменения | `ROLLBACK` |

```sql
MERGE INTO demo_orders d
USING (SELECT 'A101' AS code, 'Anna' AS buyer, 2500 AS cents FROM dual) s
ON (d.order_code = s.code)
WHEN MATCHED THEN UPDATE SET d.total_cents = s.cents
WHEN NOT MATCHED THEN INSERT (order_code, customer, total_cents)
VALUES (s.code, s.buyer, s.cents);

SELECT order_code, total_cents FROM demo_orders WHERE order_code = 'A101';
ROLLBACK;
```

Запрос перед откатом покажет `2500`. После отката у `A101` снова будет `1500`. Oracle начинает транзакцию автоматически при первом изменении. DDL вроде `CREATE TABLE`, `DROP TABLE` и `TRUNCATE TABLE` может зафиксировать текущую транзакцию. Перед `UPDATE` и `DELETE` проверьте условие через `SELECT`.

## 6. Типы данных

Тип определяет допустимые значения столбца и правила сравнения. В Oracle меньше типов, чем в PostgreSQL, и у них свои особенности.

| Что хранить | Тип | Комментарий |
|---|---|---|
| Числа, деньги, целые | `NUMBER(p, s)` | Точное десятичное число. `NUMBER(12, 0)` — целое, `NUMBER(10, 2)` — две цифры после запятой |
| Приближённые дроби | `BINARY_FLOAT`, `BINARY_DOUBLE` | Двоичная арифметика с ошибками округления. Не для денег |
| Текст | `VARCHAR2(n CHAR)` | Используйте `VARCHAR2`, а не `VARCHAR`. Без `CHAR` длина `n` считается в байтах. Максимум 4000 байт (до 32767 при включённом расширенном режиме) |
| Дополненный текст | `CHAR(n)` | Дополняет пробелами, обычно не нужен |
| Национальные символы | `NVARCHAR2(n)` | Нужен при нестандартной кодировке базы |
| Большой текст и файлы | `CLOB`, `BLOB` | Для длинных текстов и бинарных данных |
| Дата и время | `DATE` | Хранит дату **и время** до секунды |
| Точное время | `TIMESTAMP`, `TIMESTAMP WITH TIME ZONE`, `TIMESTAMP WITH LOCAL TIME ZONE` | Доли секунды и часовые пояса |
| Интервалы | `INTERVAL YEAR TO MONTH`, `INTERVAL DAY TO SECOND` | Длительности |
| Двоичные значения | `RAW(n)` | Короткие бинарные данные |
| Логические значения | `BOOLEAN` | Есть в Oracle Database 23ai. В 19c используйте `NUMBER(1)` с `CHECK (col IN (0, 1))` |
| JSON | `JSON` | Есть в 21c и новее. В 19c храните в `VARCHAR2` или `CLOB` с `CHECK (col IS JSON)` |

Автоинкремент задаётся свойством `GENERATED BY DEFAULT AS IDENTITY` у `NUMBER`, как в `demo_orders`. Пустая строка `''` в Oracle — это `NULL`.

Преобразование: `CAST(x AS тип)`, а также `TO_NUMBER`, `TO_CHAR`, `TO_DATE`. Для дат всегда задавайте маску или используйте литералы `DATE '2026-01-15'` и `TIMESTAMP '2026-01-15 10:30:00'`. Полагаться на неявные преобразования и на настройки формата сеанса опасно.

```sql
SELECT CAST('42' AS NUMBER) + 1 AS cast_sum,
       TO_DATE('2026-01-15', 'YYYY-MM-DD') + 30 AS plus_30_days,
       TO_CHAR(DATE '2026-01-15', 'DD.MM.YYYY') AS as_text,
       7 / 2 AS div
FROM dual;
```

Результат: `43`, дата на 30 дней позже, `15.01.2026`, `3.5`. В Oracle деление чисел `NUMBER` не отбрасывает дробную часть. Чтобы получить целое, используйте `TRUNC(7 / 2)`.

Типы столбцов своей таблицы покажет словарь данных:

```sql
SELECT column_name, data_type, data_length, data_precision, data_scale
FROM user_tab_columns
WHERE table_name = 'DEMO_ORDERS'
ORDER BY column_id;
```

## 7. Конфигурация

Для подключения по псевдониму храните `tnsnames.ora` в каталоге, заданном переменной `TNS_ADMIN`. В Windows это может быть `C:\Users\имя\oracle-net`, в macOS и Linux — `~/oracle-net`. Для подключения вида `//host:port/service` файл можно опустить.

```text
TRAINING =
  (DESCRIPTION =
    (ADDRESS = (PROTOCOL = TCP)(HOST = localhost)(PORT = 1521))
    (CONNECT_DATA = (SERVICE_NAME = FREEPDB1)))
```

```bash
sql student@TRAINING
```

## 8. Полезные приёмы

- Oracle считает пустую строку `''` значением `NULL`.
- `DATE` хранит дату и время с точностью до секунды. Литерал `DATE '2026-01-01'` задаёт полночь. Для фильтра по дню используйте диапазон от начала этого дня до начала следующего.
- Для стабильной пагинации добавляйте уникальный столбец в конец `ORDER BY`.
- Используйте параметры драйвера для пользовательского ввода. В Oracle именованные параметры часто записываются как `:customer`.
- `DESC demo_orders` показывает столбцы в SQLcl. `USER_TABLES` показывает таблицы вашей схемы.
- `EXPLAIN PLAN FOR` строит оценочный план. Его можно прочитать через `SELECT * FROM TABLE(DBMS_XPLAN.DISPLAY())`.

## 9. Частые ошибки

| Ошибка | Причина | Решение |
|---|---|---|
| `ORA-12514` | Слушатель не знает указанный сервис | Проверьте `SERVICE_NAME` и состояние PDB |
| `ORA-00942` | Таблица отсутствует в схеме или недоступна | Проверьте имя, схему и права `SELECT` |
| `ORA-01950` | У пользователя нет квоты на табличное пространство | Попросите администратора выделить квоту для учебной схемы |
| `ORA-00001` | Повторяется уникальный ключ | Проверьте `order_code` или используйте осознанный `MERGE` |
| `ORA-00979` | Выбранное выражение отсутствует в группировке | Добавьте выражение в `GROUP BY` или агрегируйте его |
| `ORA-01861` | Строка даты не совпадает с ожидаемым форматом | Используйте литерал `DATE` или `TO_DATE` с явной маской |
| `ORA-01722` | Строку нельзя преобразовать в число | Найдите нечисловое значение и используйте `TO_NUMBER` с явной маской |

## 10. Документация

- [Oracle Database SQL Language Reference](https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/)
- [Загрузки SQLcl](https://www.oracle.com/database/sqldeveloper/technologies/sqlcl/download/)
- [Oracle Database Free](https://www.oracle.com/database/free/)
- [Oracle Container Registry](https://container-registry.oracle.com/)
