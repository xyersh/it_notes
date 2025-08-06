### Что это такое
Данный паттерн предназначен для управления отменой выполнения функции.
Такая функция дожна полчать канал для сигнала отмены/завершения (с типом struc{}) она также  также возвращает канал(struct{}) для подтверждения завершения

### Пример
```go
package main  
  
import (  
    "fmt"  
    "time")  
  
func process(closeCh <-chan struct{}) <-chan struct{} {  
    closeDoneCh := make(chan struct{})  
  
    go func() {  
       defer close(closeDoneCh)  
  
       for {  
          select {  
          case <-closeCh:  
             fmt.Println("Close on channel")  
             return  
          default:  
             //processing  
             fmt.Println("Some work")  
          }  
       }  
    }()  
    return closeDoneCh  
}  
  
func main() {  
    closeCh := make(chan struct{})  
    closeDoneCh := process(closeCh)  
    time.Sleep(time.Second)  
    close(closeCh)  
    <-closeDoneCh  
    fmt.Println("terminated")  
}
```

### Пример 2 
В данном примере каналы для закрытия  не являются частью  сигратуры какой либо фукнции, а являются полями  структуры. Во всем остпльном логика аналогична предыдущему примеру.

```go
package main  
  
import "time"  
  
type Worker struct {  
    closeCh     chan struct{}  
    closeDoneCh chan struct{}  
}  
  
func NewWorker() *Worker {  
    return &Worker{  
       closeCh:     make(chan struct{}),  
       closeDoneCh: make(chan struct{}),  
    }  
}  
  
func (w *Worker) Run() {  
    go func() {  
       ticker := time.NewTicker(time.Second)  
       defer ticker.Stop()  
       defer close(w.closeDoneCh)  
  
       for {  
          select {  
          case <-w.closeCh:  
             return  
          default:  
          }  
  
          select {  
          case <-w.closeCh:  
             return  
          case <-ticker.C:  
          }  
       }  
    }()  
  
}  
  
func (w *Worker) Shutdown() {  
    close(w.closeCh)  
    <-w.closeDoneCh  
}  
  
func main() {  
    worker := NewWorker()  
    worker.Run()  
    time.Sleep(time.Second * 5)  
    worker.Shutdown()  
}
```