## Зависимости: `golang.org/x/sync/errgroup`

### Что это такое?
**errgroup** - это примитив(паттерн?) конкурентности, который может быть полезен в случае, когда параллельно  работают несколько горутин и есть необходимость дождаться выполнения ВСЕХ горутин (как с применением sync.Wait).... НО в случае, если хотя бы одна горутина возвращает ошибку -  программа не ждет выполнения остальных, связанных в группу горутин, а прекращает выполнение всей группы горутин.

`errgroup` решает эти проблемы, предоставляя:

- **Простое управление группой горутин:** Легко запускать горутины в группе.
- **Автоматическое ожидание:** Метод `Wait()` блокируется до тех пор, пока все горутины в группе не завершатся.
- **Обработка ошибок:** Возвращает первую не-nil ошибку от любой из горутин.
- **Отмена через контекст (Context Cancellation):** Автоматически отменяет контекст группы при возникновении первой ошибки, сигнализируя остальным горутинам о необходимости завершения.

### Основные компоненты `errgroup`

1. **`errgroup.Group`**: Основной тип, представляющий группу горутин.
2. **`errgroup.WithContext(context.Context)(*Group, context.Context)`** - создание ерроргруппы с контекстом
3. **`group.Go(f func() error)`**: Запускает переданную функцию `f` в новой горутине внутри группы. Если `f` возвращает ошибку, эта ошибка будет возвращена методом `Wait()`.
4. **`group.Wait() error`**: Блокируется до тех пор, пока все функции, запущенные с помощью `Go()`, не завершатся. Возвращает первую не-nil ошибку, возвращенную одной из функций, или `nil`, если все функции завершились успешно.


### Примеры использования

#### Базовый пример
```go
package main

import (
	"fmt"
	"net/http"
	"time"

	"golang.org/x/sync/errgroup"
)

func main() {
	var g errgroup.Group
	urls := []string{
		"http://www.golang.org",
		"http://www.google.com",
		"http://www.example.com/nonexistent", // Эта ссылка вернет ошибку
	}

	for _, url := range urls {
		// Захватываем url в локальную переменную для корректной работы в замыкании
		url := url
		g.Go(func() error {
			fmt.Printf("Fetching %s \n", url)
			resp, err := http.Get(url)
			if err != nil {
				fmt.Printf("Error fetching %s: %v\n", url, err)
				return err // Возвращаем ошибку
			}
			defer resp.Body.Close()
			if resp.StatusCode != http.StatusOK {
				err := fmt.Errorf("bad status for %s: %s", url, resp.Status)
				fmt.Printf("%v\n", err)
				return err // Возвращаем ошибку
			}
			fmt.Printf("Successfully fetched %s \n", url)
			return nil // Успех
		})
	}

	// Ожидаем завершения всех горутин и проверяем на наличие ошибок
	if err := g.Wait(); err != nil {
		fmt.Printf("\nAn error occurred: %v\n", err)
	} else {
		fmt.Println("\nAll fetches completed successfully!")
	}
}
```

#### Пример с использованием контекста
```go
package main

import (
	"context"
	"fmt"
	"time"

	"golang.org/x/sync/errgroup"
)

func worker(ctx context.Context, id int, duration time.Duration, fail bool) error {
	fmt.Printf("Worker %d: started, will work for %s\n", id, duration)
	select {
	case <-time.After(duration):
		if fail {
			fmt.Printf("Worker %d: finished with error\n", id)
			return fmt.Errorf("worker %d failed", id)
		}
		fmt.Printf("Worker %d: finished successfully\n", id)
		return nil
	case <-ctx.Done():
		// Контекст был отменен
		fmt.Printf("Worker %d: cancelled\n", id)
		return ctx.Err() // Возвращаем ошибку отмены контекста
	}
}

func main() {
	// Создаем группу с контекстом. errgroup.WithContext возвращает новый контекст,
	// который будет отменен при первой ошибке или при отмене родительского контекста.
	g, ctx := errgroup.WithContext(context.Background())

	// Запускаем несколько воркеров
	g.Go(func() error {
		return worker(ctx, 1, 2*time.Second, false) // Успешный воркер
	})

	g.Go(func() error {
		return worker(ctx, 2, 1*time.Second, true) // Этот воркер упадет первым
	})

	g.Go(func() error {
		return worker(ctx, 3, 3*time.Second, false) // Этот воркер должен отмениться
	})

	fmt.Println("Waiting for workers to finish...")
	if err := g.Wait(); err != nil {
		fmt.Printf("\nAn error occurred: %v\n", err)
	} else {
		fmt.Println("\nAll workers completed successfully!")
	}
	fmt.Println("Program finished.")
```


