### Что это такое?
Примитив синхронизации "барьер" (Barrier) используется для координации выполнения нескольких параллельных потоков или горутин таким образом, чтобы ни один из них не мог продолжить выполнение определенной фазы работы, пока все остальные не достигнут этой же точки (барьера).

Работу барьера можно представить в следующем виде: группа бегунов стартует, спортсмены бегут до определенной отметки (первый барьер), ждут, пока все остальные не достигнут этой же отметки, затем все вместе продолжают бежать до следующей отметки (второй барьер), и так далее.

### Пример
```go
package main  
  
import (  
    "fmt"  
    "sync")  
  
type Barrier struct {  
    mu    sync.Mutex  
    count int  
    size  int  
  
    beforeCh chan struct{}  
    afterCh  chan struct{}  
}  
  
func NewBarrier(size int) *Barrier {  
    return &Barrier{  
       size:     size,  
       beforeCh: make(chan struct{}, size),  
       afterCh:  make(chan struct{}, size),  
    }  
}  
  
func (b *Barrier) Before() {  
    b.mu.Lock()  
  
    b.count++  
    if b.count == b.size {  
       for i := 0; i < b.size; i++ {  
          b.beforeCh <- struct{}{}  
       }  
    }  
  
    b.mu.Unlock()  
    <-b.beforeCh  
}  
  
func (b *Barrier) After() {  
    b.mu.Lock()  
  
    b.count--  
    if b.count == 0 {  
       for i := 0; i < b.size; i++ {  
          b.afterCh <- struct{}{}  
       }  
    }  
  
    b.mu.Unlock()  
    <-b.afterCh  
}  
  
func main() {  
    wg := sync.WaitGroup{}  
    wg.Add(4)  
  
    bootstrap := func() {  
       fmt.Println("Bootstrapping goroutines")  
    }  
  
    work := func() {  
       fmt.Println("Working goroutine")  
    }  
  
    count := 4  
    barrier := NewBarrier(count)  
    for i := 0; i < count; i++ {  
       go func() {  
          defer wg.Done()  
          //for j := 0; j < 2; j++ {  
          barrier.Before()  
          bootstrap()  
          barrier.After()  
  
          work()  
          //}  
       }()  
    }  
    wg.Wait()  
}
```