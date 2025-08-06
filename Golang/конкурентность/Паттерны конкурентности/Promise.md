### Что это такое?
Данный паттерн пришел из js
"Promise" представляет собой объект, который выступает в роли заполнителя (placeholder) для результата асинхронной операции, которая ещё не завершилась. Он обещает предоставить результат (или ошибку) в будущем.

#### Основные состояния "Promise":
1. **Pending (в ожидании):** Начальное состояние, операция ещё не завершена.
2. **Fulfilled (выполнено):** Операция успешно завершена, и "Promise" содержит результат.
3. **Rejected (отклонено):** Операция завершилась с ошибкой, и "Promise" содержит причину ошибки.


#### Ключевая идея 
"Promise" в том, что он позволяет вам прикрепить "колбэки" (функции обратного вызова) к будущему результату, вместо того чтобы блокировать текущий поток выполнения в ожидании. Когда асинхронная операция завершается, "Promise" уведомляет эти колбэки либо о результативной, либо об ошибочной завершении.

### Для чего он может использоваться?

Паттерн "Promise" очень полезен для:
1. **Упрощения асинхронного кода:** Позволяет избежать "callback hell" (ада колбэков), когда у вас есть много вложенных асинхронных операций, зависящих друг от друга. "Promise" делает асинхронный код более последовательным и читабельным.
2. **Обработки результатов асинхронных операций:** Предоставляет стандартизированный способ обработки успешных результатов и ошибок асинхронных вызовов.
3. **Композиции асинхронных операций:** Позволяет легко объединять несколько асинхронных операций в цепочки или запускать их параллельно и ждать завершения всех.
4. **Управление состоянием асинхронных запросов:** Вы можете передавать объект "Promise" как "ручку" к асинхронной работе, не раскрывая деталей её выполнения.
### Пример 1
```go
package main  
  
import (  
    "fmt"  
    "time")  
  
type result[T any] struct {  
    val T  
    err error  
}  
  
type Promise[T any] struct {  
    resultCh chan result[T]  
}  
  
func NewPromise[T any](asyncFn func() (T, error)) *Promise[T] {  
    promise := &Promise[T]{resultCh: make(chan result[T])}  
    go func() {  
       defer close(promise.resultCh)  
  
       value, err := asyncFn()  
       promise.resultCh <- result[T]{val: value, err: err}  
    }()  
  
    return promise  
}  
  
func (p *Promise[T]) Then(successFn func(T), errorFn func(error)) {  
    go func() {  
       result := <-p.resultCh  
       if result.err == nil {  
          successFn(result.val)  
       } else {  
          errorFn(result.err)  
       }  
    }()  
}  
  
func main() {  
    asyncJob := func() (string, error) {  
       time.Sleep(1 * time.Second)  
       return "Ok", nil  
    }  
    fmt.Println("Запускаем promise...")  
    promise := NewPromise[string](asyncJob)  
    fmt.Println("отвлечемся на прочие вещи....")  
    promise.Then(  
       func(value string) {  
          fmt.Println("success ", value)  
       },  
       func(err error) {  
          fmt.Println("error", err)  
       })  
  
    time.Sleep(2 * time.Second)  
}
```


### Пример 2 (посложнее)

