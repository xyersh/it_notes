

```go
package main

import (
	"bytes"
	"encoding/gob"
	"fmt"
	"log"
)

type Person struct {
	Name string
	Age  int
}

func main() {
	// Создание данных
	p := Person{Name: "Alice", Age: 30}

	// Сериализация с использованием Gob
	var buffer bytes.Buffer
	enc := gob.NewEncoder(&buffer)
	err := enc.Encode(p)
	if err != nil {
		log.Fatalf("Ошибка сериализации: %v", err)
	}
	fmt.Println("Сериализованные данные (Gob):", buffer.Bytes())

	// Десериализация
	var decodedPerson Person
	dec := gob.NewDecoder(&buffer)
	err = dec.Decode(&decodedPerson)
	if err != nil {
		log.Fatalf("Ошибка десериализации: %v", err)
	}
	fmt.Println("Десериализованные данные (Gob):", decodedPerson)
}
```