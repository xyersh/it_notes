
	

### gin.Driver

#### Инициализация
- `gin.Default()` -Создает экземпляр роутера Gin с предустановленными middleware: **`Logger`** (для логирования HTTP-запросов) и **`Recovery`** (для восстановления после падений/паник). Рекомендуется для большинства приложений.
```go
router := gin.Default() 
```

- `gin.New()` - Создает "пустой" экземпляр роутера Gin без каких-либо middleware по умолчанию. Вы можете добавлять их вручную с помощью `Use()`. Подходит, если нужен полный контроль над middleware.
```go
router := gin.New()
```

#### HTTP-методы
- `router.GET(relativePath string, handlers ...gin.HandlerFunc)` - Регистрирует маршрут для обработки **GET**-запросов. `relativePath` может содержать параметры URL (например, `"/users/:id"`). `handlers` - это одна или несколько функций-обработчиков.
```go
router.GET("/users", handler)
```
- `router.POST(relativePath string, handlers ...gin.HandlerFunc)` - Регистрирует маршрут для обработки **POST**-запросов. Используется для создания новых ресурсов.
```go 
router.POST("/users", handler)
```
- `router.PUT(relativePath string, handlers ...gin.HandlerFunc)` - Регистрирует маршрут для обработки **PUT**-запросов. Обычно используется для полного обновления существующих ресурсов.
```go
router.PUT("/users/:id", handler)
```
- `router.DELETE(relativePath string, handlers ...gin.HandlerFunc)` - Регистрирует маршрут для обработки **DELETE**-запросов. Используется для удаления ресурсов.
```go
router.DELETE("/users/:id", handler)
```
- `router.PATCH(relativePath string, handlers ...gin.HandlerFunc)` - Регистрирует маршрут для обработки **PATCH**-запросов. Используется для частичного обновления существующих ресурсов.
```go
router.PATCH("/users/:id", handler)
```
- `router.Any(relativePath string, handlers ...gin.HandlerFunc)` - Регистрирует маршрут для обработки **любого** HTTP-метода (GET, POST, PUT, DELETE, PATCH, OPTIONS, HEAD). Используйте осторожно, так как может усложнить отладку и безопасность.
```go
router.Any("/catchall", handler)
```

#### Middleware

- `router.Use(middlewares ...gin.HandlerFunc)` - Регистрирует одну или несколько функций **middleware**, которые будут выполняться для **каждого входящего запроса** в роутере. `middleware` выполняется в порядке их регистрации.
```go
router.Use(gin.Logger(), myCustomMiddleware())
```
#### Группировка маршрутов

- `router.Group(relativePath string, handlers ...gin.HandlerFunc) *RouterGroup` - Создает новую **группу маршрутов**. Все маршруты, определенные внутри этой группы, будут иметь префикс `relativePath` и могут использовать собственные middleware, которые будут применены только к маршрутам этой группы. `handlers` здесь - это middleware для группы.
```go
adminGroup := router.Group("/admin", authMiddleware()) 
...
adminGroup.GET("/dashboard", handler)
```

#### Запуск сервера

- `router.Run(addr ...string) error` - Запускает HTTP-сервер Gin на указанном адресе/порту. Если адрес не указан, по умолчанию используется `:8080`. Этот метод блокирует выполнение программы.
```go
router.Run()` или `router.Run(":8080")
```

- `router.RunTLS(addr, certFile, keyFile string) error` - Запускает HTTPS-сервер с использованием TLS. Требует путь к файлу сертификата и файлу ключа.
```go
router.RunTLS(":8080", "cert.pem", "key.pem")
```


#### Статические файлы

- `router.Static(relativePath, root string)` - Предоставляет статические файлы из указанной корневой директории. `relativePath` - это URL-путь, по которому файлы будут доступны, `root` - путь к папке с файлами.
```go
router.Static("/assets", "./static") // позволит получить доступ к `static/image.png` по `/assets/image.png`
```

- `router.StaticFS(relativePath string, fs http.FileSystem)` - Предоставляет статические файлы из файловой системы, реализующей интерфейс `http.FileSystem`. Полезно для встраивания файлов.
```go
router.StaticFS("/public", http.Dir("./public"))
```

- `router.StaticFile(relativePath, filepath string)` - Предоставляет один статический файл.
```go
router.StaticFile("/favicon.ico", "./favicon.ico")
```


#### HTML-рендеринг

