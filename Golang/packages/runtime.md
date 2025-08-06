### Типы
**runtime.MemStats** - это **структура (struct)** в пакете `runtime` языка Go, которая содержит подробную информацию о **статистике распределения памяти** вашей Go-программы. Она позволяет вам отслеживать, как ваша программа использует память, сколько памяти выделено, сколько освобождено, как часто запускался сборщик мусора и другие связанные с памятью метрики.

### Функции
- **runtime.NumCPU() int**  - Возвращает количество логических процессоров, доступных для выполнения программы Go. Это полезно для определения степени параллелизма, которую может использовать ваше приложение.
- **runtime.GOOS() string** - Возвращает имя операционной системы, на которой запущена программа
- **runtime.GOARCH() string** - Возвращает архитектуру процессора, на которой запущена программа (например, "amd64", "arm", "wasm"). Это также полезно для написания платформо-зависимого кода.
-  **runtime.Version() string** - Возвращает строку с версией Go, используемой для сборки и запуска программы.
- **runtime.Gosched()** - Передает управление планировщику, позволяя другим горутинам выполняться.
- **runtime.Goexit()** - Немедленно завершает вызывающую горутину. При этом не выполняются отложенные (`defer`) функции. Обычно рекомендуется использовать просто `return` из функции горутины.
- `runtime.LockOSThread()` - привязывает текущую горутину н потоку, на котором она выполняется
- `runtime.UnlockOSThread()` - снимает привязкугорутины к потоку
-  **runtime.ReadMemStats(m \*runtime.MemStats)** - Заполняет структуру `runtime.MemStats` информацией о статистике распределения памяти. Это полезно для мониторинга использования памяти вашей программой и понимания работы сборщика мусора.
- **runtime.GC()** - Запускает сборку мусора. Обычно сборщик мусора запускается автоматически, и принудительный вызов `runtime.GC()` не рекомендуется, так как может нарушить его эффективность. Однако в некоторых редких случаях (например, при профилировании) это может быть полезно.
- **runtime.KeepAlive(val)** - попросить не удалять переменную **val** при работе GC
- **`runtime.GOMAXPROCS()`** - Определяет максимальное количество процессоров, которые могут использоваться параллельно:
     - `runtime.GOMAXPROCS(1)` - работа в один поток.
     - `runtime.GOMAXPROCS(0)`- количество потоков равно количеству ядер.

### Примеры
- Использование runtime.MemStats для сбора статистик
```go 
package main

import (
	"fmt"
	"runtime"
	"time"
)

func main() {
	var m runtime.MemStats

	// Получаем начальную статистику
	runtime.ReadMemStats(&m)
	fmt.Println("--- Начальная статистика ---")
	printMemStats(m)

	// Выделяем немного памяти
	s := make([]byte, 1024*1024) // 1MB
	for i := 0; i < len(s); i++ {
		s[i] = byte(i % 256)
	}

	// Получаем статистику после выделения памяти
	runtime.ReadMemStats(&m)
	fmt.Println("\n--- После выделения памяти ---")
	printMemStats(m)

	// Запускаем сборку мусора
	runtime.GC()

	// Получаем статистику после сборки мусора
	runtime.ReadMemStats(&m)
	fmt.Println("\n--- После сборки мусора ---")
	printMemStats(m)

	// Еще немного выделений
	for i := 0; i < 5; i++ {
		_ = make([]int, 10000)
		time.Sleep(time.Millisecond * 100)
	}
	runtime.ReadMemStats(&m)
	fmt.Println("\n--- После дополнительных выделений ---")
	printMemStats(m)
}

func printMemStats(m runtime.MemStats) {
	fmt.Printf("Выделено (Alloc): %d байт\n", m.Alloc)
	fmt.Printf("Всего выделено (TotalAlloc): %d байт\n", m.TotalAlloc)
	fmt.Printf("Системная память (Sys): %d байт\n", m.Sys)
	fmt.Printf("Количество объектов heap (HeapObjects): %d\n", m.HeapObjects)
	fmt.Printf("Количество освобожденных объектов heap (HeapReleased): %d\n", m.HeapReleased)
	fmt.Printf("Количество сборок мусора (NumGC): %d\n", m.NumGC)
	fmt.Printf("Пауза GC (LastGC): %s\n", time.Unix(0, int64(m.LastGC)).String())
	fmt.Printf("Суммарное время GC (PauseTotalNs): %s\n", time.Duration(m.PauseTotalNs).String())
}
```


Пример:

```go
package main
import (
        "fmt"
        "runtime"
        "sync"
)

func main() {
        fmt.Println("Number of CPUs:", runtime.NumCPU())
        runtime.GOMAXPROCS(runtime.NumCPU()) // Использовать все доступные ядра

        var wg sync.WaitGroup
        for i := 0; i < 10; i++ {
                wg.Add(1)
                go func(id int) {
                        defer wg.Done()
                        fmt.Println("Goroutine", id)
                }(i)
        }
        wg.Wait()
}
```
 * Использование: Оптимизация параллельных вычислений, особенно при работе с I/O-bound задачами.
1) `runtime.GC()`: Вызывает сборку мусора.
 * Пример:
```go
package main

import (
        "fmt"
        "runtime"
        "time"
)

func main() {
        var a []byte
        for i := 0; i < 1000000; i++ {
                a = append(a, byte(i))
        }
        fmt.Println("Memory allocated:", runtime.MemStats().Alloc)

        runtime.GC() // Вызвать сборку мусора

        fmt.Println("Memory allocated after GC:", runtime.MemStats().Alloc)
}
```

 * Использование: В особых случаях, когда необходимо контролировать сборку мусора, например, при профилировании или тестировании.
1) `runtime.NumGoroutine()`: Возвращает текущее количество запущенных горутин.
 * Пример:
 ```go
package main

import (
        "fmt"
        "runtime"
        "time"
)

func main() {
        for i := 0; i < 10; i++ {
                go func() {
                        time.Sleep(time.Second)
                }()
        }

        time.Sleep(2 * time.Second)
        fmt.Println("Number of goroutines:", runtime.NumGoroutine())
}
```

 * Использование: Мониторинг параллелизма в приложениях, отладка проблем с утечкой горутин
 8) runtime.Stack(): Возвращает стек вызовов для текущей горутины.
 * Пример:
 ```go
package main

import (
        "fmt"
        "runtime"
)

func f() {
        buf := make([]byte, 1024)
        runtime.Stack(buf[:], true)
        fmt.Println(string(buf))
}

func main() {
        f()
}
```

 * Использование: Отладка, анализ стека при возникновении паники или ошибок.