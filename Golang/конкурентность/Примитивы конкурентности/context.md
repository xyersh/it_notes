
### Используемый пакет: context
### Что это такое?
контексты в Go предоставляет мощный механизм для управления **сигналами отмены (cancellation signals)**, **тайм-аутами (timeouts)**, **дедлайнами (deadlines)** и передачи **данных в рамках запроса (request-scoped values)** через границы API и между горутинами. Это особенно важно в распределенных системах и при работе с сетевыми запросами.

Основные задачи `context`:
- **Отмена (Cancellation):** Позволяет одной горутине сигнализировать другой (или нескольким), что выполняемая работа больше не нужна.
- **Тайм-ауты и Дедлайны:** Автоматическая отмена операции по истечении заданного времени или к определенному моменту времени.
- **Передача значений:** Передача специфичных для запроса данных (например, ID трассировки, информация об аутентификации) вниз по стеку вызовов.

### Функции и методы контекстов

#### Методы

| Метод                                                            | Описание                                                                                                                                                                     |
| ---------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `(ctx context.Context) Deadline() (deadline time.Time, ok bool)` | Возвращает время, когда контекст будет отменен, если оно установлено. Если дедлайн не установлен, возвращает  `ok = false`.                                                  |
| `(ctx context.Context) Done() <-chan struct{}`                   | Возвращает канал, который закрывается, когда контекст отменен.  Если контекст никогда не отменяется, возвращает `nil`                                                        |
| `(ctx context.Context) Err() error`                              | Возвращает ошибку, если контекст отменен. Если контекст не отменен, возвращает `nil`. Среди возвращаемых ошибок могут быть: `context.Canceled` , `context.DeadlineExceeded`. |
| `(ctx context.Context) Value(key any) any`                       | Возвращает значение, связанное с ключом `key` в контексте.  Если ключ не найден, возвращает `nil`.                                                                           |
|                                                                  |                                                                                                                                                                              |
#### Функции

| Функция                                    | Описание                                                                                                                                                         |
| ------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `context.Cause(ctx context.Context) error` Возвращает ту самую **исходную ошибку**, которую разработчик передал в функцию отмены. Возвращает более тетальные причины отмены контекста, чем метод `Err()`.  .  |


#### `context.Cause(ctx context.Context) error `
принцип возврата ошибки:
	- Если контекст был отменен из-за дедлайна (`WithDeadline` или `WithTimeout`), то `Cause` вернет `DeadlineExceeded`.
	- Если контекст был отменен функцией  `CancelFunc`, то  `Cause()` вернет `context.Canceled`  
	- Если контекст был отменен функцией `cancel() CancelCauseFunc`  (из `context.WithCancelCause`) с указанием конкретной ошибки `errCause` параметром ф-ции `cancel()` , то `Cause` вернет `errCause`.
	- Если был отменен контекст типа `WithDeadlineCause` или `WithTimeoutCause` с указанием  ошибки `errCause` в параметре - то возвращается `errCause`
	- Если контекст еще не отменен, `Cause` вернет `nil`.

###  Виды контекстов

- **`context.Background()`** - Возвращает пустой контекст, который никогда не отменяется
- **`context.TODO()`** - Аналогичен `context.Background()`, но используется, когда не ясно, какой контекст использовать.
-  **`WithCancel(parent Context)(ctx Context, cancel CancelFunc)`** - Создает новый контекст, который может быть отменен с помощью функции `cancel`. Когда вызывается `cancel`, контекст отменяется, и канал `Done()` закрывается.
- **`WithTimeout(parent Context, timeout time.Duration)(Context, CancelFunc)`** - Аналогичен `WithDeadline`, но принимает длительность времени `timeout` вместо конкретного времени.  Контекст отменяется через указанное время.
- **`WithDeadline(parent Context, deadline time.Time)(Context, CancelFunc)`**  - Создает контекст, который автоматически отменяется в указанное время `d`. Если время `d` уже прошло, контекст отменяется сразу.
- **`WithValue(parent Context, key any, value any)(Context)`** - Создает контекст, который содержит значение `val`, связанное с ключом `key`.  Значения в контексте должны использоваться для передачи данных, которые относятся к запросу, а не для передачи необязательных параметров. 
- **`WithCancelCause(parent Context)(Context, CancelCauseFunc)`** -во многом похож на контекст **`context.WithCancel`** за исключением того, что возвращает функцию отмены с типом  `CancelCauseFunc` а не `CancelFunc`. CancelCauseFunc контракт `func(cause error)`что позволяет передавать  структуру ошибки с причинами отмены.
- **`WithDeadlineCause(parent Context, d time.Time, cause error)(Context, CancelFunc)`** - аналогичен `WithDeadline` но дополнительным параметром принимает объект ошибки
- **`WithTimeoutCause(parent Context, timeout time.Duration, cause error) (Context, CancelFunc)`** - аналогичен `WithTimeout` но дополнительным параметром принимает объект ошибки `cause`
- **`AfterFunc(ctx Context, f func()) (stop func() bool)`** - контекст выполняет функцию `f` после своего закрытия. Описание:
	- `ctx`: Контекст, за завершением которого мы следим.
	- `f`: Функция без аргументов и без возвращаемых значений, которая будет выполнена после завершения `ctx`.
	- `stop`: Возвращаемая функция. Вызов `stop()` предотвращает вызов `f`. Если `f` уже была вызвана или была запланирована к вызову, или если `ctx` еще не завершен и `f` успешно "остановлена", `stop()` возвращает `true`. Если `f` уже была вызвана (или `ctx` был завершен и `f` вот-вот будет вызвана, но еще не завершилась), `stop()` возвращает `false`.
	`AfterFunc` удобна для выполнения действий по очистке или уведомлений без необходимости вручную запускать горутину с `select` на `ctx.Done()`. Это упрощает код и уменьшает вероятность ошибок.

