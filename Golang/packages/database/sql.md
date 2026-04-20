Пакет `database/sql` — это стандартный интерфейс Go для работы с реляционными базами данных. Он **не содержит драйверов**, а предоставляет унифицированный API. Драйверы (PostgreSQL, MySQL, SQLite и др.) подключаются отдельно через side-effect импорты.

## 1. Архитектура и ключевые принципы

| Концепция    | Описание                                                                                                                                                                                                                                                                   |
| ------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `DSN`        | **DSN (Data Source Name — Имя источника данных)** — это структурированная строка или файл, содержащий всю необходимую информацию для подключения к базе данных. Она сообщает приложению, какой драйвер использовать, где находится сервер, имя базы данных, логин и пароль |
| `sql.DB`     | Не одно соединение, а **пул соединений**. Безопасен для конкурентного использования. Должен создаваться один раз на приложение.                                                                                                                                            |
| `sql.Open()` | Только валидирует драйвер и создаёт объект пула. **Не открывает сеть**.                                                                                                                                                                                                    |
| `db.Ping()`  | Первое реальное подключение к БД. Рекомендуется вызывать сразу после `Open`.                                                                                                                                                                                               |
| Драйверы     | Регистрируются через `sql.Register()`. В коде импортируются с `_` для инициализации.                                                                                                                                                                                       |


## 2. Подключение и настройка пула соединений

```go
package main

import (
	"database/sql"
	"fmt"
	"log"
	"time"

	_ "github.com/mattn/go-sqlite3" // Регистрация драйвера. Замените на свой (pq, mysql и т.д.)
)

func main() {
	// sql.Open создаёт пул, но не подключается к БД
	db, err := sql.Open("sqlite3", "./test.db")
	if err != nil {
		log.Fatalf("ошибка инициализации: %v", err)
	}
	// Обязательно закрываем пул при завершении приложения
	defer db.Close()

	// Явная проверка доступности БД (открывает первое соединение)
	if err := db.Ping(); err != nil {
		log.Fatalf("БД недоступна: %v", err)
	}

	// --- Настройка пула соединений (рекомендуется для production) ---
	
	// Максимальное количество открытых соединений к БД
	db.SetMaxOpenConns(25)
	
	// Максимальное количество idle-соединений, которые пул держит готовыми
	db.SetMaxIdleConns(25)
	
	// Максимальное время жизни соединения (предотвращает утечки и stale-соединения)
	db.SetConnMaxLifetime(5 * time.Minute)
	
	// Максимальное время простоя соединения перед закрытием
	db.SetConnMaxIdleTime(2 * time.Minute)

	fmt.Println("✅ Подключение установлено, пул настроен")
}
```


## Чтение данных: `Query` и `QueryRow`

### Множественный выбор (`Query`)
Используется, когда ожидается **0 или более строк**.
```go
func readUsers(db *sql.DB) error {
	// ? — позиционный параметр. В PostgreSQL используется $1, $2
	rows, err := db.Query("SELECT id, name, age FROM users WHERE age > ?", 18)
	if err != nil {
		return fmt.Errorf("ошибка запроса: %w", err)
	}
	// Обязательно закрываем rows, иначе соединение не вернётся в пул!
	defer rows.Close()

	for rows.Next() {
		var id int
		var name string
		var age int
		// Scan читает значения из текущей строки в переменные по ссылке
		if err := rows.Scan(&id, &name, &age); err != nil {
			return fmt.Errorf("ошибка сканирования: %w", err)
		}
		fmt.Printf("ID: %d, Имя: %s, Возраст: %d\n", id, name, age)
	}

	// Проверяем ошибки, возникшие во время итерации (сеть, парсинг и т.д.)
	if err := rows.Err(); err != nil {
		return fmt.Errorf("ошибка итерации: %w", err)
	}
	return nil
}
```

### Единичный выбор (`QueryRow`)
Оптимизирован для случаев, когда ожидается **ровно одна строка**.
```go
func getUserById(db *sql.DB, id int) (string, error) {
	var name string
	// Scan сразу пытается прочитать первую строку.
	// Если строк нет → возвращает sql.ErrNoRows
	// Если строк >1 → ошибка (зависит от драйвера)
	err := db.QueryRow("SELECT name FROM users WHERE id = ?", id).Scan(&name)
	
	if err == sql.ErrNoRows {
		return "", fmt.Errorf("пользователь %d не найден", id)
	}
	if err != nil {
		return "", fmt.Errorf("ошибка запроса: %w", err)
	}
	return name, nil
}
```

## Вставка и обновление: `Exec`
`Exec` используется для DML-запросов, которые не возвращают строки (`INSERT`, `UPDATE`, `DELETE`, DDL).

