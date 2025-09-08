### База

Пакет `log/slog`  предназначен для структурированного логирования.
Во всем является заменой старому логгеру из пакета  `log`, и сушествующим популярным сторонним решениям.

#### Основные термины
Данный пекет построен вокруг нескольких ключевых сущностей, которые совместно обеспечивают гибкость и мощь инструмента.

- **Logger (Логгер)**
**`slog.Logger`** — это основной тип, используемый для логирования. Он отвечает за отправку записей логов обработчику (`Handler`). Вы можете создать новый логгер с помощью `slog.New(handler)`. Логгер имеет методы для логирования на разных уровнях: `Info`, `Debug`, `Warn` и `Error`.

- **Level (Уровень логирования)**
**`slog.Level`** определяет важность сообщения. Стандартные уровни:
	- `slog.LevelDebug`: для отладочных сообщений.
	- `slog.LevelInfo`: для информационных сообщений. Это уровень по умолчанию.
	- `slog.LevelWarn`: для предупреждений, которые могут указывать на потенциальные проблемы.
	- `slog.LevelError`: для сообщений об ошибках, которые требуют внимания. Вы также можете создавать свои **кастомные уровни**, если требуется более гранулированный контроль.

- **Handler (Обработчик)**
**`slog.Handler`** — это интерфейс, который обрабатывает запись лога, созданную логгером. Он решает, что делать с сообщением: отформатировать его, записать в файл, отправить в удалённую систему или даже проигнорировать. В Go есть несколько встроенных обработчиков:
	- **`slog.TextHandler`**: выводит логи в виде пар **ключ=значение**. Это удобно для чтения человеком.
	- **`slog.JSONHandler`**: выводит логи в формате **JSON**. Это идеальный выбор для систем мониторинга и анализа логов, таких как ELK Stack (Elasticsearch, Logstash, Kibana).

- **Attributes (Атрибуты)**
**`slog.Attr`** или **`any`** (в `...any` параметрах) — это пары **ключ-значение**, которые добавляются к записи лога. Они делают логи **структурированными**. Например, вместо "User logged in with ID 123" можно использовать `slog.Info("user logged in", "user_id", 123)`. В результате получится запись, которую легко парсить. Атрибуты могут быть добавлены в логгер на постоянной основе с помощью `With` или при вызове конкретного метода логирования (`Info`, `Error` и т.д.).


#### Логирование в несколько получателей

Одна из мощных возможностей `slog` — это возможность направлять логи из одного источника в несколько разных мест. Это достигается за счёт использования композиции обработчиков.

Для этого можно создать **кастомный обработчик**, который будет оборачивать несколько других. Вот как это можно реализовать:
1. Создайте структуру, которая будет хранить ссылки на несколько других `slog.Handler`.
2. Реализуйте для этой структуры интерфейс `slog.Handler`, который включает методы `Enabled`, `Handle` и `WithAttrs`.
3. В методе `Handle` вашей кастомной структуры вы просто вызываете `Handle` для каждого из вложенных обработчиков.
    
Этот подход позволяет, например, отправлять логи ошибок в файл, а информационные логи — в консоль.

### slog API

- `slog.New(h slog.Handler)(*slog.Logger)` - создаёт новый **логгер** . Она принимает один аргумент: `h`, который является обработчиком (`slog.Handler`).

##### slog.Logger
```go
handler := slog.NewTextHandler(os.Stdout, nil)
logger := slog.New(handler)
```
- `func (l *Logger) Debug(msg string, args ...any)`: Логирует сообщение на уровне `Debug`. Используется для отладочной информации.
    
- `func (l *Logger) Info(msg string, args ...any)`: Логирует сообщение на уровне `Info`. Используется для общей информации о работе программы.
    
- `func (l *Logger) Warn(msg string, args ...any)`: Логирует сообщение на уровне `Warn`. Используется для потенциальных проблем, которые не являются ошибками.
    
- `func (l *Logger) Error(msg string, args ...any)`: Логирует сообщение на уровне `Error`. Используется для сообщений об ошибках, которые требуют внимания.
```go
logger.Info("User logged in", "user_id", 123, "email", "john.doe@example.com")
logger.Error("Database connection failed", "error", err, "db_name", "users")
```

