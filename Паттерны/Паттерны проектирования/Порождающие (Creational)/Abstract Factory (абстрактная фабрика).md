Основная цель данного паттерна — **предоставить интерфейс для создания семейств взаимосвязанных или взаимозависимых объектов без привязки к их конкретным классам**.

## Мотивация использования

1. **Изолировать создание объектов от клиентского кода**: Клиент работает с абстракциями, а не с конкретными реализациями.
2. **Гарантировать совместимость создаваемых объектов**: Все объекты создаются одной и той же фабрикой, что гарантирует их принадлежность к одному "семейству" (например, тема UI: Windows-стиль или macOS-стиль).
3. **Упростить замену целого семейства продуктов**: Достаточно подставить другую реализацию фабрики — клиентский код не изменится.

## Когда применять?

- Когда система должна быть независимой от способа создания, компоновки и представления продуктов.
- Когда в системе есть несколько семейств связанных продуктов, и нужно обеспечить, чтобы продукты одного семейства использовались вместе.
- Когда необходимо предоставить библиотеку объектов, но скрыть детали их реализации.


## Важные моменты использования паттерна
 
 #### Преимущества
- Гарантирует совместимость продуктов одного семейства.
- Упрощает добавление новых семейств (принцип открытости/закрытости).
- Инкапсулирует логику создания объектов.

####  Недостатки
- Усложняет код из-за множества дополнительных интерфейсов и классов.
- Трудно добавлять новые типы продуктов — потребуется изменять интерфейс фабрики и все её реализации.

## Пример

#### пример с кнопками и чекбоксами
Представим, что у нас есть два семейства кнопок и чекбоксов: **MacOS** и **Windows**. Мы хотим, чтобы при выборе стиля UI все элементы были согласованы.
```go
// Абстракции продуктов
type Button interface {
	Render() string
}

type Checkbox interface {
	Render() string
}

// Конкретные реализации для macOS
type MacButton struct{}
func (b MacButton) Render() string { return "MacOS Button" }

type MacCheckbox struct{}
func (c MacCheckbox) Render() string { return "MacOS Checkbox" }

// Конкретные реализации для Windows
type WinButton struct{}
func (b WinButton) Render() string { return "Windows Button" }

type WinCheckbox struct{}
func (c WinCheckbox) Render() string { return "Windows Checkbox" }

// Абстрактная фабрика
type GUIFactory interface {
	CreateButton() Button
	CreateCheckbox() Checkbox
}

// Конкретные фабрики
type MacFactory struct{}
func (f MacFactory) CreateButton() Button   { return MacButton{} }
func (f MacFactory) CreateCheckbox() Checkbox { return MacCheckbox{} }

type WinFactory struct{}
func (f WinFactory) CreateButton() Button   { return WinButton{} }
func (f WinFactory) CreateCheckbox() Checkbox { return WinCheckbox{} }

// Клиентский код
func renderUI(factory GUIFactory) {
	button := factory.CreateButton()
	checkbox := factory.CreateCheckbox()
	fmt.Println(button.Render())
	fmt.Println(checkbox.Render())
}

// Использование
func main() {
	var factory GUIFactory

	// Например, определяем ОС по флагу или конфигурации
	useMac := true
	if useMac {
		factory = MacFactory{}
	} else {
		factory = WinFactory{}
	}

	renderUI(factory)
}
```

#### пример с  разделением разных окружений приложения
```go
package main

import (
	"fmt"
	"log"
	"os"
)

// === Абстракции продуктов ===

type Database interface {
	Connect() string
}

type Logger interface {
	Log(msg string)
}

// === Конкретные реализации ===

// Development
type SQLiteDB struct{}
func (s SQLiteDB) Connect() string { return "Connected to SQLite" }

type ConsoleLogger struct{}
func (c ConsoleLogger) Log(msg string) {
	fmt.Printf("[DEV LOG] %s\n", msg)
}

// Production
type PostgreSQLDB struct{}
func (p PostgreSQLDB) Connect() string { return "Connected to PostgreSQL" }

type FileLogger struct{}
func (f FileLogger) Log(msg string) {
	// Упрощённо: пишем в stderr или файл
	log.Printf("[PROD LOG] %s", msg)
}

// === Абстрактная фабрика ===

type AppFactory interface {
	CreateDatabase() Database
	CreateLogger() Logger
}

// === Конкретные фабрики ===

type DevFactory struct{}
func (d DevFactory) CreateDatabase() Database { return SQLiteDB{} }
func (d DevFactory) CreateLogger() Logger     { return ConsoleLogger{} }

type ProdFactory struct{}
func (p ProdFactory) CreateDatabase() Database { return PostgreSQLDB{} }
func (p ProdFactory) CreateLogger() Logger     { return FileLogger{} }

// === Клиентский код ===

type Application struct {
	db     Database
	logger Logger
}

func NewApplication(factory AppFactory) *Application {
	return &Application{
		db:     factory.CreateDatabase(),
		logger: factory.CreateLogger(),
	}
}

func (app *Application) Run() {
	fmt.Println(app.db.Connect())
	app.logger.Log("Application started")
}

// === Использование ===

func main() {
	var factory AppFactory

	// Например, читаем из переменной окружения
	env := os.Getenv("APP_ENV")
	if env == "production" {
		factory = ProdFactory{}
	} else {
		factory = DevFactory{} // по умолчанию — dev
	}

	app := NewApplication(factory)
	app.Run()
}
```
