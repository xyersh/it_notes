


```go
package main  
  
import (  
    "fmt"  
    "time")  
  
type Future[T any] struct {  
    resultChan chan T  
}  
  
func NewFuture[T any](action func() T) *Future[T] {  
    future := &Future[T]{  
       resultChan: make(chan T),  
    }  
  
    go func() {  
       defer close(future.resultChan)  
       future.resultChan <- action()  
    }()  
  
    return future  
}  
  
func (f *Future[T]) Get() T {  
    return <-f.resultChan  
}  
  
func main() {  
    asyncjob := func() interface{} {  
       time.Sleep(1 * time.Second)  
       return "success"  
    }  
  
    future := NewFuture(asyncjob)  
    result := future.Get()  
  
    fmt.Println(result)  
  
}
```