
1. **Создание**
- С использованием функции make():
```go
package main

import "fmt"

func main() {
    // Создаем пустой мап, где ключи - строки, а значения - целые числа
    myMap := make(map[string]int)

    // Добавляем элементы в мап
    myMap["Alice"] = 30
    myMap["Bob"] = 25
}
```

- С использованием литералов:
```go
package main 
import "fmt" 
func main() { 
// Создание мапы с начальными значениями 
myMap := map[string]int{ 
		"Alice": 30, 
		"Bob": 25, 
		"Charlie": 35, 
	} 
}
```