### [[JSON - база]]


### Маршалинг (сериализация) Go-структур в JSON
Маршалинг — это процесс преобразования Go-структуры (или другого типа данных Go) в JSON-представление (последовательность байтов). В Go это достигается с помощью функции `json.Marshal(v any) ([]byte, error)`.

#### Пример. Простой маршалинг
```go
package main

import (
	"encoding/json"
	"fmt"
)

type Person struct {
	Name    string
	Age     int
	IsStudent bool
}

func main() {
	p := Person{
		Name:    "Алексей",
		Age:     25,
		IsStudent: true,
	}

	// Маршалинг структуры в JSON байты
	jsonData, err := json.Marshal(p)
	if err != nil {
		fmt.Println("Ошибка маршалинга:", err)
		return
	}

	// Выводим JSON как строку
	fmt.Println(string(jsonData)) // Вывод: {"Name":"Алексей","Age":25,"IsStudent":true}
}
```

#### Пример. Форматированный маршалинг
получение  JSON в удобочитаемом формате с отступами.
```go
package main

import (
	"encoding/json"
	"fmt"
)

type Product struct {
	ID    string
	Name  string
	Price float64
}

func main() {
	prod := Product{
		ID:    "SKU123",
		Name:  "Ноутбук",
		Price: 1200.50,
	}

	// Маршалинг с отступами: префикс - пустая строка, отступ - 4 пробела
	jsonData, err := json.MarshalIndent(prod, "", "    ")
	if err != nil {
		fmt.Println("Ошибка маршалинга:", err)
		return
	}

	fmt.Println(string(jsonData))
	/*
	Вывод:
	{
	    "ID": "SKU123",
	    "Name": "Ноутбук",
	    "Price": 1200.5
	}
	*/
}
```

### Анмаршалинг (десериализация) JSON в Go-структуры
Анмаршалинг — это процесс преобразования JSON-представления (последовательности байтов) в Go-структуру. Это достигается с помощью функции `json.Unmarshal(data []byte, v any) error`.
```go
package main

import (
	"encoding/json"
	"fmt"
)

type User struct {
	Username string
	Email    string
}

func main() {
	jsonString := `{"Username":"go_dev","Email":"go_dev@example.com"}`

	var user User // Объявляем переменную типа User, куда будут десериализованы данные

	// Анмаршалинг JSON байтов в структуру
	err := json.Unmarshal([]byte(jsonString), &user) // Передаем адрес структуры
	if err != nil {
		fmt.Println("Ошибка анмаршалинга:", err)
		return
	}

	fmt.Printf("Пользователь: %+v\n", user) // Вывод: Пользователь: {Username:go_dev Email:go_dev@example.com}
	fmt.Printf("Имя пользователя: %s\n", user.Username)
}
```

**Пояснения:**
- `json.Unmarshal([]byte, interface{})`:
    - Первый аргумент — это срез байтов, содержащий JSON-данные.
    - Второй аргумент — это _указатель_ на Go-структуру (или другую переменную), куда будут десериализованы данные. Важно передать указатель, чтобы функция могла изменить значение переменной.
- В случае отсуствия json-тегов у полей структуры - поля JSON (ключи) должны точно совпадать с экспортируемыми полями Go-структуры по имени (регистр имеет значение).
- Если JSON-поле отсутствует в структуре Go, оно будет проигнорировано. 
- Если Go-поле отсутствует в JSON, оно сохранит свое нулевое значение по умолчанию.

### Настройка маршалинга/анмаршалинга с помощью тегов структур

Теги структур Go позволяют настроить, как поля структуры обрабатываются при маршалинге и анмаршалинге JSON.
Основные опции тегов:

- `json:"поле_json"`: Использует `поле_json` в качестве имени ключа в JSON.
- `json:"-"`: Игнорирует поле при маршалинге/анмаршалинге.
- `json:"omitempty"`: Пропускает поле, если его значение является нулевым (0, "", false, nil).
- `json:"строка,omitempty"`: Комбинация имени и `omitempty`.
- `json:"строка,string"`: Записывает или считывает значение как JSON-строку. Полезно для больших чисел или пользовательских форматов.