##### Добавление атрибутов к логгеру
- `func (l *Logger) With(args ...any) *Logger`: Этот метод возвращает **новый логгер**, который включает в себя дополнительные атрибуты. Это позволяет создавать контекстно-зависимые логгеры.
```go
requestLogger := logger.With("request_id", "req-12345")
requestLogger.Info("Processing request") // Этот лог будет содержать request_id
```


##### Группировка атрибутов
- `func (l *Logger) WithGroup(name string) *Logger`: Создаёт **новый логгер** с группой атрибутов. Все последующие атрибуты, добавленные к этому логгеру, будут вложены в указанную группу.
```go
loggerWithUser := logger.WithGroup("user")
loggerWithUser.Info("user data", "id", 101, "name", "Alice")
// Результат в JSON: {"user": {"id": 101, "name": "Alice"}}
```


##### slog.Handler
Это интерфейс, который обрабатывает записи логов. Он определяет, как логи обрабатываются и куда отправляются. Понимание этого интерфейса — ключ к созданию кастомных обработчиков, которые могут отправлять логи в любую систему, будь то удалённый сервис, база данных или специализированный формат.
Интерфейс `slog.Handler` объявляет четыре основных метода:

1. `Handle(ctx context.Context, r slog.Record) error`
    - **Назначение**: Это самый важный метод. Он отвечает за фактическую обработку записи лога. Когда вы вызываете `logger.Info()`, `logger.Error()` и т. д., логгер создает объект `slog.Record` и передает его этому методу.
    - **Использование**: Внутри этого метода вы должны взять данные из `slog.Record` (сообщение, уровень, время, атрибуты) и записать их в нужное вам место (например, в файл, в HTTP-запрос или в консоль).
        
2. `Enabled(ctx context.Context, level slog.Level) bool`
    - **Назначение**: Этот метод определяет, нужно ли обрабатывать лог на заданном уровне. Он вызывается перед `Handle` для **быстрой проверки**. Если `Enabled` возвращает `false`, `Handle` не будет вызван, что позволяет сэкономить ресурсы, не подготавливая запись для логирования, если она будет отброшена.
    - **Использование**: Вы можете использовать этот метод для реализации фильтрации по уровням. Например, ваш обработчик может быть настроен так, чтобы обрабатывать только логи уровня `Error` и выше.
        
3. `WithAttrs(attrs []slog.Attr) slog.Handler`
    - **Назначение**: Этот метод возвращает **новый обработчик** с добавленными атрибутами. Он используется, когда вы вызываете `logger.With()`. Атрибуты, переданные в `With()`, должны быть сохранены в новом обработчике и добавлены ко всем последующим логам.
    - **Использование**: При создании нового обработчика вы должны сохранить переданные атрибуты и убедиться, что они будут включены во все записи, обрабатываемые новым экземпляром.
        
4. `WithGroup(name string) slog.Handler`
    - **Назначение**: Этот метод также возвращает **новый обработчик**, но с группой. Он используется при вызове `logger.WithGroup()`. Все последующие атрибуты, добавленные к новому обработчику, будут вложены в указанную группу.
    - **Использование**: Ваша реализация должна отслеживать имена групп и правильно вкладывать атрибуты в итоговую запись лога.


##### Стандартные обработчики:

- `slog.NewTextHandler(w io.Writer, opts *HandlerOptions)`: Создаёт обработчик, который выводит логи в формате **ключ=значение**.
- `slog.NewJSONHandler(w io.Writer, opts *HandlerOptions)`: Создаёт обработчик, который выводит логи в формате **JSON**. Это идеальный выбор для машинного парсинга.


##### slog.HandlerOptions
Эта структура используется для настройки `slog.Handler`.

**Основные поля:**
- `Level slog.Level`: Определяет минимальный уровень логирования. Логи с уровнем ниже этого значения будут проигнорированы.
- `AddSource bool`: Если `true`, добавляет в запись лога информацию о файле и номере строки, где был вызван логгер.
- `ReplaceAttr func(groups []string, a slog.Attr) slog.Attr`: Функция-замыкание для изменения или удаления атрибутов. Очень мощный инструмент для кастомизации.
```go
opts := &slog.HandlerOptions{
    Level:     slog.LevelDebug,
    AddSource: true,
}
handler := slog.NewTextHandler(os.Stdout, opts)
```

