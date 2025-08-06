### Что это такое?
Семафор это примитив синхронизации, используется для ограничения доступа к критичным ресурсам так же как и mutex.
Но если mutex предоставляет исключительный  доступ к критической секции только  одной горутине в любой момент времени, то семафор  может разрешать доступ _N_ горутинам одновременно. Например, семафор может служить для ограничения количества одновременных подключений к базе данных; ограничения количества параллельных сетевых запросов, управление пулом воркеров.

### Сводная таблица различий семафора и мутекса
|Характеристика|Мьютекс (`sync.Mutex`)|Семафор (например, `chan struct{}` или `x/sync/semaphore`)|
|:--|:--|:--|
|**Назначение**|Взаимное исключение (доступ только одному)|Управление доступом к ограниченному числу ресурсов (доступ N)|
|**Состояние**|Заблокирован/Разблокирован|Счетчик токенов (от 0 до N)|
|**Владение**|Да (горутина, заблокировавшая, должна разблокировать)|Нет (любая горутина может высвободить токен)|
|**Типичное кол-во**|1|N (где N >= 1)|
|**Реализация в Go**|`sync.Mutex`|Буферизованные каналы, `golang.org/x/sync/semaphore`|
|**Применение**|Защита общих данных, критических секций|Ограничение параллелизма, пулы ресурсов|


### Пример 1 - семафор в виде канала
```go
package main

import (
	"fmt"
	"sync"
	"time"
)

func worker(id int, sem chan struct{}) {
	sem <- struct{}{} // Приобретаем "токен" - занимаем место в семафоре
	fmt.Printf("Worker %d: Started (concurrent workers: %d)\n", id, len(sem))
	time.Sleep(2 * time.Second) // Имитируем работу
	fmt.Printf("Worker %d: Finished (concurrent workers: %d)\n", id, len(sem)-1)
	<-sem // Высвобождаем "токен" - освобождаем место в семафоре
}

func main() {
	maxWorkers := 3                       // Максимальное количество одновременных воркеров
	sem := make(chan struct{}, maxWorkers) // Буферизованный канал как семафор
	var wg sync.WaitGroup

	for i := 1; i <= 10; i++ {
		wg.Add(1)
		go func(id int) {
			defer wg.Done()
			worker(id, sem)
		}(i)
	}

	wg.Wait()
	fmt.Println("All workers finished.")
}
```
### Пример 2 - семафор в виде структуры 
```go
package main  
  
import (  
    "fmt"  
    "sync"    
    "time")  
  
type Semaphore struct {  
    tickets chan struct{}  
}  
  
func NewSemaphore(ticketsNumber int) *Semaphore {  
    return &Semaphore{make(chan struct{}, ticketsNumber)}  
}  

// семафор занимает критическую секцию 
func (s *Semaphore) Acquire() {  
    s.tickets <- struct{}{}  
}  

// семафор отпускает критическую секцию
func (s *Semaphore) Release() {  
    <-s.tickets  
}  
  
func main() {  
    wg := sync.WaitGroup{}  
    wg.Add(6)  
    semaphore := NewSemaphore(3)  
    for i := 0; i < 6; i++ {  
       semaphore.Acquire()  
       go func() {  
          defer func() {  
             semaphore.Release()  
             wg.Done()  
          }()  
          fmt.Printf("working...%d\n", i)  
          time.Sleep(time.Second * 2)  
          fmt.Printf("done %d\n", i)  
       }()  
    }  
  
    wg.Wait()  
}
```