#### пример использования тегов
```go
package main

import (
	"encoding/json"
	"fmt"
)

type Config struct {
	// `json:"server_port"`: поле Go `Port` будет отображено как `server_port` в JSON
	Port int `json:"server_port"`
	// `json:"db_url,omitempty"`: поле `DBURL` будет отображено как `db_url`,
	// и пропущено, если оно пустое
	DBURL string `json:"db_url,omitempty"`
	// `json:"-"`: поле `InternalSecret` будет игнорироваться
	InternalSecret string `json:"-"`
	// `json:",string"`: поле `Version` будет маршалироваться/анмаршалироваться как строка
	Version float64 `json:"version,string"`
	// `json:"-"`: поле `_` будет игнорироваться
	_ string `json:"-"`
}

func main() {
	cfg1 := Config{
		Port:           8080,
		DBURL:          "postgres://user:pass@host:5432/db",
		InternalSecret: "very-secret-data",
		Version:        1.0,
	}

	jsonData1, _ := json.MarshalIndent(cfg1, "", "  ")
	fmt.Println("Config 1 (маршалинг):")
	fmt.Println(string(jsonData1))
	/*
	Вывод:
	Config 1 (маршалинг):
	{
	  "server_port": 8080,
	  "db_url": "postgres://user:pass@host:5432/db",
	  "version": "1"
	}
	*/

	cfg2 := Config{
		Port:    9000,
		Version: 2.5,
	} // DBURL пустой, InternalSecret не задан (будет пустая строка)

	jsonData2, _ := json.MarshalIndent(cfg2, "", "  ")
	fmt.Println("\nConfig 2 (omitempty, маршалинг):")
	fmt.Println(string(jsonData2))
	/*
	Вывод:
	Config 2 (omitempty, маршалинг):
	{
	  "server_port": 9000,
	  "version": "2.5"
	}
	*/

	// Анмаршалинг с тегами
	jsonInput := `{
		"server_port": 80,
		"db_url": "mysql://root@localhost/mydb",
		"internal_secret": "ignored",
		"version": "3.0"
	}`

	var loadedConfig Config
	err := json.Unmarshal([]byte(jsonInput), &loadedConfig)
	if err != nil {
		fmt.Println("Ошибка анмаршалинга:", err)
		return
	}

	fmt.Println("\nЗагруженная конфигурация (анмаршалинг):")
	fmt.Printf("%+v\n", loadedConfig)
	/*
	Вывод:
	Загруженная конфигурация (анмаршалинг):
	{Port:80 DBURL:mysql://root@localhost/mydb InternalSecret: Version:3}
	*/
	// Обратите внимание, InternalSecret пуст, т.к. он был проигнорирован.
}
```

```go
package main

import (
	"encoding/json"
	"fmt"
	"log"
)

// Определяем структуру с тегами JSON
type Person struct {
	Name    string `json:"name"`
	Age     int    `json:"age"`
	Address string `json:"address"`
}

func main() {
	// Создаем объект структуры
	p := Person{
		Name:    "Alice",
		Age:     30,
		Address: "123 Main St",
	}

	// Сериализация структуры в JSON
	jsonData, err := json.Marshal(p)
	if err != nil {
		log.Fatalf("Ошибка сериализации: %v", err)
	}
	fmt.Println("Сериализованные данные в формате JSON:")
	fmt.Println(string(jsonData))

	// Десериализация JSON в структуру
	var newPerson Person
	err = json.Unmarshal(jsonData, &newPerson)
	if err != nil {
		log.Fatalf("Ошибка десериализации: %v", err)
	}
	fmt.Println("\nДесериализованные данные:")
	fmt.Printf("Имя: %s, Возраст: %d, Адрес: %s\n", newPerson.Name, newPerson.Age, newPerson.Address)
}

```

### Работа с произвольными JSON-данными (`map[string]interface{}` и `json.RawMessage`)