## РЕКОМЕНДАЦИИ ИСПОЛЬЗОВАНИЯ КОНТЕКСТОВ

- **`context.Background()` только в `main` или на верхнем уровне:** Для инициализации корневого контекста.

- **`context.TODO()` как временная мера:** Когда вы еще не решили, какой контекст использовать, или функция находится в процессе рефакторинга.

- **Контекст распространяется вниз по стеку вызовов.** Отмена родительского контекста отменяет все дочерние.

*  **Передавать контекст первым парамeтром**  
```go
func ProcessData(ctx context.Context, data []byte) error // ✅ Good
func ProcessData(data []byte, ctx context.Context) error // ❌ Bad
```

* **Всегда вызывать cancel() для предотвращения утечки памяти**
```go
ctx, cancel := context.WithTimeout(parent, 30*time.Second)
defer cancel() // ✅ Always do this
```

- **Использовать канал Done() только для чтения**

* **Проверять ctx.Done() в циклах и долго работающих операциях**
```go
for i := 0; i < len(items); i++ {
    select {
    case <-ctx.Done():
        return ctx.Err()
    default:
    }
    processItem(items[i])
}
```

* **Использовать контексты для передачи только специфичных для запроса  данных**
```go
ctx = context.WithValue(ctx, "traceID", "abc123")
ctx = context.WithValue(ctx, "userID", "user456")
```


* **Получать дочерние контексты с использованием родительских**
```go
childCtx, cancel := context.WithTimeout(parentCtx, 10*time.Second)
```


* **НЕ хранить контексты в атрибутах структур**
```go
// ❌ Bad - context stored in struct
type Server struct {
    ctx context.Context
}

// ✅ Good - context passed as parameter
func (s *Server) ProcessRequest(ctx context.Context) error
```

* **НЕ передавать nil-контексты**
```golang
ProcessData(nil, data) // ❌ Bad
ProcessData(context.Background(), data) // ✅ Good
```

* **НЕ использовать контексты для опционных параметров**
```go
// ❌ Bad - using context for config
ctx = context.WithValue(ctx, "retryCount", 3)

// ✅ Good - use struct for config
type Config struct {
    RetryCount int
}
func ProcessData(ctx context.Context, cfg Config) error
```


* **НЕ игнорировать отмену контекста, проверять ctx.Err() после ctx.Done()**
```go
// ❌ Bad - ignoring context
func doWork(ctx context.Context) {
    for i := 0; i < 1000; i++ {
        // No context checking
        time.Sleep(100 * time.Millisecond)
    }
}

// ✅ Good - respecting context
func doWork(ctx context.Context) error {
    for i := 0; i < 1000; i++ {
        select {
        case <-ctx.Done():
            return ctx.Err()
        default:
        }
        time.Sleep(100 * time.Millisecond)
    }
    return nil
}
```


## ОБЩИЕ ОШИБКИ ИСПОЛЬЗОВАНИЯ КОНТЕКСТОВ

- **Утечки памяти из-за игнорирования cancel()**
```go 
// ❌ Memory leak - cancel not called
func badExample() {
    ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
    // cancel() never called - goroutine and timer leak!
    doWork(ctx)
}

// ✅ Fixed - always call cancel
func goodExample() {
    ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
    defer cancel() // Always call cancel
    doWork(ctx)
}
```

