Паттерн предназначен для замены конструкции вида:
```go 
for{
	select {
		case value, ok := <- inoutCh:
			if  !ok {
				return
			}
			//processing
		case <-doneCh:
			return
	}
}
```
на более лаконичную кострукцию:
```go
for value := range OrDone(inputCh, done) {
	//processing
}
```

### Пример
```go 
package main  
  
import (  
    "fmt"  
    "time")  
  
func OrDone[T any](inputCh chan T, done chan struct{}) <-chan T {  
    outputCh := make(chan T)  
  
    go func() {  
       defer close(outputCh)  
  
       for {  
          select {  
          case <-done:  
             return  
          default:  
          }  
  
          select {  
          case value, opened := <-inputCh:  
             if !opened {  
                return  
             }  
             outputCh <- value  
          case <-done:  
             return  
          }  
       }  
    }()  
    return outputCh  
}  
  
func main() {  
    ch := make(chan string)  
  
    //горутинка для наполнения канала значениями  
    go func() {  
       for {  
          ch <- "test"  
          time.Sleep(200 * time.Millisecond)  
       }  
    }()  
  
    //закрытие канала done для остановки работы  
    done := make(chan struct{})  
    go func() {  
       time.Sleep(1 * time.Second)  
       close(done)  
    }()  
  
    for value := range OrDone(ch, done) {  
       fmt.Println(value)  
    }  
}
```

 