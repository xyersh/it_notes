Оператор `MERGE` (также известен как **UPSERT** — от _UP_date + in_SERT_) позволяет **объединить** логику `INSERT`, `UPDATE` и (начиная с Oracle 10g) даже `DELETE` в одном SQL-запросе.  
Он особенно полезен при **синхронизации** двух таблиц: например, загрузка изменений из промежуточной таблицы-источника/SQL-выражения в целевую таблицу.

## Синтаксис MERGE в Oracle
```sql
MERGE INTO target_table t
USING source_table_or_query s
ON (join_condition)
WHEN MATCHED THEN
    UPDATE SET ...
    [DELETE WHERE ...]
WHEN NOT MATCHED THEN
    INSERT (columns) VALUES (...)
    [WHERE ...];
```

- **`target_table`** — таблица, в которую будут вноситься изменения.
- **`source_table_or_query`** — источник данных (может быть таблицей, представлением или подзапросом).
- **`ON (join_condition)`** — условие соединения между целевой и исходной таблицами.
- **`WHEN MATCHED`** — действия, если запись найдена в целевой таблице.
- **`WHEN NOT MATCHED`** — действия, если запись не найдена.

### Важные замечания
- Условие в `ON` должно быть **уникальным** по целевой таблице, иначе может возникнуть ошибка `ORA-30926: unable to get a stable set of rows`.
- Все действия (`UPDATE`, `INSERT`, `DELETE`) **необязательны**, но должен быть хотя бы один `WHEN MATCHED` или `WHEN NOT MATCHED`.
- Можно использовать `WHERE` в блоках `UPDATE` и `INSERT` для дополнительной фильтрации.
- `MERGE` — это **одно транзакционное действие**, что делает его эффективным и безопасным.

## ПРИМЕРЫ

#### 1. Простой UPSERT
Допустим, имеются таблицы:
```sql
-- Таблица сотрудников (основная)
CREATE TABLE employees (
    id NUMBER PRIMARY KEY,
    name VARCHAR2(100),
    salary NUMBER
);

-- Таблица обновлений
CREATE TABLE emp_updates (
    id NUMBER,
    name VARCHAR2(100),
    salary NUMBER
);
```

Заполняем данными
```sql
INSERT INTO employees VALUES (1, 'Alice', 5000);
INSERT INTO employees VALUES (2, 'Bob', 6000);
INSERT INTO employees VALUES (3, 'Billy', 3500);

INSERT INTO emp_updates VALUES (1, 'Alice', 5500);   -- Обновление
INSERT INTO emp_updates VALUES (7, 'Charlie', 7000); -- Новый сотрудник
```


Теперь синхронизируем:
```sql
MERGE INTO employees e        -- сюда пишем изменения
USING emp_updates u           -- используя данные из emp_updates
ON (e.id = u.id)              -- объединяем employees и emp_updates
WHEN MATCHED THEN             -- если соо
    UPDATE SET e.salary = u.salary
WHEN NOT MATCHED THEN
    INSERT (id, name, salary) VALUES (u.id, u.name, u.salary);
```

Результат:
- Зарплата Alice изменится в employees на  5500.
- Charlie будет добавлен в employees.

#### 2: MERGE с DELETE
Предположим, мы хотим **удалить** сотрудников, чья зарплата стала ниже 4000:
```sql
MERGE INTO employees e
USING emp_updates u
ON (e.id = u.id)
WHEN MATCHED THEN
    UPDATE SET e.salary = u.salary 
    DELETE WHERE (u.salary < 4000)
WHEN NOT MATCHED THEN
    INSERT (id, name, salary) VALUES (u.id, u.name, u.salary);
```

Результат:
- Зарплата Alice изменится в employees на  5500.
- Charlie будет добавлен в employees.
- `Billy` будет удален из `employees` так как его ЗП < 4000

#### 3: Использование подзапроса как источника
```sql
MERGE INTO employees e
USING (
    SELECT 4 AS id, 'David' AS name, 8000 AS salary FROM dual
) src
ON (e.id = src.id)
WHEN NOT MATCHED THEN
    INSERT (id, name, salary) VALUES (src.id, src.name, src.salary);
```