- **Условия гонки со значениями контекстов**
```go
// ❌ Race condition - value might change
func badExample(ctx context.Context) {
    go func() {
        userID := ctx.Value("userID").(string) // Might panic if nil
        processUser(userID)
    }()
}

// ✅ Safe value extraction
func goodExample(ctx context.Context) {
    userIDValue := ctx.Value("userID")
    if userIDValue == nil {
        return // Handle missing value
    }
    userID, ok := userIDValue.(string)
    if !ok {
        return // Handle wrong type
    }
    
    go func() {
        processUser(userID)
    }()
}
```

- **Проблемы наследования контекста**
```go
// ❌ Bad - creating independent contexts
func badChain() {
    ctx1, cancel1 := context.WithTimeout(context.Background(), 10*time.Second)
    defer cancel1()
    
    ctx2, cancel2 := context.WithTimeout(context.Background(), 5*time.Second) // Independent!
    defer cancel2()
    
    doWork(ctx2) // Won't inherit ctx1's cancellation
}

// ✅ Good - proper context chaining
func goodChain() {
    ctx1, cancel1 := context.WithTimeout(context.Background(), 10*time.Second)
    defer cancel1()
    
    ctx2, cancel2 := context.WithTimeout(ctx1, 5*time.Second) // Inherits from ctx1
    defer cancel2()
    
    doWork(ctx2) // Will be cancelled when ctx1 OR ctx2 times out
}
```




## ПРИМЕРЫ
####  WithCancel:
	
```go
package main

import (
	"context"
	"fmt"
	"time"
)

func main() {
	ctx, cancel := context.WithCancel(context.Background())
	go monitor(ctx)

	time.Sleep(2 * time.Second) // Симулируем некоторую работу
	cancel()                    // Отменяем контекст
	time.Sleep(1 * time.Second)
}

func monitor(ctx context.Context) {
	for {
		select {
		case <-ctx.Done():
			fmt.Println("Monitoring stopped:", ctx.Err())
			return
		default:
			fmt.Println("Monitoring in progress...")
			time.Sleep(500 * time.Millisecond)
		}
	}
}
```

#### WithDeadline:
```go
package main

import (
	"context"
	"fmt"
	"time"
)

func operationWithDeadline(ctx context.Context) {
	n := 0
	for {
		select {
		case <-time.After(500 * time.Millisecond):
			n++
			fmt.Printf("Работаем... %d\n", n)
		case <-ctx.Done():
			fmt.Printf("Операция прервана (дедлайн): %v\n", ctx.Err())
			return
		}
	}
}

func main() {
	parentCtx := context.Background()
	deadlineTime := time.Now().Add(2 * time.Second) // Дедлайн через 2 секунды

	ctx, cancel := context.WithDeadline(parentCtx, deadlineTime)
	defer cancel() // Важно!

	fmt.Printf("Запускаем операцию с дедлайном в %s\n", deadlineTime.Format(time.RFC3339))
	go operationWithDeadline(ctx)

	// Ждем, пока не сработает дедлайн
	<-ctx.Done()
	fmt.Printf("Дедлайн достигнут в main. Причина: %v\n", ctx.Err())
	time.Sleep(100 * time.Millisecond) // Дать горутине время вывести сообщение
}
```

#### WithTimeout:
```go
package main

import (
	"context"
	"fmt"
	"time"
)

func main() {
	// Создаём контекст с таймаутом 2 секунды
	ctx, cancel := context.WithTimeout(context.Background(), 2*time.Second)
	defer cancel()

	// Передаём контекст в функцию
	result := make(chan string)
	go performTask(ctx, result)

	select {
	case res := <-result:
		fmt.Println("Result:", res)
	case <-ctx.Done():
		fmt.Println("Operation timed out:", ctx.Err())
	}
}

func performTask(ctx context.Context, result chan<- string) {
	// Симулируем длительную задачу
	time.Sleep(3 * time.Second)
	select {
	case <-ctx.Done():
		// Если контекст завершён, выходим
		fmt.Println("Task canceled:", ctx.Err())
	default:
		// Отправляем результат
		result <- "Task completed successfully!"
	}
}
```