Иногда структура JSON неизвестна заранее или может меняться. В таких случаях можно использовать `map[string]interface{}` для представления JSON-объектов или `[]interface{}` для JSON-массивов.

#### Использование `map[string]interface{}`
```go
package main

import (
	"encoding/json"
	"fmt"
)

func main() {
	jsonString := `{"event_id": "12345", "type": "click", "data": {"user_id": 1, "product_id": "P500"}}`

	// Анмаршалинг в map[string]interface{}
	var eventData map[string]interface{}
	err := json.Unmarshal([]byte(jsonString), &eventData)
	if err != nil {
		fmt.Println("Ошибка анмаршалинга:", err)
		return
	}

	fmt.Printf("Тип eventData: %T\n", eventData) // map[string]interface{}

	// Доступ к полям
	eventID, ok := eventData["event_id"].(string) // Приведение к строке
	if ok {
		fmt.Println("Event ID:", eventID)
	}

	eventType, ok := eventData["type"].(string)
	if ok {
		fmt.Println("Event Type:", eventType)
	}

	// Доступ к вложенным данным
	dataMap, ok := eventData["data"].(map[string]interface{})
	if ok {
		userID, ok := dataMap["user_id"].(float64) // Числа в JSON по умолчанию десериализуются в float64
		if ok {
			fmt.Println("User ID:", int(userID)) // Приводим к int
		}
		productID, ok := dataMap["product_id"].(string)
		if ok {
			fmt.Println("Product ID:", productID)
		}
	}

	// Маршалинг map[string]interface{} обратно в JSON
	newJSONData, _ := json.MarshalIndent(eventData, "", "  ")
	fmt.Println("\nМаршаленый обратно JSON:")
	fmt.Println(string(newJSONData))
}
```
**Пояснения:**

- При использовании `map[string]interface{}`:
    - JSON-объекты становятся `map[string]interface{}`.
    - JSON-массивы становятся `[]interface{}`.
    - Строки становятся `string`.
    - Числа становятся `float64`.
    - Булевы значения становятся `bool`.
    - `null` становится `nil`.
- Необходимо использовать утверждения типов (`.(тип)`) для безопасного доступа к значениям и их преобразования к нужным Go-типам.


#### Использование `json.RawMessage`
`json.RawMessage` — это специальный тип, который позволяет отложить десериализацию части JSON до тех пор, пока это не потребуется. Это очень полезно, когда у вас есть поле, которое может содержать разные структуры JSON, или когда вы хотите просто передать сырой JSON без его немедленной обработки.
```go
package main

import (
	"encoding/json"
	"fmt"
)

type Event struct {
	EventType string          `json:"event_type"`
	Payload   json.RawMessage `json:"payload"` // Сырые байты JSON
}

type UserLoginPayload struct {
	UserID   int    `json:"user_id"`
	Username string `json:"username"`
	IP       string `json:"ip"`
}

type ProductViewPayload struct {
	ProductID string `json:"product_id"`
	PageViews int    `json:"page_views"`
}

func main() {
	loginEventJSON := `{
		"event_type": "user_login",
		"payload": {
			"user_id": 101,
			"username": "tester",
			"ip": "192.168.1.1"
		}
	}`

	productViewEventJSON := `{
		"event_type": "product_view",
		"payload": {
			"product_id": "ABC-XYZ",
			"page_views": 5
		}
	}`

	var loginEvent Event
	err := json.Unmarshal([]byte(loginEventJSON), &loginEvent)
	if err != nil {
		fmt.Println("Ошибка анмаршалинга логин-события:", err)
		return
	}

	fmt.Printf("Тип события: %s\n", loginEvent.EventType)

	// Теперь десериализуем Payload, основываясь на EventType
	if loginEvent.EventType == "user_login" {
		var loginPayload UserLoginPayload
		err := json.Unmarshal(loginEvent.Payload, &loginPayload)
		if err != nil {
			fmt.Println("Ошибка анмаршалинга логин-пейлоада:", err)
			return
		}
		fmt.Printf("Login Payload: %+v\n", loginPayload)
	}

	var productEvent Event
	err = json.Unmarshal([]byte(productViewEventJSON), &productEvent)
	if err != nil {
		fmt.Println("Ошибка анмаршалинга просмотра продукта:", err)
		return
	}

	fmt.Printf("\nТип события: %s\n", productEvent.EventType)
	if productEvent.EventType == "product_view" {
		var productPayload ProductViewPayload
		err := json.Unmarshal(productEvent.Payload, &productPayload)
		if err != nil {
			fmt.Println("Ошибка анмаршалинга пейлоада продукта:", err)
			return
		}
		fmt.Printf("Product View Payload: %+v\n", productPayload)
	}
}
```


