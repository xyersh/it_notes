### Что это такое?

Данный паттерн используется для отсува из канала значений по какому либо признаку. На выходе получаем количество  значений <= количеству значений на входе 

![[Pasted image 20250524222624.png]]

### Пример
```go
package main  
  
import (  
    "fmt"  
)  
  
func Filter[T any](inChan <-chan T, predicat func(T) bool) <-chan T {  
    outChan := make(chan T)  
  
    go func() {  
       defer close(outChan)  
       for val := range inChan {  
          if predicat(val) {  
             outChan <- val  
          }  
       }  
    }()  
    return outChan  
}  
  
func isOdd(number int) bool {  
    return number%2 != 0  
}  
  
func main() {  
    ch := make(chan int)  
  
    //наполняем канал данными в отдельной горутине  
    go func() {  
       defer close(ch)  
       for i := range 10 {  
          ch <- i  
       }  
    }()  
      
    //Применим фильтр к каналу  
    for num := range Filter(ch, isOdd) {  
       fmt.Println(num)  
    }  
}
```