#### WithValue:
```go
package main

import (
	"context"
	"fmt"
)

// Определяем пользовательский тип для ключа контекста
type keyType string

const requestIDKey keyType = "requestID"
const userKey keyType = "userID"

func processRequest(ctx context.Context) {
	reqID, ok := ctx.Value(requestIDKey).(string)
	if !ok {
		reqID = "unknown"
	}
	fmt.Printf("Обработка запроса с ID: %s\n", reqID)

	userID, ok := ctx.Value(userKey).(int)
	if ok {
		fmt.Printf("  Пользователь ID: %d\n", userID)
	}

	// Представим, что мы вызываем другую функцию, передавая контекст дальше
	logSomething(ctx, "Начало обработки данных")
}

func logSomething(ctx context.Context, message string) {
	reqID, _ := ctx.Value(requestIDKey).(string) // Мы ожидаем, что он там есть
	fmt.Printf("[LOG ReqID: %s] %s\n", reqID, message)
}

func main() {
	parentCtx := context.Background()

	// Добавляем requestID
	ctxWithReqID := context.WithValue(parentCtx, requestIDKey, "abc-123-xyz")

	// Добавляем userID поверх предыдущего контекста
	ctxWithUser := context.WithValue(ctxWithReqID, userKey, 42)

	processRequest(ctxWithUser)

	fmt.Println("\n--- Другой запрос без userID ---")
	ctxAnotherReq := context.WithValue(parentCtx, requestIDKey, "def-456-uvw")
	processRequest(ctxAnotherReq)
}
```


#### WithCancelCause
```go
package main

import (
	"context"
	"errors"
	"fmt"
	"time"
)

func operationWithSpecificCause(ctx context.Context) {
	select {
	case <-time.After(5 * time.Second): // Имитация долгой работы
		fmt.Println("Операция (operationWithSpecificCause) успешно завершена.")
	case <-ctx.Done():
		// ctx.Err() все еще вернет context.Canceled, если отмена не по дедлайну
		fmt.Printf("Операция (operationWithSpecificCause) отменена. ctx.Err(): %v\n", ctx.Err())
		// А вот context.Cause(ctx) даст нам специфическую причину
		fmt.Printf("Причина отмены (context.Cause): %v\n", context.Cause(ctx))
	}
}

func main() {
	parentCtx := context.Background()

	// Создаем контекст с возможностью указания причины отмены
	ctx, cancelWithCause := context.WithCancelCause(parentCtx)
	// defer cancelWithCause(nil) // Если бы мы хотели "просто" отменить без особой причины к концу функции

	go operationWithSpecificCause(ctx)

	time.Sleep(1 * time.Second) // Дадим операции немного поработать

	// Отменяем операцию с конкретной причиной
	customError := errors.New("произошла критическая ошибка в системе")
	fmt.Printf("Главная горутина: отменяем операцию с причиной: %v\n", customError)
	cancelWithCause(customError)

	time.Sleep(100 * time.Millisecond) // Дать время горутине обработать отмену
	fmt.Println("Главная горутина: завершение.")
}
```

#### AfterFunc() без применения функции остановки
``` go
package main

import (
	"context"
	"fmt"
	"time"
)

func main() {
	parentCtx := context.Background()
	ctx, cancel := context.WithTimeout(parentCtx, 1*time.Second) // Таймаут через 1 секунду
	defer cancel()

	cleanupFunc := func() {
		fmt.Println("AfterFunc: Контекст завершен! Выполняю очистку...")
		// Здесь может быть логика по освобождению ресурсов, закрытию соединений и т.д.
	}

	fmt.Println("Регистрирую cleanupFunc с AfterFunc.")
	stopCleanup := context.AfterFunc(ctx, cleanupFunc)
	_ = stopCleanup // В этом примере мы не будем останавливать

	// Ждем, пока контекст не завершится (из-за таймаута)
	<-ctx.Done()
	fmt.Printf("Основная горутина: контекст завершен с ошибкой: %v\n", ctx.Err())

	// Дадим немного времени AfterFunc выполниться и вывести сообщение
	// В реальном коде это может быть не нужно, если f не делает длительных I/O операций
	time.Sleep(100 * time.Millisecond)
	fmt.Println("Основная горутина: завершение.")
}
```

