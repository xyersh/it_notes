

## Мотивация использования Factory
1. **Сокрытие деталей создания объекта**: Клиенту не нужно знать, какой именно тип объекта создаётся.
2. **Упрощение расширения системы**: Добавление новых типов продуктов не требует изменения клиентского кода.
3. **Снижение связанности (coupling)**: Клиент взаимодействует с абстракцией (интерфейсом), а не с конкретными реализациями.
4. **Централизация логики создания**: Весь код, отвечающий за создание объектов, сосредоточен в одном месте.


## Паттерн в Go
В Go паттерн Factory особенно полезен, когда:
- Имеется необходимость работы  с несколькими реализациями одного интерфейса.
- Нужно легко подменять зависимости (например, для тестирования).
- Стоит придерживаться  чистой архитектуры (Clean Architecture, Hexagonal Architecture), где зависимости инжектятся через фабрики или конструкторы.

## Пример использования 
```go 
package main

import "fmt"

// Продукт
type Notifier interface {
	Notify(message string)
}

// Конкретные реализации
type EmailNotifier struct{}

func (e EmailNotifier) Notify(message string) {
	fmt.Printf("Sending email: %s\n", message)
}

type SMSNotifier struct{}

func (s SMSNotifier) Notify(message string) {
	fmt.Printf("Sending SMS: %s\n", message)
}

// Фабрика
func NewNotifier(notifierType string) (Notifier, error) {
	switch notifierType {
	case "email":
		return EmailNotifier{}, nil
	case "sms":
		return SMSNotifier{}, nil
	default:
		return nil, fmt.Errorf("unknown notifier type: %s", notifierType)
	}
}

// Клиентский код
func main() {
	notifier, err := NewNotifier("email")
	if err != nil {
		panic(err)
	}
	notifier.Notify("Hello via email!")
}
```