### Что это такое
Паттерн **bridge** - используется когда имеется канал каналов некого  типа T **(chan chan T)** и нужно объединить входные данные в один выходной канал такого же типа T **(chan T)**

### Пример использования
паттерн имплементируется функцией `Bridge[T any](inputChCh chan chan T) <-chan T`
Функция принимает канал типа `chan chan T` и возвращает `<-chan T`

```go
package main  
  
import (  
    "fmt"  
    "sync")  
  
func Bridge[T any](inputChCh chan chan T) <-chan T {  
    outputCh := make(chan T)  
  
    go func() {  
       defer close(outputCh)  
       for inputCh := range inputChCh {  
          //go func() {  
          for value := range inputCh {  
             outputCh <- value  
          }  
          //}()  
       }  
    }()  
    return outputCh  
}  
  
func main() {  
    channnelChannel := make(chan chan string)  
    wg := sync.WaitGroup{}  
  
    wg.Add(1)  
    go func() {  
       wg.Done()  
       ch1 := make(chan string, 3)  
       for _ = range 3 {  
          //fmt.Println("1")  
          ch1 <- "channel_1"  
       }  
       close(ch1)  
  
       ch2 := make(chan string, 3)  
       for _ = range 3 {  
          //fmt.Println("2")  
          ch2 <- "channel_2"  
       }  
       close(ch2)  
  
       channnelChannel <- ch1  
       channnelChannel <- ch2  
       close(channnelChannel)  
    }()  
  
    wg.Wait()  
  
    for value := range Bridge(channnelChannel) {  
       fmt.Println(value)  
    }  
}
```