- `router.LoadHTMLGlob(pattern string)` - Загружает HTML-шаблоны из файлов, соответствующих указанному шаблону (glob-паттерну).
```go
router.LoadHTMLGlob("templates/*")
```

- `router.LoadHTMLFiles(files ...string)` - Загружает HTML-шаблоны из списка указанных файлов.
```go
router.LoadHTMLFiles("templates/index.html", "templates/about.html")
```

#### Другие
- `router.NoRoute(handlers ...gin.HandlerFunc)` - Регистрирует обработчик, который будет вызван, если ни один из других маршрутов не совпал с входящим запросом (аналог 404 Not Found).
```go
router.NoRoute(func(c *gin.Context) { c.JSON(404, gin.H{"message": "Не найдено"}) })
```

- `router.SetTrustedProxies(trustedProxies []string)` - Устанавливает список доверенных прокси-серверов. Это важно для корректного определения IP-адреса клиента, когда приложение находится за прокси-сервером (например, Nginx, Cloudflare).
```go
router.SetTrustedProxies([]string{"192.168.1.1", "10.0.0.0/8"})
```

- `router.SetTrustedProxies(nil)` - Сбрасывает список доверенных прокси, отключая доверие к прокси.
```go
router.SetTrustedProxies(nil)
```




### gin.Context - методы

Структура `gin.Context` является центральным элементом в Gin, предоставляя доступ ко всей информации о входящем HTTP-запросе и позволяя формировать ответ. Это ваш основной инструмент для взаимодействия с запросом и клиентом. 

Пусть `c` - переменна  с типом  `gin.Contex`


- `c.GetHeader(key string) string` - получить значение заголовка  запроса ключу `key`

- `c.Request.Header() map[string][]string` - получить из запроса ВСЕ заголовки в виде мапы

- `c.Header(key string, value string)` - Этот метод устанавливает или перезаписывает заголовок ответа с указанным ключом и значением.
	
- `c.Param(key string) string`  Извлекает значение **параметра пути (URL parameter)** из запроса. Параметры пути определяются в маршруте с двоеточием, например, `/users/:id`.  Используется, когда часть URL является динамической и представляет собой идентификатор или имя ресурса.
	```go
	router.GET("/users/:id", func(c *gin.Context) {
	    id := c.Param("id") // Если запрос /users/123, id будет "123"
	    c.String(http.StatusOK, "User ID: %s", id)
	})
```

- `c.Query(key string) string` - Извлекает значение **параметра запроса (query parameter)** из URL (часть после `?`). Используются, когда нужно получить опциональные параметры из URL, часто для фильтрации, сортировки или пагинации.

- `c.DefaultQuery(key, defaultValue string)` - то же самое, что и `Query`, но возвращает `defaultValue`, если параметр не найден в запросе. 

```go
router.GET("/search", func(c *gin.Context) {
keyword := c.Query("q") // Запрос /search?q=golang, keyword будет "golang"
limit := c.DefaultQuery("limit", "10") // Запрос /search, limit будет "10"
c.String(http.StatusOK, "Searching for '%s' with limit %s", keyword, limit)
})
```


- `c.BindJSON(obj interface{}) error`: Пытается привязать **JSON-тело запроса** к указанной Go-структуре (`obj`). В случае ошибки привязки (например, неверный формат JSON или проблемы с валидацией) возвращает ошибку и автоматически записывает её в `c.Errors`. Применима, когда  клиент отправляет данные в формате JSON (например, через POST, PUT, PATCH запросы) и вам нужно преобразовать их в Go-структуру для дальнейшей обработки.

- `c.ShouldBindJSON(obj interface{}) error`: Аналогично `BindJSON`, но в случае ошибки **не записывает** её в `c.Errors`. Это полезно, когда вы хотите полностью контролировать обработку ошибок валидации или парсинга. Рекомендуется использовать `ShouldBindJSON` для более гибкой обработки ошибок. 
```go
type User struct {
    Name  string `json:"name" binding:"required"`
    Email string `json:"email" binding:"required,email"`
}

router.POST("/users", func(c *gin.Context) {
    var user User
    if err := c.ShouldBindJSON(&user); err != nil {
        c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
        return
    }
    c.JSON(http.StatusOK, user)
})
```

- `c.String(code int, format string, values ...interface{})` - Отправляет **простой текстовый ответ** клиенту с указанным HTTP-статусом. Поддерживает форматирование строки в стиле `fmt.Sprintf`.Применима для простых текстовых ответов, сообщений об ошибках или отладочной информации.
```go
router.GET("/hello", func(c *gin.Context) {
    c.String(http.StatusOK, "Hello, %s!", "Gin User")
})
```