##### #### Глобальный логгер
Глобальный логгер в контексте пакета `log/slog` — это **логгер, который используется по умолчанию во всей программе**. Он доступен через функции верхнего уровня в пакете, такие как `slog.Info`, `slog.Error` и так далее.
- `slog.Default() *slog.Logger`: Возвращает глобальный логгер, который по умолчанию использует `slog.NewTextHandler(os.Stderr, nil)` с уровнем `Info`.
- `slog.SetDefault(l *Logger)`: Устанавливает переданный логгер как глобальный. Это удобно для централизованной конфигурации.
```go
// Установка глобального логгера
slog.SetDefault(slog.New(slog.NewJSONHandler(os.Stdout, nil)))

// Использование глобального логгера
slog.Info("Application started")
```



### Примеры
#### Приложение
```go
package main

import (
	"context"
	"log/slog"
	"os"
)

// MultiHandler - кастомный обработчик, который отправляет логи в несколько получателей.
type MultiHandler struct {
	handlers []slog.Handler
}

// NewMultiHandler создает новый MultiHandler
func NewMultiHandler(handlers ...slog.Handler) *MultiHandler {
	return &MultiHandler{
		handlers: handlers,
	}
}

// Enabled реализует метод интерфейса slog.Handler.
// В этом примере, если хотя бы один обработчик включен, MultiHandler тоже будет включен.
func (h *MultiHandler) Enabled(ctx context.Context, level slog.Level) bool {
	for _, handler := range h.handlers {
		if handler.Enabled(ctx, level) {
			return true
		}
	}
	return false
}

// Handle реализует метод интерфейса slog.Handler.
// Отправляет запись лога каждому из вложенных обработчиков.
func (h *MultiHandler) Handle(ctx context.Context, record slog.Record) error {
	for _, handler := range h.handlers {
		if handler.Enabled(ctx, record.Level) {
			if err := handler.Handle(ctx, record); err != nil {
				return err
			}
		}
	}
	return nil
}

// WithAttrs реализует метод интерфейса slog.Handler.
func (h *MultiHandler) WithAttrs(attrs []slog.Attr) slog.Handler {
	newHandlers := make([]slog.Handler, len(h.handlers))
	for i, handler := range h.handlers {
		newHandlers[i] = handler.WithAttrs(attrs)
	}
	return NewMultiHandler(newHandlers...)
}

// WithGroup реализует метод интерфейса slog.Handler.
func (h *MultiHandler) WithGroup(name string) slog.Handler {
	newHandlers := make([]slog.Handler, len(h.handlers))
	for i, handler := range h.handlers {
		newHandlers[i] = handler.WithGroup(name)
	}
	return NewMultiHandler(newHandlers...)
}

func main() {
	// Создаем JSON-обработчик для вывода в файл
	jsonFile, err := os.Create("logs.json")
	if err != nil {
		slog.Error("failed to create log file", "error", err)
		os.Exit(1)
	}
	defer jsonFile.Close()

	jsonHandler := slog.NewJSONHandler(jsonFile, &slog.HandlerOptions{
		Level: slog.LevelInfo,
	})

	// Создаем текстовый обработчик для вывода в консоль (stdout)
	textHandler := slog.NewTextHandler(os.Stdout, &slog.HandlerOptions{
		Level: slog.LevelDebug,
	})

	// Объединяем обработчики в MultiHandler
	multiHandler := NewMultiHandler(jsonHandler, textHandler)

	// Создаем логгер, который использует MultiHandler
	logger := slog.New(multiHandler)

	// Устанавливаем наш логгер как глобальный, чтобы его могли использовать все функции
	slog.SetDefault(logger)

	// Пример логирования
	slog.Debug("This is a debug message") // Будет выведено только в консоль (textHandler)
	slog.Info("User authenticated successfully", "user_id", 42, "role", "admin") // Будет выведено и в файл, и в консоль
	slog.Warn("Cache miss occurred", "cache_key", "user:123") // Будет выведено и в файл, и в консоль
	slog.Error("Failed to connect to database", "db_name", "users_db", "error", "connection refused") // Будет выведено и в файл, и в консоль

	// Добавление атрибутов к логгеру
	appLogger := logger.With("app_name", "auth-service")
	appLogger.Info("Application is starting up")

	// Логирование с группами
	requestLogger := logger.With("request_id", "req-12345")
	requestLogger.Info("Processing incoming request",
		slog.Group("user_info",
			slog.String("user_agent", "Mozilla/5.0"),
			slog.String("ip_address", "192.168.1.1"),
		))
}
```


#### Создание кастомного обработчика