#### AfterFunc() с применения функции остановки
```go
package main

import (
	"context"
	"fmt"
	"time"
)

func main() {
	parentCtx := context.Background()
	// Контекст, который сам по себе не отменится быстро
	ctx, cancelManually := context.WithCancel(parentCtx)
	defer cancelManually() // Отменится в конце main, если не раньше

	expensiveNotification := func() {
		fmt.Println("AfterFunc: Отправляю дорогое уведомление...")
		// Представим, что это действительно дорогая операция
	}

	fmt.Println("Регистрирую expensiveNotification с AfterFunc.")
	stopNotification := context.AfterFunc(ctx, expensiveNotification)

	// Имитируем ситуацию, когда уведомление больше не нужно
	time.Sleep(500 * time.Millisecond)
	fmt.Println("Уведомление больше не требуется. Вызываю stopNotification().")

	if stopNotification() {
		fmt.Println("Функция expensiveNotification была успешно остановлена до вызова.")
	} else {
		fmt.Println("Не удалось остановить expensiveNotification (возможно, уже выполнилась или запланирована).")
	}

	// Отменяем контекст (хотя AfterFunc уже не должна сработать)
	// cancelManually() // Уже есть в defer

	// Убедимся, что AfterFunc не вызвалась
	fmt.Println("Ждем немного, чтобы убедиться, что AfterFunc не сработала...")
	time.Sleep(1 * time.Second)
	fmt.Println("Основная горутина: завершение.")
}
```



#### Пример: HTTP сервер
```go
package main

import (
	"context"
	"fmt"
	"log"
	"net/http"
	"time"
)

func slowAPICall(ctx context.Context) (string, error) {
	// Имитируем вызов к внешнему API, который может занять время
	// или должен быть отменен, если родительский контекст завершился.
	select {
	case <-time.After(5 * time.Second): // Эта операция занимает 5 секунд
		return "Данные от медленного API", nil
	case <-ctx.Done(): // Если контекст запроса отменен (например, клиент ушел или таймаут сервера)
		log.Printf("slowAPICall: контекст отменен: %v", ctx.Err())
		return "", ctx.Err()
	}
}

func handler(w http.ResponseWriter, r *http.Request) {
	ctx := r.Context() // Получаем контекст запроса

	log.Printf("Обработка запроса: %s %s", r.Method, r.URL.Path)

	// Можно создать дочерний контекст с таймаутом для конкретной операции
	// Например, если мы хотим, чтобы наш slowAPICall не длился дольше 3 секунд,
	// даже если общий таймаут сервера больше.
	opCtx, opCancel := context.WithTimeout(ctx, 3*time.Second)
	defer opCancel()

	data, err := slowAPICall(opCtx) // Передаем opCtx
	if err != nil {
		log.Printf("Ошибка при вызове slowAPICall: %v", err)
		if err == context.DeadlineExceeded {
			http.Error(w, "Операция заняла слишком много времени (внутренний таймаут)", http.StatusGatewayTimeout)
		} else if err == context.Canceled { // Может быть из-за r.Context()
			http.Error(w, "Запрос был отменен клиентом", 499) // 499 Client Closed Request
		} else {
			http.Error(w, "Внутренняя ошибка сервера", http.StatusInternalServerError)
		}
		return
	}

	fmt.Fprintln(w, "Успешно получены данные:", data)
	log.Printf("Запрос успешно обработан для: %s %s", r.Method, r.URL.Path)
}

func main() {
	http.HandleFunc("/data", handler)

	server := &http.Server{
		Addr: ":8080",
		// ReadTimeout:  10 * time.Second, // Общий таймаут чтения запроса
		// WriteTimeout: 10 * time.Second, // Общий таймаут записи ответа
		// IdleTimeout:  15 * time.Second, // Таймаут простоя для keep-alive
	}

	log.Println("Сервер запущен на http://localhost:8080/data")
	log.Println("Попробуйте открыть в браузере. slowAPICall занимает 5с, таймаут на него 3с.")
	log.Println("Также попробуйте обновить страницу в браузере до завершения 3с (это отменит предыдущий r.Context())")

	if err := server.ListenAndServe(); err != nil && err != http.ErrServerClosed {
		log.Fatalf("Ошибка запуска сервера: %v", err)
	}
}
```

#### Распространение контекста (Propagation)
Ключевой аспект работы с `context` — это его передача (проброс) через вызовы функций.

- Функции, которые могут быть длительными или требуют отмены/таймаута, должны принимать `context.Context` **в качестве первого параметра**.
- Имя параметра обычно `ctx`.
```go
func MyFunction(ctx context.Context, arg1 string, arg2 int) error {
    // ... какая-то подготовка ...

    // Если MyFunction вызывает другую функцию, которая также поддерживает контекст:
    err := AnotherFunction(ctx, "данные") // Передаем тот же или дочерний контекст
    if err != nil {
        return err
    }

    // ... основная работа MyFunction, периодически проверяя ctx.Done() ...
    select {
    case <-ctx.Done():
        return ctx.Err() // Контекст отменен
    default:
        // Продолжаем работу
    }

    // ...
    return nil
}
```