```go
func createUser(db *sql.DB, name string, age int) error {
	res, err := db.Exec("INSERT INTO users (name, age) VALUES (?, ?)", name, age)
	if err != nil {
		return fmt.Errorf("ошибка вставки: %w", err)
	}

	// Получаем ID новой записи (не поддерживается всеми драйверами/БД)
	newID, err := res.LastInsertId()
	if err != nil {
		log.Printf("LastInsertId не поддерживается: %v", err)
	}

	affected, _ := res.RowsAffected()
	fmt.Printf("✅ Создан пользователь ID=%d, затронуто строк: %d\n", newID, affected)
	return nil
}
```

## Работа с `NULL`-значениями
Стандартные типы Go (`string`, `int`, `time.Time`) не могут представлять `NULL`. Для этого в пакете есть типы-обёртки: `sql.NullString`, `sql.NullInt64`, `sql.NullTime`, `sql.NullFloat64`, `sql.NullBool`.
```go
func readNullableField(db *sql.DB, id int) error {
	var name sql.NullString
	// Если в БД NULL, Scan установит name.Valid = false
	err := db.QueryRow("SELECT name FROM users WHERE id = ?", id).Scan(&name)
	if err != nil && err != sql.ErrNoRows {
		return err
	}

	if name.Valid {
		fmt.Println("Имя:", name.String)
	} else {
		fmt.Println("Имя: NULL")
	}
	return nil
}
```


## Подготовленные выражения (`Prepared Statements`)

Подготовка запроса на сервере БД ускоряет многократное выполнение и защищает от SQL-инъекций.

```go
func usePreparedStmt(db *sql.DB) error {
	// Запрос компилируется и кэшируется на стороне БД
	stmt, err := db.Prepare("INSERT INTO users (name, age) VALUES (?, ?)")
	if err != nil {
		return err
	}
	defer stmt.Close() // Возвращает ресурсы драйверу

	// Многократное быстрое выполнение
	_, err = stmt.Exec("Alice", 30)
	if err != nil { return err }
	
	_, err = stmt.Exec("Bob", 25)
	if err != nil { return err }

	return nil
}
```


## Транзакции
Транзакции обеспечивают атомарность: либо всё, либо ничего.


```go
func transferFunds(db *sql.DB, fromID, toID int, amount int64) error {
	// Начинаем транзакцию
	tx, err := db.Begin()
	if err != nil {
		return err
	}
	
	// Паттерн безопасности: если Commit не вызовется, Rollback откатит изменения.
	// После успешного Commit Rollback становится no-op.
	defer tx.Rollback()

	// Списание
	_, err = tx.Exec("UPDATE accounts SET balance = balance - ? WHERE id = ?", amount, fromID)
	if err != nil {
		return fmt.Errorf("ошибка списания: %w", err)
	}

	// Зачисление
	_, err = tx.Exec("UPDATE accounts SET balance = balance + ? WHERE id = ?", amount, toID)
	if err != nil {
		return fmt.Errorf("ошибка зачисления: %w", err)
	}

	// Фиксируем транзакцию
	if err := tx.Commit(); err != nil {
		return fmt.Errorf("ошибка коммита: %w", err)
	}

	return nil
}
```


## Контекст и таймауты (`context.Context`)
В production все запросы должны принимать `context` для отмены, таймаутов и трассировки.
```go
import "context"

func queryWithContext(db *sql.DB) error {
	// Таймаут 3 секунды на весь запрос
	ctx, cancel := context.WithTimeout(context.Background(), 3*time.Second)
	defer cancel() // Обязательно! Иначе возможна утечка горутин/таймеров

	// QueryContext автоматически прервёт запрос при отмене контекста
	rows, err := db.QueryContext(ctx, "SELECT id, name FROM users")
	if err != nil {
		// ctx.Err() == context.DeadlineExceeded если таймаут
		return err
	}
	defer rows.Close()

	// ... обработка rows.Next()
	return nil
}
```


## Мониторинг пула и production-практики
```go
func monitorPool(db *sql.DB) {
	stats := db.Stats()
	fmt.Printf("Открыто соединений: %d\n", stats.OpenConnections)
	fmt.Printf("Используется сейчас: %d\n", stats.InUse)
	fmt.Printf("В пуле (idle): %d\n", stats.Idle)
	fmt.Printf("Ожидает соединения: %d\n", stats.WaitCount)
	fmt.Printf("Среднее время ожидания: %v\n", stats.WaitDuration)
}
```


## DSN при подключении

### `mattn/go-sqlite3`
SQLite поддерживает **URI-формат** строки подключения. Для передачи параметров обязательно использовать префикс `file:`:

```go
file:путь/к/базе.db?параметр1=значение1&параметр2=значение2
```

#### Таблица основных параметров
|Категория|Параметр|Допустимые значения|Описание|
|---|---|---|---|
|**Режим доступа**|`mode`|`ro`, `rw`, `rwc`, `memory`|`ro` – только чтение, `rwc` – чтение/запись/создание, `memory` – БД в ОЗУ|
|**Кэш**|`cache`|`private` (по умолч.), `shared`|`shared` обязателен для безопасной многопоточной работы с `:memory:`|
|**Журнал транзакций**|`_journal_mode`|`delete`, `truncate`, `persist`, `memory`, `wal`, `off`|`wal` (Write-Ahead Logging) рекомендуется для конкурентных записей|
|**Синхронизация с диском**|`_synchronous`|`full`, `normal`, `off`|`normal` – баланс скорости/безопасности, `full` – максимальная надёжность|
|**Таймаут блокировок**|`_busy_timeout`|целое число (миллисекунды)|Сколько ждать, если БД заблокирована. Решает `database is locked`|
|**Внешние ключи**|`_foreign_keys`|`1`/`true` или `0`/`false`|⚠️ В SQLite **выключены по умолчанию**. Всегда включайте в production!|
|**Уровень блокировки транзакций**|`_txlock`|`deferred`, `immediate`, `exclusive`|`deferred` (по умолч.) блокирует только при первой записи|
|**Часовой пояс**|`_loc`|`auto`, `utc`, `Europe/Moscow`|Как парсить/хранить `DATETIME` без явного указания TZ|
|**Размер кэша страниц**|`_cache_size`|целое число (страницы, обычно 1K=2048 байт)|Увеличивает RAM-буфер для частых запросов|
|**Memory-mapped I/O**|`_mmap_size`|байты (например `268435456` = 256MB)|Прямое отображение файла в память. Ускоряет чтение в 2-5x|
|**Безопасное удаление**|`_secure_delete`|`1`, `fast`, `0`|`1` – заполняет удалённые данные нулями (безопасно, но медленнее)|
|**Только чтение**|`_query_only`|`1` или `0`|Запрещает любые записи на уровне драйвера|

#### Примеры DSN строк
|Задача|DSN|
|---|---|
|**Базовое подключение**|`file:app.db?mode=rwc`|
|**БД в памяти (многопоточная)**|`file::memory:?cache=shared`|
|**Production-конфиг (WAL + FK + таймаут)**|`file:app.db?mode=rwc&_journal_mode=WAL&_synchronous=NORMAL&_busy_timeout=5000&_foreign_keys=1`|
|**Только чтение + кэш**|`file:analytics.db?mode=ro&cache=private&_cache_size=-2000`|
|**Высокая производительность чтения**|`file:data.db?_mmap_size=268435456&_cache_size=-4000`|
### `modernc.org/sqlite`
Все параметры передаются как query-параметры в строке подключения. Поскольку драйвер не парсит стандартные SQLite URI-параметры, **все PRAGMA-настройки передаются через `_pragma`**.

#### Таблица основных параметров
|Параметр|Тип|Значения по умолчанию|Описание|
|---|---|---|---|
|**`_pragma`**|строка (повторяемый)|—|Выполняет `PRAGMA <значение>` при подключении. Можно указывать несколько раз.<br><br>pkg.go.dev|
|**`_time_format`**|строка|`"2006-01-02 15:04:05.999999999"` (Go `String()`)|Формат записи `time.Time` в БД: `"sqlite"` (формат 4 с под-секундами и таймзоной) или `"datetime"` (формат 3, вывод SQLite `datetime()`).<br><br>pkg.go.dev|
|**`_time_integer_format`**|строка|— (хранение как строка)|Хранить время как INTEGER: `"unix"`, `"unix_milli"`, `"unix_micro"`, `"unix_nano"` (секунды/мс/мкс/нс с эпохи Unix). При указании `_time_format` игнорируется.|
|**`_inttotime`**|bool|`false`|Конвертировать INTEGER-поля в `time.Time` при чтении, если колонка имеет тип `DATE`/`DATETIME`/`TIMESTAMP`.|
|**`_texttotime`**|bool|`false`|Возвращать `time.Time` вместо `string` при `ColumnTypeScanType()` для TEXT-колонок с типами даты/времени.|
|**`_timezone`**|строка (IANA)|`nil` (без конвертации)|Таймзона для всех операций с временем (например, `"UTC"`, `"Europe/Moscow"`). При записи — конвертирует `time.Time` в эту зону перед форматированием; при чтении — интерпретирует строки без таймзоны как находящиеся в этой зоне.<br><br>pkg.go.dev|
|**`_txlock`**|строка|`"deferred"`|Режим блокировки при `BEGIN`: `"deferred"`, `"immediate"`, `"exclusive"` (регистр не важен).<br><br>riverqueue.com|
|**`vfs`**|строка|—|Имя VFS-модули для доступа к файлу (расширенный сценарий).|