Чтобы создать свой обработчик, вы должны определить структуру, которая будет хранить его состояние, а затем реализовать все четыре метода интерфейса `slog.Handler`.

 Пошаговый пример создания простого кастомного обработчика, который отправляет логи в **HTTP-сервер**.

##### 1. **Определите структуру обработчика**. 
Она будет хранить HTTP-клиент, URL-адрес и, возможно, уровень логирования.
```go
package main

import (
	"context"
	"io"
	"log/slog"
	"net/http"
	"bytes"
	"encoding/json"
	"os"
)

// HTTPHandler отправляет логи в виде JSON-документов по HTTP
type HTTPHandler struct {
	client  *http.Client
	url     string
	level   slog.Level
	handler slog.Handler // Оборачиваем стандартный JSONHandler для удобства
}

// NewHTTPHandler создает новый HTTPHandler
func NewHTTPHandler(url string, level slog.Level) *HTTPHandler {
	return &HTTPHandler{
		client:  &http.Client{},
		url:     url,
		level:   level,
		// Оборачиваем slog.NewJSONHandler для простоты
		// Он будет форматировать запись в JSON
		handler: slog.NewJSONHandler(io.Discard, &slog.HandlerOptions{Level: level}),
	}
}
```

##### 2.  **Реализуйте метод `Enabled`**.
```go
   func (h *HTTPHandler) Enabled(_ context.Context, level slog.Level) bool {
	return level >= h.level
}
```


##### 3.**Реализуйте метод `Handle`**. 
Это самая сложная часть. Сначала мы используем вложенный `JSONHandler` для форматирования записи, а затем отправляем её по HTTP.
```go
func (h *HTTPHandler) Handle(ctx context.Context, r slog.Record) error {
	// Создаем буфер для записи JSON
	var buf bytes.Buffer
	bufHandler := slog.NewJSONHandler(&buf, &slog.HandlerOptions{
		Level: h.level,
	})

	// Записываем форматированную запись в буфер
	if err := bufHandler.Handle(ctx, r); err != nil {
		return err
	}

	// Создаем HTTP-запрос
	req, err := http.NewRequestWithContext(ctx, "POST", h.url, &buf)
	if err != nil {
		return err
	}
	req.Header.Set("Content-Type", "application/json")

	// Отправляем запрос
	resp, err := h.client.Do(req)
	if err != nil {
		return err
	}
	defer resp.Body.Close()

	if resp.StatusCode != http.StatusOK {
		return &http.Response{Status: resp.Status}
	}

	return nil
}
```

##### **4.Реализуйте методы `WithAttrs` и `WithGroup`**. 
Это важно, чтобы ваш обработчик правильно поддерживал контекстные логи.
```go
func (h *HTTPHandler) WithAttrs(attrs []slog.Attr) slog.Handler {
	// Создаем новый обработчик с добавленными атрибутами
	// Это важно для сохранения состояния
	newHandler := *h
	newHandler.handler = h.handler.WithAttrs(attrs)
	return &newHandler
}

func (h *HTTPHandler) WithGroup(name string) slog.Handler {
	// Создаем новый обработчик с группой
	newHandler := *h
	newHandler.handler = h.handler.WithGroup(name)
	return &newHandler
}
```

##### Полный пример с использованием
```go
// Для примера создадим простой HTTP-сервер, который будет принимать логи
func startMockServer() {
	http.HandleFunc("/logs", func(w http.ResponseWriter, r *http.Request) {
		body, _ := io.ReadAll(r.Body)
		slog.Info("Received log via HTTP", "body", string(body))
		w.WriteHeader(http.StatusOK)
	})
	go http.ListenAndServe(":8080", nil)
}

func main() {
	// Запускаем наш мок-сервер
	startMockServer()

	// Устанавливаем наш кастомный обработчик в качестве глобального
	httpHandler := NewHTTPHandler("http://localhost:8080/logs", slog.LevelDebug)
	slog.SetDefault(slog.New(httpHandler))

	slog.Info("Service started", "version", "1.0.0")
	slog.Error("Failed to load config", "file", "config.yml")
}
```

При запуске этого кода `slog.Info` и `slog.Error` отправят логи в виде JSON-документов на `http://localhost:8080/logs`, а наш мок-сервер выведет их в консоль. Это демонстрирует, как можно использовать интерфейс `slog.Handler` для создания собственного решения для логирования, интегрированного с любой системой.