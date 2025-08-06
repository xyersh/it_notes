### http.ServeMux 
- стандартный роутер из библиотеки. 
- На данный момент он весьма не плох: включает поддержку параметров в URL и методов HTTP и тд.
- Отвечает за распределения функций-обработчиков (`type http.HandleFunc`) соответствующим маршрутам.

#### Основные возможности нового роутера
1. **Параметры в URL**:  
    Теперь можно использовать параметры в URL, например `/users/{id}`.
    
2. **Методы HTTP**:  
    Можно явно указывать методы HTTP (`GET`, `POST`, `PUT`, `DELETE` и т.д.) для обработки запросов.
    
3. **Регулярные выражения**:  
    Поддерживаются ограничения на параметры с использованием регулярных выражений.
    
4. **Группировка маршрутов**:  
    Можно группировать маршруты с общими префиксами.


#### Пример использования роутера с применением новых фич:
```go
package main

import (
    "fmt"
    "net/http"
)

func main() {
    mux := http.NewServeMux()

    // Корневой маршрут
    mux.HandleFunc("GET /", func(w http.ResponseWriter, r *http.Request) {
        fmt.Fprintln(w, "Welcome to the Home Page!")
    })

    // Маршрут с параметром
    mux.HandleFunc("GET /users/{id}", func(w http.ResponseWriter, r *http.Request) {
        id := r.PathValue("id")
        fmt.Fprintf(w, "User ID: %s\n", id)
    })

    // Маршрут с несколькими параметрами и регулярным выражением
    mux.HandleFunc("GET /posts/{category}/{id:[0-9]+}", func(w http.ResponseWriter, r *http.Request) {
        category := r.PathValue("category")
        id := r.PathValue("id")
        fmt.Fprintf(w, "Category: %s, Post ID: %s\n", category, id)
    })

    // Группировка маршрутов
    mux.HandleFunc("GET /api/users", func(w http.ResponseWriter, r *http.Request) {
        fmt.Fprintln(w, "List of users")
    })

    mux.HandleFunc("POST /api/users", func(w http.ResponseWriter, r *http.Request) {
        fmt.Fprintln(w, "Create a new user")
    })

    // Middleware
    handler := loggingMiddleware(mux)

    fmt.Println("Server is running on http://localhost:8080")
    http.ListenAndServe(":8080", handler)
}

func loggingMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        fmt.Printf("Request: %s %s\n", r.Method, r.URL.Path)
        next.ServeHTTP(w, r)
    })
}
```



### http.Request
**http.Request** - структура, представляющая собой http запрос в нотации golang

**Важные поля структуры**:
- `Method (string)` - Метод HTTP-запроса (GET, POST, PUT, DELETE и т.д.)
- `URL (*url.URL)` - URL запроса. Содержит информацию о пути, параметрах запроса и т.д.
- `Header (http.Header)` - Заголовки HTTP-запроса. Тип `http.Header` — это `map[string][]string`
- `Body (io.ReadCloser)` - ело запроса. Реализует интерфейс `io.ReadCloser`, что позволяет читать данные из тела запроса.
	Пример: `body, err := io.ReadAll(r.Body)`
- `ContentLength (int64)` - Длина тела запроса в байтах. Если длина неизвестна, значение будет `-1`
- `Host (string)` - Хост, указанный в запросе (например, `example.com`).
- `RemoteAddr (string)` - Адрес клиента, отправившего запрос (например, `192.168.1.1:12345`).
- `Form (url.Values)` - - Параметры формы, извлеченные из тела запроса и URL. Тип `url.Values` — это `map[string][]string`. Пример: `r.FormValue("username")`.
- `PostForm (url.Values)` - Параметры формы, извлеченные только из тела запроса (для POST-запросов). Пример: 
		`r.PostFormValue("password")`.
- `MultipartForm (*multipart.Form)` - - Данные формы, если запрос содержит multipart/form-data (например, загрузка файлов). Пример:       
		`r.MultipartForm.File["file"]`.
- `Context (context.Context)` - Контекст запроса. Полезен для управления временем жизни запроса и передачи значений между middleware. Пример: 
		`ctx := r.Context()`.
- `Cookies ([]*http.Cookie)` - Куки, отправленные клиентом. Пример: 
		`cookie, err := r.Cookie("session")`.
- `TLS (*tls.ConnectionState)` - - Информация о TLS-соединении (если используется HTTPS).   Пример: `r.TLS != nil`.

**Методы запроса( http.Request )** :
- `ParseForm() error`:
         - Парсит данные формы из тела запроса и URL. Данные становятся доступны в полях `Form` и `PostForm`.
	    - Возвращает ошибку, если парсинг не удался.
	    - Пример: `err := r.ParseForm()`.
        
- `FormValue(key string) string`:
        - Возвращает значение параметра формы по ключу. Ищет как в теле запроса, так и в URL.   
	    - Пример: `username := r.FormValue("username")`.
        
- `PostFormValue(key string) string`:
        - Возвращает значение параметра формы только из тела запроса.
        - Пример: `password := r.PostFormValue("password")`.
        