**Пояснения:**
- `json.RawMessage` позволяет сохранить часть JSON-документа как "сырые" байты, не пытаясь их десериализовать немедленно.
- Это позволяет вам решать, как интерпретировать `Payload` на основе других полей (например, `EventType`).
- Когда вы готовы обработать `Payload`, вы просто вызываете `json.Unmarshal()` еще раз, передавая `loginEvent.Payload` (который является `[]byte`) и указатель на соответствующую структуру.

### Пользовательские методы маршалинга/анмаршалинга
Для сложных случаев, когда стандартные теги или `map[string]interface{}` недостаточны, вы можете реализовать интерфейсы `json.Marshaler` и `json.Unmarshaler`.
- **`json.Marshaler`**: Интерфейс для пользовательского маршалинга.
```go
type Marshaler interface {
    MarshalJSON() ([]byte, error)
}
```
* **`json.Unmarshaler`**: Интерфейс для пользовательского анмаршалинга.
```go
type Unmarshaler interface {
    UnmarshalJSON([]byte) error
}
```

#### Пользовательский маршалинг `Time`
Предположим, вы хотите форматировать `time.Time` в определенный строковый формат JSON, отличный от ISO 8601, который Go использует по умолчанию.

```go
package main

import (
	"encoding/json"
	"fmt"
	"time"
)

// CustomTime обертка для time.Time для пользовательского маршалинга/анмаршалинга
type CustomTime struct {
	time.Time
}

// MarshalJSON реализует интерфейс json.Marshaler для CustomTime
func (ct CustomTime) MarshalJSON() ([]byte, error) {
	// Форматируем время в формате DD.MM.YYYY HH:MM:SS
	formattedTime := ct.Format("02.01.2006 15:04:05")
	// Возвращаем строковое представление в кавычках
	return json.Marshal(formattedTime)
}

// UnmarshalJSON реализует интерфейс json.Unmarshaler для CustomTime
func (ct *CustomTime) UnmarshalJSON(data []byte) error {
	var s string
	if err := json.Unmarshal(data, &s); err != nil {
		return fmt.Errorf("CustomTime: ожидалась строка, получено %s", data)
	}

	t, err := time.Parse("02.01.2006 15:04:05", s)
	if err != nil {
		return fmt.Errorf("CustomTime: не удалось распарсить время из '%s': %w", s, err)
	}
	ct.Time = t
	return nil
}

type Appointment struct {
	Name string     `json:"name"`
	When CustomTime `json:"when"`
}

func main() {
	now := time.Now()
	appt := Appointment{
		Name: "Встреча с клиентом",
		When: CustomTime{now},
	}

	jsonData, err := json.MarshalIndent(appt, "", "  ")
	if err != nil {
		fmt.Println("Ошибка маршалинга:", err)
		return
	}
	fmt.Println("Маршалинг с CustomTime:")
	fmt.Println(string(jsonData))
	/*
	Пример вывода:
	Маршалинг с CustomTime:
	{
	  "name": "Встреча с клиентом",
	  "when": "20.06.2025 14:22:57"
	}
	*/

	jsonInput := `{
		"name": "Прием у врача",
		"when": "21.07.2025 10:30:00"
	}`

	var loadedAppt Appointment
	err = json.Unmarshal([]byte(jsonInput), &loadedAppt)
	if err != nil {
		fmt.Println("Ошибка анмаршалинга:", err)
		return
	}

	fmt.Println("\nАнмаршалинг с CustomTime:")
	fmt.Printf("Название: %s, Время: %s\n", loadedAppt.Name, loadedAppt.When.Format("Monday, 02 January 2006"))
}
```