#### Ключевые моменты в примере с контекстом:
1. `errgroup.WithContext(parentCtx)`: Создает группу и новый контекст (`ctx`), который является дочерним по отношению к `parentCtx`. Этот `ctx` будет отменен, когда:
    - Одна из горутин в группе вернет не-nil ошибку.
    - `parentCtx` будет отменен.
2. Каждая горутина (`worker`) принимает этот `ctx` и использует его в `select` для прослушивания сигнала отмены (`<-ctx.Done()`).
3. Если воркер 2 завершается с ошибкой, контекст `ctx` отменяется. Воркер 3, который еще работает, обнаружит это через `<-ctx.Done()` и завершится досрочно, вернув `ctx.Err()`.


### Пример с ограничением одновременно работающих горутин

Иногда имеется необходимость  ограничить количество горутин, работающих одновременно, чтобы избежать исчерпания ресурсов (например, при работе с файлами или сетевыми соединениями). `errgroup` предоставляет метод `SetLimit(n int)` для этой цели (начиная с Go 1.19).


```go
package main

import (
	"context"
	"fmt"
	"runtime"
	"time"

	"golang.org/x/sync/errgroup"
)

func processItem(ctx context.Context, item int) error {
	select {
	case <-ctx.Done():
		fmt.Printf("Processing item %d cancelled\n", item)
		return ctx.Err()
	case <-time.After(1 * time.Second): // Имитация работы
		fmt.Printf("Processed item %d\n", item)
		if item == 3 { // Имитация ошибки
			return fmt.Errorf("failed to process item %d", item)
		}
		return nil
	}
}

func main() {
	g, ctx := errgroup.WithContext(context.Background())

	numItems := 10
	concurrencyLimit := runtime.NumCPU() // Ограничиваем количество горутин числом CPU
	// В более новых версиях errgroup (например, golang.org/x/sync v0.7.0+)
	// можно использовать g.SetLimit(concurrencyLimit)
	// Для старых версий или для иллюстрации, можно использовать канальный семафор:
	// sem := make(chan struct{}, concurrencyLimit)

	// В версиях, поддерживающих SetLimit:
	g.SetLimit(concurrencyLimit)
	fmt.Printf("Concurrency limit set to: %d\n", concurrencyLimit)


	for i := 0; i < numItems; i++ {
		item := i // Захват переменной
		g.Go(func() error {
			// Если SetLimit не используется или для старых версий:
			// sem <- struct{}{}        // Захватываем слот
			// defer func() { <-sem }() // Освобождаем слот
			return processItem(ctx, item)
		})
	}

	if err := g.Wait(); err != nil {
		fmt.Printf("\nError during processing: %v\n", err)
	} else {
		fmt.Println("\nAll items processed successfully.")
	}
}
```

**Пояснение к `SetLimit`:**

- `g.SetLimit(n)`: Устанавливает максимальное количество горутин, которые могут быть активны одновременно в группе. Если `n < 0`, лимита нет. Если `n == 0`, используется значение по умолчанию (без лимита).
- Когда лимит достигнут, последующие вызовы `g.Go()` будут блокироваться до тех пор, пока одна из активных горутин не завершится.



### Лучшие практики и советы

- **Захват переменных цикла:** Всегда создавайте локальную копию переменной цикла перед использованием ее в замыкании горутины, чтобы избежать использования одного и того же значения во всех горутинах.
    ```go
    for _, val := range items {
        val := val // Создаем локальную копию
        g.Go(func() error {
            // используем val
            return nil
        })
    }
    ```
    
- **Возвращайте ошибки:** Убедитесь, что ваши функции, запущенные с `g.Go()`, возвращают ошибки. `nil` означает успех.
- **Используйте `errgroup.WithContext` для отмены:** Это хороший тон для длительных задач, чтобы они могли быть корректно остановлены.
- **Не игнорируйте ошибку от `Wait()`:** Всегда проверяйте ошибку, возвращаемую `g.Wait()`.
- **`SetLimit` для контроля ресурсов:** Если вы выполняете большое количество операций, которые могут исчерпать ресурсы (CPU, память, файловые дескрипторы, сетевые лимиты), используйте `g.SetLimit()`.
- **`errgroup` не для всего:** Если вам нужна сложная логика синхронизации или сбор всех ошибок (а не только первой), `errgroup` может быть недостаточен, и вам придется использовать более низкоуровневые примитивы синхронизации.