```go
// Production-конфиг через _pragma:
dsn := "file:app.db" +
    "?_pragma=foreign_keys(1)" +
    "&_pragma=journal_mode(WAL)" +
    "&_pragma=synchronous(NORMAL)" +
    "&_pragma=busy_timeout(5000)" +
    "&_pragma=cache_size(-2000)" +
    "&_timezone=UTC" +
    "&_time_format=sqlite"

db, err := sql.Open("sqlite", dsn)
```

#### Примеры DSN строк
|Задача|DSN|
|---|---|
|**Базовое подключение**|`file:app.db`|
|**БД в памяти (многопоточная)**|`file::memory:?_pragma=cache_shared(1)`|
|**Production-конфиг (время + таймзона)**|`file:prod.db?_pragma=foreign_keys(1)&_pragma=journal_mode(WAL)&_timezone=UTC&_time_format=sqlite`|
|**Хранение времени как Unix-timestamp**|`file:logs.db?_time_integer_format=unix&_inttotime=1`|
|**Немедленная блокировка транзакций**|`file:queue.db?_txlock=immediate`<br><br>riverqueue.com|
|**Только чтение + кэш**|`file:analytics.db?_pragma=journal_mode(MEMORY)&_pragma=cache_size(-4000)`|


## СОВЕТЫ
1. Всегда используйте `defer rows.Close()` и проверяйте `rows.Err()`.
2. Никогда не оставляйте `*sql.DB` открытой на уровне функций/хендлеров.
3. Используйте `context.Context` для всех запросов.
4. Настраивайте `SetMaxOpenConns` под нагрузку и лимиты БД.
5. Избегайте `SELECT *` — явно указывайте колонки для стабильности `Scan`.
6. Для массовых вставок используйте `COPY`/`LOAD DATA` или батчинг, а не цикл `Exec`.
7. Логгируйте `sql.ErrNoRows` отдельно от сетевых/синтаксических ошибок.


## РАБОЧИЙ ПРИМЕР
```go
package main

import (
	"context"
	"database/sql"
	"fmt"
	"log"
	"time"

	_ "github.com/mattn/go-sqlite3"
)

func main() {
	db, err := sql.Open("sqlite3", ":memory:") // in-memory БД для теста
	if err != nil {
		log.Fatal(err)
	}
	defer db.Close()

	if err := db.Ping(); err != nil {
		log.Fatal("DB ping failed:", err)
	}

	// 1. Создание таблицы
	_, err = db.Exec(`CREATE TABLE users (
		id INTEGER PRIMARY KEY AUTOINCREMENT,
		name TEXT,
		email TEXT,
		age INTEGER
	)`)
	if err != nil {
		log.Fatal(err)
	}

	// 2. Вставка с транзакцией
	tx, _ := db.Begin()
	defer tx.Rollback()
	
	stmt, _ := tx.Prepare("INSERT INTO users (name, email, age) VALUES (?, ?, ?)")
	defer stmt.Close()
	
	_, _ = stmt.Exec("Анна", "anna@example.com", 28)
	_, _ = stmt.Exec("Иван", NULL, 35) // NULL в SQL
	_, _ = stmt.Exec("Мария", NULL, NULL) // Проверим работу с NULL
	
	if err := tx.Commit(); err != nil {
		log.Fatal("Commit failed:", err)
	}

	// 3. Чтение с обработкой NULL
	ctx, cancel := context.WithTimeout(context.Background(), 2*time.Second)
	defer cancel()

	rows, err := db.QueryContext(ctx, "SELECT id, name, email, age FROM users")
	if err != nil {
		log.Fatal(err)
	}
	defer rows.Close()

	for rows.Next() {
		var id int
		var name sql.NullString
		var email sql.NullString
		var age sql.NullInt64

		if err := rows.Scan(&id, &name, &email, &age); err != nil {
			log.Fatal("Scan error:", err)
		}

		emailStr := "<нет>"
		if email.Valid { emailStr = email.String }
		
		ageStr := "?"
		if age.Valid { ageStr = fmt.Sprint(age.Int64) }

		fmt.Printf("ID:%d | Имя:%s | Email:%s | Возраст:%s\n",
			id, name.String, emailStr, ageStr)
	}
	if err := rows.Err(); err != nil {
		log.Fatal("Rows iteration error:", err)
	}

	fmt.Println("\n✅ Все операции выполнены успешно")
}
```


