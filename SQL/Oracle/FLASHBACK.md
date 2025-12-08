
## Предварительные настройки

Возможность отката зависит от настройки базы данных
- **`UNDO_RETENTION`**: определяет, как долго будут храниться данные отмены (UNDO), которые необходимы для Flashback Query и Flashback Table.    
- **Flashback Data Archive (FDA/Total Recall)**: Если требуется хранить исторические данные дольше, чем позволяет `UNDO_RETENTION`, можно настроить FDA.

## Flashback Query (AS OF)

Flashback Query позволяет вам выполнить оператор `SELECT` и увидеть данные в таблице, как они выглядели на определенный момент времени в прошлом (по **метке времени** - `TIMESTAMP`) или по **системному номеру изменения** (`SCN`). Это полезно, чтобы просмотреть старые данные, идентифицировать ошибку или получить старые значения для вставки/обновления.

### Синтаксис
```SQL
SELECT *
FROM имя_таблицы AS OF TIMESTAMP
TO_TIMESTAMP('YYYY-MM-DD HH24:MI:SS', 'YYYY-MM-DD HH24:MI:SS');
```

```SQL
SELECT *
FROM имя_таблицы AS OF SCN 3187302511937; -- Пример SCN
```



### Примеры
#### **Найти старые значения** (например, на 5 минут назад):
```SQL
SELECT *
FROM employees AS OF TIMESTAMP (SYSTIMESTAMP - INTERVAL '5' MINUTE)
WHERE employee_id = 100;
```


#### **Выполнить откат** (используя `UPDATE` с Flashback Query):
```SQL
UPDATE employees tgt
SET (tgt.salary, tgt.commission_pct) =
  (SELECT src.salary, src.commission_pct
   FROM employees AS OF TIMESTAMP (SYSTIMESTAMP - INTERVAL '5' MINUTE) src
   WHERE src.employee_id = tgt.employee_id)
WHERE tgt.employee_id = 100;

COMMIT;
```



## Flashback Table (TO TIMESTAMP/SCN)

Оператор `FLASHBACK TABLE` **откатывает всю таблицу** к предыдущему состоянию (по времени или SCN). Это более быстрый способ, чем восстановление из резервной копии.

 **Важные условия:*
- Для таблицы должна быть включена функция **Row Movement** (перемещение строк): `ALTER TABLE имя_таблицы ENABLE ROW MOVEMENT;`
- Должны быть доступны данные **UNDO** за требуемый период времени (определяется параметром `UNDO_RETENTION`).

### Примеры

#### Откат таблицы до определенного времени:
```SQL
FLASHBACK TABLE имя_таблицы
TO TIMESTAMP TO_TIMESTAMP('2023-11-20 15:00:00', 'YYYY-MM-DD HH24:MI:SS');
```

#### Откат таблицы до определенного SCN:
```SQL
FLASHBACK TABLE имя_таблицы
TO SCN 3187302511937;
```


## Flashback Drop (Для удаленных таблиц)
Если вы случайно удалили таблицу с помощью `DROP TABLE`, вы можете восстановить ее, используя **Flashback Drop**, если она еще находится в **Recycle Bin** (корзине):

```SQL
FLASHBACK TABLE имя_таблицы
TO BEFORE DROP;
```