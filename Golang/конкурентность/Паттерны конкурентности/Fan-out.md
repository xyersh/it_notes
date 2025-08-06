### Что это такое?
Один канал на входе - несколько каналов на выходе.  Данные распределяются по выходным каналам.
Допустим, мы считываем 100 чисел из входного канала, и распер

![[Pasted image 20250524105323.png]]


### Пример
```go 

package main  
  
import (  
    "fmt"  
)  
  
func FanOut(in <-chan int, num_chans int) []<-chan int {  
    chans := make([]<-chan int, num_chans)  
    for i := range chans {  
       chans[i] = pipeline(in)  
    }  
  
    return chans  
  
}  
  
func pipeline(in <-chan int) <-chan int {  
    out := make(chan int)  
    go func() {  
       for v := range in {  
          out <- sqr(v)  
       }  
       close(out)  
    }()  
    return out  
}  
  
func sqr(v int) int {  
    return v * v  
}  
  
func main() {  
    in := make(chan int)  
    go func() {  
       for i := range 100 {  
          in <- i  
       }  
       close(in)  
    }()  
  
    chans := FanOut(in, 10)  
    for _, ch := range chans {  
       for v := range ch {  
          fmt.Println(v)  
       }  
    }  
  
}
```