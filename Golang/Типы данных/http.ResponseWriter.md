### База

**`http.ResponseWriter`**- Это интерфейс, который позволяет вашему обработчику **отправлять HTTP-ответ обратно клиенту**. Через него вы можете устанавливать заголовки ответа, записывать данные в тело ответа и указывать HTTP-статус.


### API
- **`w.Header()`** - возвращает `http.Header` ( карта), который позволяет вам манипулировать заголовками ответа
- **`w.Header().Set("Content-Type", "text/plain; charset=utf-8")`**: устанавливает заголовок, перезаписывая существующий, если он есть.
- **`w.Header().Add("Set-Cookie", "my_cookie=hello; Path=/; Max-Age=3600")`**: Метод `Add` добавляет заголовок. Если такой заголовок уже существует, он будет добавлен как еще одно значение для этого заголовка (например, можно иметь несколько `Set-Cookie` заголовков).
- **`w.WriteHeader(http.StatusInternalServerError)`**: Устанавливает HTTP-статус ответа. Важно вызывать `WriteHeader` **только один раз**. Если вы не вызываете `WriteHeader` явно, то при первой записи данных в `ResponseWriter` (например, через `fmt.Fprintf` или `w.Write`), Go автоматически отправит статус `200 OK`.
    
- **`fmt.Fprintf(w, "Привет от Go сервера!\n")`**: Это наиболее распространенный способ записи данных в тело ответа. `fmt.Fprintf` работает аналогично `fmt.Printf`, но вместо вывода в консоль, он записывает данные в указанный `io.Writer` (в данном случае, `http.ResponseWriter` реализует этот интерфейс).

- **`w.Write([]byte("Это просто байты"))`**: Более низкоуровневый способ записи сырых байтов в тело ответа.

### Важное
#### `w.WriteHeader()` вызывается только один раз!

Как уже упоминалось, **`w.WriteHeader()`** устанавливает HTTP-статус. Если вы не вызываете его явно, Go автоматически установит статус `200 OK` при первой записи данных в `ResponseWriter`. **Не пытайтесь вызвать `WriteHeader` несколько раз**, это приведет к панике или ошибкам.
```go
func statusTrickHandler(w http.ResponseWriter, r *http.Request) {
	// Вариант 1: Установка статуса до записи
	if r.URL.Path == "/not-found" {
		w.WriteHeader(http.StatusNotFound) // Установили 404
		fmt.Fprintf(w, "Страница не найдена\n")
		return
	}

	// Вариант 2: Статус 200 OK будет установлен автоматически при первой записи
	fmt.Fprintf(w, "Все хорошо, статус 200 OK по умолчанию.\n")

	// Плохая практика:
	// w.WriteHeader(http.StatusOK) // Это уже не сработает или вызовет панику, если до этого был Write или WriteHeader
	// fmt.Fprintf(w, "Еще текст")
}
```

####  Заголовки должны быть установлены до записи тела
Вы можете изменять заголовки ответа, вызвав **`w.Header().Set(...)`** или **`w.Header().Add(...)`**, но эти изменения должны произойти **до того, как вы начнете записывать что-либо в тело ответа**. Как только вы пишете данные в `w` (через `fmt.Fprintf`, `w.Write` и т.д.), заголовки и статус уже отправлены клиенту.
```go
func headerTrickHandler(w http.ResponseWriter, r *http.Request) {
	// Правильно: сначала заголовки, потом тело
	w.Header().Set("Content-Type", "application/json")
	w.Header().Set("X-Powered-By", "GoLang")
	w.WriteHeader(http.StatusOK) // Опционально, можно не вызывать для 200 OK
	fmt.Fprintf(w, `{"message": "Hello, world!"}`)

	// Неправильно (заголовки будут игнорироваться, так как ответ уже начат):
	// fmt.Fprintf(w, "Какой-то текст...")
	// w.Header().Set("Content-Type", "text/plain") // Это не окажет никакого эффекта
}
```



####  Использование `http.Error` для стандартизированных ошибок
Вместо ручного вызова `w.WriteHeader()` и `fmt.Fprintf()` для каждой ошибки, используйте **`http.Error(w, сообщение, статусКод)`**. Эта функция устанавливает правильный заголовок `Content-Type: text/plain` и записывает сообщение об ошибке, а также HTTP-статус.
```go
// Вместо:
// w.WriteHeader(http.StatusBadRequest)
// fmt.Fprintf(w, "Некорректный запрос")

// Используйте:
http.Error(w, "Некорректный запрос", http.StatusBadRequest)
```


#### Потоковая передача данных
Для больших ответов (например, файлов) или для Comet-приложений (долгоживущие соединения, Server-Sent Events), `http.ResponseWriter` позволяет **потоковую передачу данных**. Просто продолжайте записывать данные, и они будут отправляться клиенту по мере их генерации. Для `Server-Sent Events` используйте `w.(http.Flusher).Flush()`.
```go
package main

import (
	"fmt"
	"net/http"
	"time"
)

func streamHandler(w http.ResponseWriter, r *http.Request) {
	w.Header().Set("Content-Type", "text/event-stream")
	w.Header().Set("Cache-Control", "no-cache")
	w.Header().Set("Connection", "keep-alive")

	flusher, ok := w.(http.Flusher)
	if !ok {
		http.Error(w, "Streaming unsupported!", http.StatusInternalServerError)
		return
	}

	for i := 0; i < 5; i++ {
		fmt.Fprintf(w, "data: Сообщение %d\n\n", i)
		flusher.Flush() // Отправляем данные клиенту немедленно
		time.Sleep(1 * time.Second)
	}
}

func main() {
	http.HandleFunc("/stream", streamHandler)
	fmt.Println("Сервер запущен на http://localhost:8080")
	http.ListenAndServe(":8080", nil)
}
```


#### Управление таймаутами HTTP/2
В HTTP/2 клиент может закрыть соединение до того, как сервер закончит отправлять ответ. В таком случае, запись в `http.ResponseWriter` может вернуть ошибку. Важно обрабатывать такие ошибки, чтобы избежать паники или утечек ресурсов. Используйте `context.Done()` для отслеживания отмены.
```go
// Пример обработки отмены см. в разделе про r.Context() выше.
// При записи в w, если контекст отменен, Write может вернуть ошибку io.ErrClosedPipe.
```