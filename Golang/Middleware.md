
### Что это такое?

Middleware — это программный компонент, который перехватывает запросы и/или ответы в веб-приложении, выполняя определённые действия до или после вызова основного обработчика.

Middleware в Go, особенно в контексте веб-разработки с использованием таких фреймворков, как `net/http` или `Gin`, представляет собой функцию, которая располагается между обработчиком запроса (handler) и самим запросом. Она позволяет выполнять определенные действия до или после вызова основного обработчика.

Думай о middleware как о конвейере обработки запроса. Классические задачи решаемые при помощи   middleware:

- **Логирование:** Записывать информацию о входящих запросах.
- **Аутентификация и авторизация:** Проверять подлинность пользователя и его права доступа.
- **Валидация:** Проверять корректность входящих данных.
- **Сжатие данных**
- **Изменение запроса или ответа:** Добавлять/изменять заголовки, сжимать данные и т.д.
- **Обработка ошибок:** Перехватывать и обрабатывать ошибки, возникшие в последующих обработчиках.
- **Ограничение скорости (Rate Limiting):** Предотвращение злоупотребления запросами.



### Принцип работы Middleware

Основная идея middleware заключается в том, чтобы вынести общую логику обработки запросов в отдельные, переиспользуемые компоненты, что делает код более чистым, модульным и легким в поддержке.

Ключевой элемент здесь — это интерфейс `http.Handler` и функция `http.HandlerFunc`.

- `http.Handler` — это интерфейс с одним методом `ServeHTTP(w http.ResponseWriter, r *http.Request)`.
- `http.HandlerFunc` — это адаптер, который позволяет обычной функции `func(w http.ResponseWriter, r *http.Request)` соответствовать интерфейсу `http.Handler`.

#### Основные аксиомы использования Middleware
- **Порядок Имеет Значение (Order Matters)** 
	Порядок, в котором вы применяете middleware, определяет порядок их выполнения и взаимодействия с запросом и ответом:
	- Если **middleware логирования** стоит _перед_ **middleware аутентификации**, он зафиксирует попытку доступа даже для неавторизованных запросов.
	- Если **middleware аутентификации** стоит _перед_ **middleware логирования**, а запрос неавторизован, то логирование этого конкретного запроса может быть пропущено или будет неполным (в зависимости от реализации логирования).
- **Middleware Должен Быть Независим от Бизнес-Логики (Separation of Concerns)** 
	Middleware должен выполнять сквозные задачи, которые не являются частью основной бизнес-логики приложения, и быть максимально от неё отвязанным. 
	Middleware предназначен для решения общих задач, таких как аутентификация, логирование, обработка CORS, компрессия и т.д. В идеале, вы должны иметь возможность легко подключать или отключать middleware, не затрагивая основной код обработчика. Если ваш middleware начинает содержать сложную бизнес-логику, это признак того, что её следует перенести в основной обработчик или отдельный сервис.
- **Middleware Может Изменять Запрос и/или Ответ (Request/Response Transformation)**
	Middleware имеет полный доступ и возможность модифицировать объекты `http.Request` и `http.ResponseWriter`. 
	Это одно из мощных свойств middleware. Вы можете добавлять заголовки, изменять тело запроса (например, декодировать JSON), добавлять данные в контекст запроса (`r.Context()`), перехватывать ошибки, модифицировать статус ответа и т.д. **Пример:** **Middleware аутентификации** может добавить информацию о пользователе (полученную из токена) в контекст запроса, чтобы конечный обработчик мог легко получить доступ к ID пользователя.
- **Middleware Может Вызывать Следующий Обработчик (Or Terminate)**
	Middleware либо вызывает следующий `http.Handler` в цепочке, либо завершает обработку запроса, отправляя ответ клиенту.
	Это фундамент работы цепочки middleware. Каждый middleware решает, передать ли управление дальше или остановить обработку. Если middleware обнаруживает ошибку (например, неверный токен), он отправляет соответствующий HTTP-ответ (например, `401 Unauthorized`) и не вызывает `next.ServeHTTP()`. Это предотвращает выполнение лишнего кода и возможные уязвимости.
- **Стоит избегать Чрезмерного Использования `context.WithValue()`**
	Используйте `context.WithValue()` для передачи данных между middleware и обработчиками осмысленно и экономно, избегая создания "неявных" зависимостей.
 - **Middleware Должен Быть Переиспользуемым и Компонуемым (Reusable and Composable)**
	Хорошо спроектированный m
	iddleware должен быть универсальным и легко комбинироваться с другими middleware.
