### Что это такое?
`errgroup` можно рассматривать как более высокоуровневую абстракцию над `sync.WaitGroup`. В то время как `WaitGroup` просто позволяет ждать завершения группы горутин, `errgroup` добавляет важные функции:
- **Пропагация ошибок:** `Wait()` возвращает первую ошибку.
- **Отмена контекста:** При ошибке контекст автоматически отменяется, позволяя другим горутинам корректно завершиться.


Случаи применения паттерна:
- **Выполнение нескольких независимых, параллельных задач с обработкой ошибок:**
    - **Пример:** Вам нужно выполнить несколько HTTP-запросов к разным URL-адресам одновременно. Если какой-либо из запросов завершается с ошибкой, вы хотите знать об этом. `errgroup` позволяет запускать каждую загрузку в отдельной горутине и собирает первую ошибку, возвращенную любой из этих горутин.
    - **Преимущество:** Упрощает код по сравнению с ручной проверкой ошибок из каналов или использованием `sync.WaitGroup` с отдельной логикой обработки ошибок.
    
- **Отмена связанных горутин при первой ошибке:**
    - **Пример:** У вас есть группа горутин, которые выполняют взаимозависимые шаги общего процесса. Если один шаг завершается неудачей, нет смысла продолжать выполнение остальных.
    - **Принцип:** При использовании `errgroup.WithContext()`, если любая горутина в группе возвращает ошибку, связанный `context` отменяется. Горутины, которые уважают этот контекст (например, проверяя `ctx.Done()`), могут прекратить свою работу досрочно, экономя ресурсы и избегая ненужной работы.
    
- **Ожидание завершения всех горутин и получение первой ошибки:**
    - **Пример:** Вы запускаете несколько фоновых задач, и вам нужно убедиться, что все они завершились, прежде чем двигаться дальше. При этом вы также хотите знать, если какая-либо из них вернула ошибку.
    - **Как это работает:** Метод `g.Wait()` блокируется до тех пор, пока все горутины, запущенные через `g.Go()`, не завершатся. Он возвращает первую ошибку, которую вернула одна из горутин (если таковая была).
    
- **Ограничение количества одновременно выполняемых горутин (Rate Limiting):**
    - **Пример:** Вы хотите ограничить количество параллельных сетевых запросов или операций ввода-вывода, чтобы не перегружать систему или внешний сервис.
    - **Возможность:** `errgroup` предоставляет метод `SetLimit()`, который позволяет установить максимальное количество одновременно активных горутин. Это полезно для контроля потребления ресурсов.
    
- **Параллельная обработка данных (например, в цикле `for`):**
    - **Пример:** У вас есть список элементов, и вы хотите обработать каждый элемент параллельно, при этом корректно обрабатывая любые ошибки, возникающие в процессе.
    - **Важно:** При использовании `errgroup` в циклах, особенно до Go 1.22, важно правильно захватывать переменные цикла в замыканиях горутин, чтобы избежать неожиданного поведения (создавать новую переменную внутри цикла для каждого итерируемого элемента). В Go 1.22 и новее переменные цикла имеют новую семантику и автоматически захватываются по итерациям, что упрощает этот аспект.

### Стандартная реализация  
#### путь: golang.org/x/sync/errgroup

```go
package main  
  
import (  
    "context"  
    "fmt"   
	"golang.org/x/sync/errgroup"    
	"log"    
	"net/http"    
	"time")  
  
var urls = []string{"https://example.com", "https://example.org", "https://example.net"}  
  
func IsAvailable(ctx context.Context, url string) error {  
    c := http.Client{}  
    req, err := http.NewRequestWithContext(ctx, "OPTIONS", url, nil)  
    if err != nil {  
       return err  
    }  
  
    res, err := c.Do(req)  
    if err != nil {  
       return err  
    }  
  
    if res.StatusCode != 200 {  
       return fmt.Errorf("wrong status code %d", res.StatusCode)  
    }  
    time.Sleep(time.Second * 2)  
    return nil  
}  
  
func main() {  
    g, qctx := errgroup.WithContext(context.Background())  
    g.SetLimit(2)  
    for _, url := range urls {  
       g.Go(func() error {  
          return IsAvailable(qctx, url)  
       })  
    }  
    if err := g.Wait(); err != nil {  
       log.Fatal(err)  
    } else {  
       log.Println("All urls available")  
    }  
}
```


### Пример реализации Errgroup с применением канала
```go
package main  
  
import (  
    "errors"  
    "fmt"    
    "math/rand"    
    "sync"    
    "time")  
  
type ErrGroup struct {  
    err      error  
    wg       sync.WaitGroup  
    once     sync.Once  
    doneChan chan struct{}  
}  
  
func NewErrGroup() (*ErrGroup, chan struct{}) {  
    doneChan := make(chan struct{})  
    return &ErrGroup{doneChan: doneChan}, doneChan  
}  
  
func (eg *ErrGroup) Go(f func() error) {  
    eg.wg.Add(1)  
    go func() {  
       defer eg.wg.Done()  
  
       select {  
       case <-eg.doneChan:  
          return  
       default:  
          if err := f(); err != nil {  
             eg.once.Do(func() {  
                eg.err = err  
                close(eg.doneChan)  
             })  
          }  
       }  
    }()  
}  
  
func (eg *ErrGroup) Wait() error {  
    eg.wg.Wait()  
    return eg.err  
}  
  
func main() {  
    group, groupDone := NewErrGroup()  
    for i := 0; i < 5; i++ {  
       group.Go(func() error {  
          timeout := time.Second * time.Duration(rand.Intn(10))  
          timer := time.NewTimer(timeout)  
          defer timer.Stop()  
  
          select {  
          case <-groupDone:  
             fmt.Println("canceled")  
             return errors.New("canceled")  
          case <-timer.C:  
             fmt.Println("timeout")  
             return errors.New("timeout")  
          }  
       })  
    }  
    if err := group.Wait(); err != nil {  
       fmt.Println(err.Error())  
    }  
}
```