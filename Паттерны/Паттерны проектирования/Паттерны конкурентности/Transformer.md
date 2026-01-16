Является декоратором каналов: Получает значение из входного канала ->  преобразует значение -> возвращает преобразованное значение в выходной канал. 
Фактически - это просто декоратор для каналов.

![[Pasted image 20250524192704.png]]

```go
package main  
  
import (  
    "fmt"  
)  
  
func Transform[T any](inChan <-chan T, action func(T) T) <-chan T {  
    outChan := make(chan T)  
    go func() {  
       defer close(outChan)  
       for in := range inChan {  
          outChan <- action(in)  
       }  
    }()  
    return outChan  
}  
  
func mul(number int) int {  
    return number * number  
}  
  
func main() {  
    ch := make(chan int)  
  
    //наполняем канал данными  
    go func() {  
       defer close(ch)  
       for i := range 10 {  
          ch <- i  
       }  
    }()  
  
    for num := range Transform(ch, mul) {  
       fmt.Println(num)  
    }  
}
```