### Примеры использования Middleware
#### Классический пример
```go
package main

import (
	"fmt"
	"net/http"
	"time"
)

// Middleware для логирования времени выполнения запроса
func loggingMiddleware(next http.Handler) http.Handler {
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		startTime := time.Now()
		fmt.Printf("Начало обработки запроса: %s %s\n", r.Method, r.URL.Path)
		next.ServeHTTP(w, r)
		duration := time.Since(startTime)
		fmt.Printf("Завершение обработки запроса: %s %s (длительность: %s)\n", r.Method, r.URL.Path, duration)
	})
}

// Middleware для аутентификации (простой пример)
func authMiddleware(next http.Handler) http.Handler {
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		// Простая проверка наличия заголовка Authorization
		if r.Header.Get("Authorization") == "" {
			http.Error(w, "Требуется авторизация", http.StatusUnauthorized)
			return
		}
		fmt.Println("Пользователь авторизован")
		next.ServeHTTP(w, r)
	})
}

// Обработчик для публичного маршрута
func publicHandler(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w, "Это публичный ресурс")
}

// Обработчик для защищенного маршрута
func protectedHandler(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w, "Это защищенный ресурс")
}

func main() {
	// Создаем новый ServeMux
	mux := http.NewServeMux()

	// Регистрируем обработчик для публичного маршрута без middleware
	mux.HandleFunc("/public", publicHandler)

	// Создаем цепочку middleware для защищенного маршрута
	protectedChain := loggingMiddleware(authMiddleware(http.HandlerFunc(protectedHandler)))

	// Регистрируем обработчик с применением цепочки middleware
	mux.Handle("/protected", protectedChain)

	// Запускаем HTTP-сервер, передавая наш ServeMux
	fmt.Println("Сервер запущен на http://localhost:8080")
	http.ListenAndServe(":8080", mux)
}
```


#### Подход с применением цепочек
Более красивый метод
```go
package main  
  
import (  
    "fmt"  
    "net/http"    
    "slices"    
    "time")  
  
type chain []func(http.Handler) http.Handler  
  
func (c chain) thenFunc(h http.HandlerFunc) http.Handler {  
    return c.then(h)  
}  
  
func (c chain) then(h http.Handler) http.Handler {  
    for _, mw := range slices.Backward(c) {  
       h = mw(h)  
    }  
    return h  
}  
  
// Middleware для логирования времени выполнения запроса  
func loggingMiddleware(next http.Handler) http.Handler {  
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {  
       startTime := time.Now()  
       fmt.Printf("Начало обработки запроса: %s %s\n", r.Method, r.URL.Path)  
       next.ServeHTTP(w, r)  
       duration := time.Since(startTime)  
       fmt.Printf("Завершение обработки запроса: %s %s (длительность: %s)\n", r.Method, r.URL.Path, duration)  
    })  
}  
  
// Middleware для аутентификации (простой пример)  
func authMiddleware(next http.Handler) http.Handler {  
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {  
       // Простая проверка наличия заголовка Authorization  
       if r.Header.Get("Authorization") == "" {  
          http.Error(w, "Требуется авторизация", http.StatusUnauthorized)  
          return  
       }  
       fmt.Println("Пользователь авторизован")  
       next.ServeHTTP(w, r)  
    })  
}  
  
// Обработчик для публичного маршрута  
func publicHandler(w http.ResponseWriter, r *http.Request) {  
    fmt.Fprintf(w, "Это публичный ресурс")  
}  
  
// Обработчик для защищенного маршрута  
func protectedHandler(w http.ResponseWriter, r *http.Request) {  
    fmt.Fprintf(w, "Это защищенный ресурс")  
}  
  
func main() {  
    // Создаем новый ServeMux  
    mux := http.NewServeMux()  
  
    // Создаем цепочку middleware для публичного маршрута  
    baseChain := chain{loggingMiddleware}  
    // Создаем цепочку middleware для защищенного маршрута  
    authChain := chain{loggingMiddleware, authMiddleware}  
  
    //// Регистрируем обработчик для публичного маршрута без middleware  
    mux.Handle("/public", baseChain.thenFunc(publicHandler))  
    // Регистрируем обработчик с применением цепочки middleware  
    mux.Handle("/protected", authChain.thenFunc(protectedHandler))  
  
    // Запускаем HTTP-сервер, передавая наш ServeMux  
    fmt.Println("Сервер запущен на http://localhost:8888")  
    http.ListenAndServe(":8888", mux)  
}
```