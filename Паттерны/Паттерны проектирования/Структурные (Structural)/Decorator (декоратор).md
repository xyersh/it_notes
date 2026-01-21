Когда нужно динамически добавлять объекту новые обязанности (поведение), не изменяя его класс и не используя наследование.

**Цель:**  
Не меняя установленного API целевого объекта  создать обертку с таким же API, таким образом, чтобы модифицировать поведение объекта

```go
package main

import "fmt"

// Интерфейс компонента
type Coffee interface {
	Cost() float64
	Description() string
}

// Конкретный компонент
type SimpleCoffee struct{}

func (s SimpleCoffee) Cost() float64       { return 2.0 }
func (s SimpleCoffee) Description() string { return "Simple coffee" }

// Базовый декоратор
type CoffeeDecorator struct {
	coffee Coffee
}

func (d CoffeeDecorator) Cost() float64       { return d.coffee.Cost() }
func (d CoffeeDecorator) Description() string { return d.coffee.Description() }

// Конкретные декораторы
type Milk struct{ CoffeeDecorator }

func (m Milk) Cost() float64       { return m.CoffeeDecorator.Cost() + 0.5 }
func (m Milk) Description() string { return m.CoffeeDecorator.Description() + ", milk" }

type Sugar struct{ CoffeeDecorator }

func (s Sugar) Cost() float64       { return s.CoffeeDecorator.Cost() + 0.2 }
func (s Sugar) Description() string { return s.CoffeeDecorator.Description() + ", sugar" }

// Использование
func main() {
	coffee := SimpleCoffee{}
	coffee = Milk{CoffeeDecorator{coffee}}
	coffee = Sugar{CoffeeDecorator{coffee}}

	fmt.Println(coffee.Description()) // Simple coffee, milk, sugar
	fmt.Println(coffee.Cost())        // 2.7
}
```
- `SimpleCoffee` — базовый напиток.
- `Milk` и `Sugar` — декораторы, оборачивающие любой `Coffee` и добавляющие стоимость/описание.
- Можно комбинировать в любом порядке и количестве.