- `Cookie(name string) (*http.Cookie, error)`:
        - Возвращает куку по имени.
	    - Пример: `cookie, err := r.Cookie("session")`.
        
- `Cookies() []*http.Cookie`:
        - Возвращает все куки, отправленные клиентом.    
	    - Пример: `cookies := r.Cookies()`.
        
- `MultipartReader() (*multipart.Reader, error)`:
	    - Возвращает `multipart.Reader` для чтения multipart/form-data.    
	    - Пример: `reader, err := r.MultipartReader()`.
        
- `ParseMultipartForm(maxMemory int64) error`:
	    - Парсит multipart/form-data и сохраняет данные в `MultipartForm`.
	    - Параметр `maxMemory` указывает максимальный объем памяти (в байтах), который может быть использован для хранения данных в памяти.
	    - Пример: `err := r.ParseMultipartForm(10 << 20)` (10 MB).
        
- `Context() context.Context`:
	    - Возвращает контекст запроса.
	    - Пример: `ctx := r.Context()`.
        
- `WithContext(ctx context.Context) *http.Request`:
	    - Создает новый запрос с обновленным контекстом.   
	    - Пример: `newReq := r.WithContext(newCtx)`.
        
- `PathValue(name string) string` (начиная с Go 1.22):
        - Возвращает значение параметра из URL по имени.    
	    - Пример: `id := r.PathValue("id")`.
        
- `Write(w io.Writer) error`:
        - Записывает HTTP-запрос в `io.Writer`. Полезно для логирования или отладки.    
	    - Пример: `err := r.Write(os.Stdout)`.
        
- `WriteProxy(w io.Writer) error`:
        - Записывает запрос в `io.Writer` в формате, подходящем для прокси-серверов.    
	    - Пример: `err := r.WriteProxy(os.Stdout)`.




### url.URL

**Поля и методы**

| Поле/Метод                           | Описание                                                                  |
| ------------------------------------ | ------------------------------------------------------------------------- |
| `String() string`                    | Возвращает строковое представление URL.                                   |
| `Scheme`                             | Поле, содержащее схему URL (например, `http` или `https`).                |
| `Host`                               | Поле, содержащее хост и порт URL (например, `example.com:8080`).          |
| `Path`                               | Поле, содержащее путь к ресурсу (например, `/users/profile`).             |
| `Query() url.Values`                 | Возвращает декодированные параметры запроса в виде `map[string][]string`. |
| `Parse(rawurl string) (*URL, error)` |                                                                           |
### http.Header

**Поля и методы**

| Поле/Метод               | Описание                                                                                       |
| ------------------------ | ---------------------------------------------------------------------------------------------- |
| `Add(key, value string)` | Добавляет значение к ключу, не перезаписывая существующие.                                     |
| `Set(key, value string)` | Устанавливает значение для ключа, перезаписывая существующие.                                  |
| `Get(key string) string` | Возвращает первое значение, связанное с ключом. Если ключ не найден, возвращает пустую строку. |
| `Del(key string)`        | Удаляет все значения, связанные с ключом.                                                      |
| `Values()`               | Возвращает все значения, связанные с ключом.                                                   |


### http.Cookie

| Поле                | Описание                                                                          |
| ------------------- | --------------------------------------------------------------------------------- |
| `Name string`       | Имя cookie.                                                                       |
| `Value string`      | Значение cookie.                                                                  |
| `Path string`       | Путь, на который распространяется действие cookie. По умолчанию `/`.              |
| `Domain string`     | Домен, на который распространяется действие cookie.                               |
| `Expires time.Time` | Время истечения срока действия cookie.                                            |
| `MaxAge int`        | Максимальное время жизни cookie в секундах.                                       |
| `HttpOnly bool`     | Если `true`, cookie будет доступен только по HTTP(S) и недоступен для JavaScript. |
| `Secure bool`       | Если `true`, cookie будет передаваться только по защищенному протоколу HTTPS.     |
| `SameSite SameSite` | Определяет, когда cookie должен отправляться вместе с кросс-сайтовыми запросами.  |

### Пример использования атрибутов запроса:
```go
package main

import (
    "fmt"
    "net/http"
)

func handler(w http.ResponseWriter, r *http.Request) {
    // Пример использования полей и методов
    fmt.Printf("Method: %s\n", r.Method)
    fmt.Printf("Path: %s\n", r.URL.Path)
    fmt.Printf("Host: %s\n", r.Host)
    fmt.Printf("RemoteAddr: %s\n", r.RemoteAddr)

    // Парсинг формы
    err := r.ParseForm()
    if err != nil {
        http.Error(w, "Failed to parse form", http.StatusBadRequest)
        return
    }

    // Получение значения параметра формы
    username := r.FormValue("username")
    fmt.Printf("Username: %s\n", username)

    // Получение куки
    cookie, err := r.Cookie("session")
    if err == nil {
        fmt.Printf("Session cookie: %s\n", cookie.Value)
    }

    // Ответ клиенту
    fmt.Fprintln(w, "Request processed successfully!")
}

func main() {
    http.HandleFunc("/", handler)
    fmt.Println("Server is running on http://localhost:8080")
    http.ListenAndServe(":8080", nil)
}
```

