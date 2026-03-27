### Профилирование работающего веб-сервиса (Live Profiling)
##### Шаг 1: Код для примера
Создадим простой веб-сервер с функцией, имитирующей высокую нагрузку на CPU.

```go
package main

import (
	"log"
	"net/http"
	_ "net/http/pprof" // ВАЖНО: импорт для регистрации обработчиков pprof
)

// Имитация сложной вычислительной задачи
func heavyTask() {
	sum := 0
	for i := 0; i < 1000000000; i++ {
		sum += i
	}
}

func helloHandler(w http.ResponseWriter, r *http.Request) {
	heavyTask()
	w.Write([]byte("Task done!"))
}

func main() {
	http.HandleFunc("/hello", helloHandler)

	// Для простоты pprof будет доступен на том же порту, по пути /debug/pprof/
	log.Println("Starting server on :8080 (pprof on /debug/pprof/)")
	log.Fatal(http.ListenAndServe(":8080", nil))
}
```

##### Шаг 2: Сбор и анализ профиля

1. **Запустите сервер**: `go run main.go`
2. **Создайте нагрузку**: В другом терминале вызовите `curl http://localhost:8080/hello`.
3. **Соберите профиль CPU**: Команда подключится к серверу и будет собирать данные 30 секунд.
    
    ```bash
    go tool pprof http://localhost:8080/debug/pprof/profile
    ```
    
4. **Анализ**: В открывшейся консоли `pprof` введите команду `web`. Она сгенерирует и откроет в браузере **граф вызовов**, где самые "тяжелые" функции будут выделены. Вы сразу увидите, что `main.heavyTask` является главным потребителем CPU.