- `c.JSON(code int, obj interface{})` - Отправляет **JSON-ответ** клиенту. Gin автоматически преобразует предоставленный объект (`obj`) в JSON и устанавливает правильный заголовок `Content-Type: application/json`.  Это основной метод для построения RESTful API, где данные обмениваются в формате JSON.
 Параметр `obj` в этом случае может быть:
	 - `gin.H` - синтаксический  сахар  для `map[string]interface{}`
	 - структуры 
	 - срезы структур. В этом случае будет отправлен массив json-данных в качестве ответа.
	 - `map[string]interface{}`
	 - `map[string]string`
	 - `[]string`
	 - `int`, `float64`, `bool`, `string`
```go
router.GET("/api/data", func(c *gin.Context) {
    c.JSON(http.StatusOK, gin.H{
        "status": "success",
        "message": "Data retrieved successfully",
    })
})
```

- `c.IndentedJSON(code int, obj interface{})` - аналогично `c.JSON` но в итоге json-ответ будет содержать отступы в содержимом.

- `c.HTML(code int, name string, obj interface{})` -  Рендерит и отправляет **HTML-шаблон** клиенту с указанным HTTP-статусом. `name` - имя шаблона, а `obj` - данные, которые будут переданы в шаблон. Применяется при создании традиционных веб-приложений, где сервер рендерит HTML-страницы. Перед использованием необходимо загрузить шаблоны с помощью `router.LoadHTMLGlob()` или `router.LoadHTMLFiles()`.
```go
// В main: router.LoadHTMLGlob("templates/*")
router.GET("/web", func(c *gin.Context) {
    c.HTML(http.StatusOK, "index.html", gin.H{
        "title": "My Web Page",
        "data": "Hello from Gin HTML!",
    })
})
```

- `c.AbortWithStatusJSON(code int, obj interface{})` -  Прерывает выполнение цепочки обработчиков/мидлваров для текущего запроса и отправляет **JSON-ответ** с указанным HTTP-статусом.  Часто используется в мидлварах для немедленного прекращения обработки запроса в случае ошибки (например, неавторизованный доступ, некорректные данные). После вызова `AbortWithStatusJSON` важно также использовать `return`, чтобы избежать дальнейшего выполнения кода в текущем обработчике.

```go
func AuthMiddleware() gin.HandlerFunc {
    return func(c *gin.Context) {
        token := c.GetHeader("Authorization")
        if token == "" {
            c.AbortWithStatusJSON(http.StatusUnauthorized, gin.H{"error": "Authorization token required"})
            return // Важно!
        }
        c.Next()
    }
}
```

- `c.Next()` -  Передает управление **следующему обработчику** в цепочке мидлваров или непосредственно обработчику маршрута. Применяется исключительно в функциях **middleware**. Код, расположенный до `c.Next()`, выполняется до обработки запроса, а код после `c.Next()` выполняется после того, как все последующие мидлвары и основной обработчик завершат свою работу.


- `c.ShouldBindUri(obj interface{}) error`, `c.ShouldBindHeader(obj interface{}) error`, `c.ShouldBindQuery(obj interface{}) error`, `c.ShouldBindForm(obj interface{}) error` - Расширенные методы для привязки данных из различных источников к Go-структурам.
	- `ShouldBindUri`: Привязывает параметры пути (из URL) к полям структуры.
    - `ShouldBindHeader`: Привязывает заголовки запроса к полям структуры.
    - `ShouldBindQuery`: Привязывает параметры запроса (query parameters) к полям структуры.
    - `ShouldBindForm`: Привязывает данные из форм (`application/x-www-form-urlencoded` или `multipart/form-data`) к полям структуры.
    Применяются, когда нужно комплексно валидировать и структурировать данные, полученные из различных частей запроса, особенно полезно для сложных фильтров или DTO (Data Transfer Objects).
```go
type SearchQuery struct {
    Keyword string `form:"q" binding:"required"`
    Limit   int    `form:"limit" binding:"gte=1,lte=100"`
}

router.GET("/advanced-search", func(c *gin.Context) {
    var query SearchQuery
    if err := c.ShouldBindQuery(&query); err != nil {
        c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
        return
    }
    c.JSON(http.StatusOK, query)
})
```