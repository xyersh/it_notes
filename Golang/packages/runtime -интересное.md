

 1. `runtime.NumCPU()` - Возвращает количество логических процессоров.
 2. `runtime.MemStats`- Предоставляет информацию о статистике памяти.
 3. `runtime.GOOS` - Возвращают операционную систему
 4. `runtime.GOARCH` - Возвращает архитектуру, на которой выполняется программа.
  5. `runtime.GOMAXPROCS` - Определяет максимальное количество процессоров, которые могут использоваться параллельно:
     - runtime.GOMAXPROCS(1) - работа в один поток.
     - runtime.GOMAXPROCS(0)- количество потоков равно количеству ядер.
 * Пример:

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
6. `runtime.GC()` - Вызывает сборку мусора.
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
7. `runtime.NumGoroutine()` - Возвращает текущее количество запущенных горутин.
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
 8. `runtime.Stack()` - Возвращает стек вызовов для текущей горутины.
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
 1.  `runtime.Gosched()` - дает указание переключить выполнение на другую горутину