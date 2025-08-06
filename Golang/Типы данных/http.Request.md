### База

**`http.Request`**: Эта структура инкапсулирует всю информацию о **входящем HTTP-запросе**. Думайте о ней как о контейнере, который содержит детали запроса от клиента, такие как URL, заголовки, метод запроса, тело запроса (если есть) и многое другое.

### API
- **`r.Method`**: Строковое представление HTTP-метода (например, "GET", "POST", "PUT").
- **`r.URL.Path`**: Путь запрошенного URI (например, для `http://localhost:8080/users?id=123` это будет `/users`).
- **`r.Header`**: Карта (map) `http.Header`, содержащая все заголовки запроса. 
- Метод **`r.Header.Get("User-Agent")`** — это безопасный способ получить значение заголовка по имени.
- **`r.URL.Query()`**: Возвращает `url.Values`, которая является картой параметров строки запроса. 
- Метод **`r.URL.Query().Get("name")`** используется для получения значения параметра `name`.
- **`r.Body`**: `io.ReadCloser`, представляющий тело запроса. Это особенно важно для POST, PUT и других методов, которые могут содержать данные в теле. Важно помнить, что `r.Body` можно прочитать только **один раз**. Для больших запросов используйте `http.MaxBytesReader` для защиты от переполнения памяти.
- **`r.Context()`** - получить контекст из запроса
### Важное
####  Тело запроса `r.Body` читается только один раз!   
Это одна из самых частых "ловушек". **`r.Body`** является `io.ReadCloser`, и после того, как вы его прочитали (например, с помощью `ioutil.ReadAll` или `json.NewDecoder`), данные из него **исчезают**. Если вам нужно прочитать тело несколько раз, вам придется сохранить его в буфере (например, `bytes.Buffer`), а затем сбросить его обратно в `r.Body`
```go
package main

import (
	"bytes"
	"fmt"
	"io"
	"net/http"
)

func bodyTrickHandler(w http.ResponseWriter, r *http.Request) {
	if r.Method != http.MethodPost {
		http.Error(w, "Метод не поддерживается", http.StatusMethodNotAllowed)
		return
	}

	// 1. Прочитаем тело запроса
	bodyBytes, err := io.ReadAll(r.Body)
	if err != nil {
		http.Error(w, "Ошибка чтения тела запроса", http.StatusInternalServerError)
		return
	}
	fmt.Fprintf(w, "Первое чтение тела: %s\n", string(bodyBytes))

	// 2. Теперь r.Body пуст. Если мы попробуем прочитать снова, ничего не получится.
	// r.Body = io.NopCloser(bytes.NewBuffer(bodyBytes)) // Вот эта строка позволит прочитать заново

	// 3. Чтобы прочитать тело повторно, нужно восстановить его.
	r.Body = io.NopCloser(bytes.NewBuffer(bodyBytes))

	// 4. Прочитаем тело второй раз
	secondBodyBytes, err := io.ReadAll(r.Body)
	if err != nil {
		http.Error(w, "Ошибка повторного чтения тела запроса", http.StatusInternalServerError)
		return
	}
	fmt.Fprintf(w, "Второе чтение тела: %s\n", string(secondBodyBytes))
}

func main() {
	http.HandleFunc("/body", bodyTrickHandler)
	fmt.Println("Сервер запущен на http://localhost:8080")
	http.ListenAndServe(":8080", nil)
}
```


#### Ограничение размера тела запроса для безопасности
Всегда используйте **`http.MaxBytesReader`** при чтении тела запроса, особенно для `POST` или `PUT` запросов. Это предотвращает атаки типа "отказ в обслуживании" (DoS), когда злоумышленник отправляет гигантское тело запроса, чтобы исчерпать ресурсы вашего сервера.
```go
// До:
// bodyBytes, err := io.ReadAll(r.Body)

// После (рекомендуется):
const maxBodySize = 1 * 1024 * 1024 // 1 MB
r.Body = http.MaxBytesReader(w, r.Body, maxBodySize)
bodyBytes, err := io.ReadAll(r.Body)
if err != nil {
    // В случае превышения лимита err будет "http: request body too large"
    http.Error(w, "Тело запроса слишком большое", http.StatusRequestEntityTooLarge)
    return
}
```


#### Получение значений форм и JSON
- **Для `application/x-www-form-urlencoded` и `multipart/form-data`**: Используйте **`r.FormValue("ключ")`** или **`r.PostFormValue("ключ")`**. `r.FormValue` сначала ищет в параметрах URL, затем в теле формы. `r.PostFormValue` ищет только в теле POST-запроса. Перед их использованием, если ожидаются большие формы, рекомендуется вызвать `r.ParseForm()` или `r.ParseMultipartForm(maxMemory int64)`
- **Для `application/json`**: Используйте `json.NewDecoder(r.Body).Decode(&вашаСтруктура)`.

```go
// Пример для JSON
type User struct {
	Name  string `json:"name"`
	Email string `json:"email"`
}

func jsonHandler(w http.ResponseWriter, r *http.Request) {
    if r.Method != http.MethodPost {
        http.Error(w, "Метод не поддерживается", http.StatusMethodNotAllowed)
        return
    }

    var user User
    // Ограничение размера тела и декодирование JSON
    r.Body = http.MaxBytesReader(w, r.Body, 1048576) // 1MB
    decoder := json.NewDecoder(r.Body)
    decoder.DisallowUnknownFields() // Опционально: отклонять неизвестные поля

    err := decoder.Decode(&user)
    if err != nil {
        http.Error(w, "Некорректный JSON или превышение размера", http.StatusBadRequest)
        return
    }
    fmt.Fprintf(w, "Получен пользователь: Имя=%s, Email=%s\n", user.Name, user.Email)
}
```


#### Доступ к контексту запроса
**`r.Context()`** позволяет вам получить `context.Context` из запроса. Это критически важно для передачи значений (например, аутентифицированного пользователя), отмены операций (например, при тайм-ауте клиента) и отслеживания трассировки.
```go
package main

import (
	"context"
	"fmt"
	"net/http"
	"time"
)

type userKeyType string

const userKey userKeyType = "user"

func authMiddleware(next http.Handler) http.Handler {
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		// Здесь могла бы быть логика аутентификации
		// Например, чтение токена из заголовка и проверка его.

		// Допустим, мы аутентифицировали пользователя "admin"
		ctx := context.WithValue(r.Context(), userKey, "admin")
		r = r.WithContext(ctx) // Обновляем запрос новым контекстом
		next.ServeHTTP(w, r)
	})
}

func protectedHandler(w http.ResponseWriter, r *http.Request) {
	// Получаем пользователя из контекста
	user, ok := r.Context().Value(userKey).(string)
	if !ok {
		http.Error(w, "Пользователь не найден в контексте", http.StatusInternalServerError)
		return
	}

	// Пример использования контекста для отмены операции
	select {
	case <-time.After(5 * time.Second):
		fmt.Fprintf(w, "Привет, %s! Операция завершена.\n", user)
	case <-r.Context().Done():
		// Если клиент отключается или происходит тайм-аут
		fmt.Printf("Запрос отменен: %v\n", r.Context().Err())
		return
	}
}

func main() {
	mux := http.NewServeMux()
	mux.Handle("/protected", authMiddleware(http.HandlerFunc(protectedHandler)))

	fmt.Println("Сервер запущен на http://localhost:8080")
	http.ListenAndServe(":8080", mux)
}
```