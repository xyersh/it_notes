В Go нет встроенного ключевого слова `enum` (как в C++, Java или C#). Вместо этого перечисления реализуются с помощью:
- ключевого слова **`const`**; 
- кастомного типа - для типо-безопасности 
- специального генератора последовательных целых чисел **`iota`**;
- реализации метода String() в нашем перечислении.

```go
package main

import "fmt"

// 1. Создаем собственный тип для типобезопасности нашего перечисления
type Status int

func (s Status) String() string {
    switch s {
    case StatusPending:
        return "Pending"
    case StatusActive:
        return "Active"
    case StatusClosed:
        return "Closed"
    }
    return "Unknown" // На случай невалидных значений
}

// 2. Определяем константы этого типа
const (
    StatusPending Status = iota // 0
    StatusActive                // 1
    StatusClosed                // 2
)


// Функция принимает только тип Status, передать обычный int нельзя
func ProcessStatus(s Status) {
    fmt.Println("Processing status:", s)
}

func main() {
    ProcessStatus(StatusActive)
    // ProcessStatus(1) // <- Компилятор выдаст ошибку!
}
```