```go
package main  
  
import (  
    "errors"  
    "fmt"    "sync"    "time")  
  
// PromiseResult содержит результат или ошибку асинхронной операцииtype PromiseResult[T any] struct {  
    Value T  
    Err   error  
}  
  
// Promise представляет собой объект Promise  
type Promise[T any] struct {  
    resultCh chan PromiseResult[T] // Канал для передачи результата/ошибки  
    once     sync.Once             // Для обеспечения однократной установки результата  
}  
  
// NewPromise создает новый Promise и запускает асинхронную функцию// producer - это функция, которая выполняет асинхронную работу и возвращает результат или ошибку  
func NewPromise[T any](producer func() (T, error)) *Promise[T] {  
    p := &Promise[T]{  
       resultCh: make(chan PromiseResult[T], 1), // Буферизованный канал на 1, чтобы не блокировать producer  
    }  
  
    go func() {  
       defer close(p.resultCh) // Закрываем канал после отправки результата  
  
       val, err := producer() // Выполняем асинхронную работу  
  
       // Устанавливаем результат только один раз       p.once.Do(func() {  
          p.resultCh <- PromiseResult[T]{Value: val, Err: err}  
       })  
    }()  
  
    return p  
}  
  
// Then позволяет прикрепить колбэки для успешного выполнения и обработки ошибок.// Возвращает новый Promise для цепочки операций.func (p *Promise[T]) Then(  
    onFulfilled func(T) (T, error), // Колбэк при успешном выполнении  
    onRejected func(error) (T, error), // Колбэк при ошибке  
) *Promise[T] {  
    return NewPromise(func() (T, error) {  
       res := <-p.resultCh // Ждем результат от предыдущего Promise  
  
       var nextVal T  
       var nextErr error  
  
       if res.Err != nil {  
          // Если была ошибка, вызываем onRejected  
          if onRejected != nil {  
             nextVal, nextErr = onRejected(res.Err)  
          } else {  
             // Если onRejected не предоставлен, просто пробрасываем ошибку  
             return res.Value, res.Err  
          }  
       } else {  
          // Если успешно, вызываем onFulfilled  
          if onFulfilled != nil {  
             nextVal, nextErr = onFulfilled(res.Value)  
          } else {  
             // Если onFulfilled не предоставлен, просто пробрасываем значение  
             return res.Value, nil  
          }  
       }  
       return nextVal, nextErr  
    })  
}  
  
// Catch позволяет прикрепить колбэк только для обработки ошибок.func (p *Promise[T]) Catch(onRejected func(error) (T, error)) *Promise[T] {  
    return p.Then(nil, onRejected) // Передаем nil для onFulfilled  
}  
  
// Await блокирует выполнение до получения результата Promise  
func (p *Promise[T]) Await() (T, error) {  
    res := <-p.resultCh  
    return res.Value, res.Err  
}  
  
// --- Пример использования ---  
  
func fetchData(url string) (string, error) {  
    fmt.Printf("Fetching data from %s...\n", url)  
    time.Sleep(2 * time.Second) // Имитация задержки сетевого запроса  
    if url == "error.com" {  
       return "", errors.New("network error: failed to connect")  
    }  
    return fmt.Sprintf("Data from %s", url), nil  
}  
  
func processData(data string) (string, error) {  
    fmt.Printf("Processing data: %s\n", data)  
    time.Sleep(1 * time.Second) // Имитация обработки данных  
    if len(data) < 10 {  
       return "", errors.New("data too short")  
    }  
    return "Processed: " + data, nil  
}  
  
func main() {  
    fmt.Println("--- Promise 1 (успех) ---")  
    promise1 := NewPromise(func() (string, error) {  
       return fetchData("example.com")  
    }).Then(func(data string) (string, error) {  
       fmt.Println("Promise 1: Data fetched successfully!")  
       return processData(data)  
    }, nil).Catch(func(err error) (string, error) {  
       fmt.Println("Promise 1: An error occurred!", err)  
       return "", err // Пробрасываем ошибку дальше  
    })  
  
    result1, err1 := promise1.Await()  
    if err1 != nil {  
       fmt.Println("Promise 1 Final Error:", err1)  
    } else {  
       fmt.Println("Promise 1 Final Result:", result1)  
    }  
    fmt.Println("--------------------------\n")  
  
    fmt.Println("--- Promise 2 (ошибка на первом этапе) ---")  
    promise2 := NewPromise(func() (string, error) {  
       return fetchData("error.com") // Здесь произойдет ошибка  
    }).Then(func(data string) (string, error) {  
       fmt.Println("Promise 2: Data fetched successfully (this won't be called)!")  
       return processData(data)  
    }, nil).Catch(func(err error) (string, error) {  
       fmt.Println("Promise 2: Caught error in Catch:", err)  
       // Можно вернуть какое-то значение по умолчанию или пробросить ошибку дальше  
       return "Default Data", nil // Возвращаем значение по умолчанию после ошибки  
    })  
  
    result2, err2 := promise2.Await()  
    if err2 != nil {  
       fmt.Println("Promise 2 Final Error:", err2)  
    } else {  
       fmt.Println("Promise 2 Final Result:", result2)  
    }  
    fmt.Println("--------------------------\n")  
  
    fmt.Println("--- Promise 3 (ошибка на втором этапе) ---")  
    promise3 := NewPromise(func() (string, error) {  
       return fetchData("another.com")  
    }).Then(func(data string) (string, error) {  
       fmt.Println("Promise 3: Data fetched successfully!")  
       return processData("short") // Здесь произойдет ошибка  
    }, nil).Catch(func(err error) (string, error) {  
       fmt.Println("Promise 3: Caught error in Catch:", err)  
       return "", err // Пробрасываем ошибку дальше  
    })  
  
    result3, err3 := promise3.Await()  
    if err3 != nil {  
       fmt.Println("Promise 3 Final Error:", err3)  
    } else {  
       fmt.Println("Promise 3 Final Result:", result3)  
    }  
    fmt.Println("--------------------------\n")  
  
    fmt.Println("--- Promise 4 (параллельные вызовы) ---")  
    // Запускаем несколько Promise параллельно  
    pA := NewPromise(func() (string, error) { return fetchData("A.com") })  
    pB := NewPromise(func() (string, error) { return fetchData("B.com") })  
  
    // WaitAll (или All) - полезная функция для Promise  
    // В Go это можно реализовать с помощью WaitGroup    var wg sync.WaitGroup  
    wg.Add(2)  
    results := make(chan PromiseResult[string], 2)  
  
    go func() {  
       defer wg.Done()  
       val, err := pA.Await()  
       results <- PromiseResult[string]{Value: val, Err: err}  
    }()  
    go func() {  
       defer wg.Done()  
       val, err := pB.Await()  
       results <- PromiseResult[string]{Value: val, Err: err}  
    }()  
  
    wg.Wait()      // Ждем завершения всех горутин, которые ждут Promise  
    close(results) // Закрываем канал после того, как все результаты получены  
  
    fmt.Println("Promise 4: All parallel fetches completed.")  
    for res := range results {  
       if res.Err != nil {  
          fmt.Println("Promise 4 Parallel Error:", res.Err)  
       } else {  
          fmt.Println("Promise 4 Parallel Result:", res.Value)  
       }  
    }  
}
```