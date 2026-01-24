Используется,  когда нужно заставить работать вместе классы (или интерфейсы), чьи интерфейсы несовместимы.


## Суть адаптера
- есть старая реализация какого-либо функционала вместе со своим интерфейсом.  
- есть клиентский код  который может работать  только  объектами, реализующими новый интерфейс (новый функционал)
- Проблема - новое не умеет работать со старым
- Решение создать прокладку в виде структуры, класса (**Adapter**) содержащую в своем состоянии старье(объект) и имплементирующую новый интерфейс
- Итого имеем объект класса/структуры , который можно передавать в новый функционал так как
адаптер является полностью совместимым с новым функционалом.
## Пример реализации паттерна

Допустим, у нас есть старая система, которая работает с `LegacyPrinter`, имеющим метод `PrintOld(s string)`.  
Но новая система требует интерфейс `ModernPrinter` с методом `Print(s string)`.
Мы не можем менять `LegacyPrinter`, но хотим использовать его в новой системе — создаём адаптер.

```go
package main

import "fmt"

// LegacyPrinter — старый интерфейс, который нельзя менять
type LegacyPrinter interface {
	PrintOld(s string)
}

// ModernPrinter — новый интерфейс, который ожидает клиент
type ModernPrinter interface {
	Print(s string)
}

// ConcreteLegacyPrinter — конкретная реализация старого интерфейса
type ConcreteLegacyPrinter struct{}

func (c *ConcreteLegacyPrinter) PrintOld(s string) {
	fmt.Printf("Legacy: %s\n", s)
}

// Adapter — адаптер, реализующий ModernPrinter и оборачивающий LegacyPrinter
type Adapter struct {
	legacy LegacyPrinter
}

func (a *Adapter) Print(s string) {
	a.legacy.PrintOld(s) // делегирует вызов старому интерфейсу
}

// Клиентская функция, работающая только с ModernPrinter
func UseModernPrinter(p ModernPrinter) {
	p.Print("Hello from modern system!")
}

func main() {
	legacy := &ConcreteLegacyPrinter{}
	adapter := &Adapter{legacy: legacy}
	UseModernPrinter(adapter) // работает!
}
```


- `Adapter` реализует `ModernPrinter`.
- Внутри он вызывает метод `PrintOld` у `LegacyPrinter`.
- Клиент (`UseModernPrinter`) ничего не знает о старом интерфейсе — он работает только с `ModernPrinter`.