**Пояснения:**

- Мы создали тип `CustomTime`, который встраивает `time.Time`.
- Реализовали `MarshalJSON` для преобразования `CustomTime` в желаемый строковый формат.
- Реализовали `UnmarshalJSON` для парсинга строкового формата обратно в `time.Time`.

### Потоковая обработка JSON (декодеры/кодеры)
Функции `json.Marshal` и `json.Unmarshal` работают с целыми байтовыми срезами. Для работы с JSON-данными из `io.Reader` (например, HTTP-запрос, файл) или записи в `io.Writer` (например, HTTP-ответ, файл) более эффективны `json.Decoder` и `json.Encoder`. Они работают с потоками данных, что особенно полезно для больших JSON-документов.

#### Декодер (чтение из потока)
```go
package main

import (
	"encoding/json"
	"fmt"
	"strings"
)

type Book struct {
	Title  string `json:"title"`
	Author string `json:"author"`
	Pages  int    `json:"pages"`
}

func main() {
	jsonStream := `{"title": "The Go Programming Language", "author": "Donovan & Kernighan", "pages": 380}
	{"title": "Effective Go", "author": "Google", "pages": 50}` // Два JSON-объекта в одном потоке

	decoder := json.NewDecoder(strings.NewReader(jsonStream))

	for i := 0; decoder.More(); i++ { // Проверяем, есть ли еще JSON-значения в потоке
		var book Book
		err := decoder.Decode(&book) // Декодируем следующее JSON-значение
		if err != nil {
			fmt.Printf("Ошибка декодирования книги %d: %v\n", i+1, err)
			continue
		}
		fmt.Printf("Прочитана книга: %+v\n", book)
	}

	// Пример с одним объектом
	singleJSON := `{"title": "Mastering Go", "author": "Mihalis Tsoukalos", "pages": 700}`
	singleDecoder := json.NewDecoder(strings.NewReader(singleJSON))
	var book2 Book
	err := singleDecoder.Decode(&book2)
	if err != nil {
		fmt.Println("Ошибка декодирования одной книги:", err)
	} else {
		fmt.Printf("\nПрочитана одна книга: %+v\n", book2)
	}
}
```

#### Кодер (запись в поток)
```go
package main

import (
	"encoding/json"
	"fmt"
	"os"
)

type Movie struct {
	Title  string `json:"title"`
	Director string `json:"director"`
	Year   int    `json:"year"`
}

func main() {
	movies := []Movie{
		{Title: "Inception", Director: "Christopher Nolan", Year: 2010},
		{Title: "The Matrix", Director: "Wachowskis", Year: 1999},
	}

	// Создаем кодер, который будет писать в os.Stdout
	encoder := json.NewEncoder(os.Stdout)
	encoder.SetIndent("", "  ") // Устанавливаем отступы для читаемости

	fmt.Println("Запись фильмов в JSON:")
	for _, movie := range movies {
		err := encoder.Encode(movie) // Кодируем каждый фильм и записываем в поток
		if err != nil {
			fmt.Println("Ошибка кодирования фильма:", err)
			return
		}
	}
	/*
	Вывод:
	Запись фильмов в JSON:
	{
	  "title": "Inception",
	  "director": "Christopher Nolan",
	  "year": 2010
	}
	{
	  "title": "The Matrix",
	  "director": "Wachowskis",
	  "year": 1999
	}
	*/
}
```


### Примеры использования в реальных сценариях
#### Обработка HTTP-запроса с JSON-телом
Часто в веб-приложениях мы получаем JSON-данные в теле HTTP POST/PUT запроса.
```go
package main

import (
	"encoding/json"
	"fmt"
	"io"
	"net/http"
)

type UserRegistration struct {
	FirstName string `json:"first_name"`
	LastName  string `json:"last_name"`
	Email     string `json:"email"`
	Password  string `json:"password"`
}

func registerUser(w http.ResponseWriter, r *http.Request) {
	if r.Method != http.MethodPost {
		http.Error(w, "Метод не разрешен", http.StatusMethodNotAllowed)
		return
	}

	// Декодер для чтения JSON из тела запроса
	decoder := json.NewDecoder(r.Body)
	var reg UserRegistration

	err := decoder.Decode(&reg)
	if err != nil {
		// Обработка ошибок декодирования JSON (например, некорректный формат)
		if err == io.EOF {
			http.Error(w, "Тело запроса пусто", http.StatusBadRequest)
			return
		}
		http.Error(w, fmt.Sprintf("Ошибка декодирования JSON: %v", err), http.StatusBadRequest)
		return
	}

	// Закрываем тело запроса после чтения
	defer r.Body.Close()

	// Здесь вы обычно валидируете данные, сохраняете в базу данных и т.д.
	fmt.Printf("Получена регистрация пользователя:\n%+v\n", reg)

	// Отправляем успешный ответ
	w.Header().Set("Content-Type", "application/json")
	w.WriteHeader(http.StatusOK)
	response := map[string]string{"message": "Пользователь успешно зарегистрирован", "email": reg.Email}
	json.NewEncoder(w).Encode(response) // Кодируем ответ в JSON и пишем в ResponseWriter
}

func main() {
	fmt.Println("Сервер запущен на :8080. Отправьте POST-запрос на /register")
	fmt.Println("Пример запроса:\n curl -X POST -H \"Content-Type: application/json\" -d '{\"first_name\":\"Иван\",\"last_name\":\"Петров\",\"email\":\"ivan@example.com\",\"password\":\"password123\"}' http://localhost:8080/register")
	http.HandleFunc("/register", registerUser)
	http.ListenAndServe(":8080", nil)
}
```

#### Чтение JSON-конфигурации из файла
```go
package main

import (
	"encoding/json"
	"fmt"
	"os"
)

type AppConfig struct {
	Database struct {
		Host     string `json:"host"`
		Port     int    `json:"port"`
		User     string `json:"user"`
		Password string `json:"password"`
		DBName   string `json:"dbname"`
	} `json:"database"`
	API struct {
		Port    int    `json:"port"`
		Timeout int    `json:"timeout_seconds"`
		Key     string `json:"api_key,omitempty"`
	} `json:"api"`
	DebugMode bool `json:"debug_mode"`
}

func loadConfig(filePath string) (*AppConfig, error) {
	file, err := os.Open(filePath)
	if err != nil {
		return nil, fmt.Errorf("не удалось открыть файл конфигурации: %w", err)
	}
	defer file.Close()

	decoder := json.NewDecoder(file)
	var config AppConfig
	err = decoder.Decode(&config)
	if err != nil {
		return nil, fmt.Errorf("не удалось декодировать JSON конфигурации: %w", err)
	}
	return &config, nil
}

func main() {
	// Создадим временный файл конфигурации для примера
	configContent := `{
		"database": {
			"host": "localhost",
			"port": 5432,
			"user": "appuser",
			"password": "secretpassword",
			"dbname": "myapp_db"
		},
		"api": {
			"port": 8080,
			"timeout_seconds": 30,
			"api_key": "some-secret-api-key"
		},
		"debug_mode": true
	}`
	err := os.WriteFile("config.json", []byte(configContent), 0644)
	if err != nil {
		fmt.Println("Ошибка создания файла конфигурации:", err)
		return
	}
	defer os.Remove("config.json") // Удаляем файл после выполнения

	config, err := loadConfig("config.json")
	if err != nil {
		fmt.Println("Ошибка загрузки конфигурации:", err)
		return
	}

	fmt.Printf("Загруженная конфигурация:\n%+v\n", config)
	fmt.Printf("DB Host: %s, DB Port: %d\n", config.Database.Host, config.Database.Port)
	fmt.Printf("API Port: %d, API Timeout: %d\n", config.API.Port, config.API.Timeout)
	fmt.Printf("Debug Mode: %t\n", config.DebugMode)
}
```