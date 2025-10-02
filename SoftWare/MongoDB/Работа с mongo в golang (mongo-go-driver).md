
`mongo-go-driver` — это официальный драйвер от MongoDB, предоставляющий idiomatic Go API для взаимодействия с базой данных. Мы рассмотрим основные CRUD-операции и продвинутые техники.


#### 1. Установка и настройка соединения
```bash
go get go.mongodb.org/mongo-driver/mongo
```

##### Создание соединения (Connection)
Соединение с MongoDB устанавливается через объект `mongo.Client`.
```go
package main

import (
	"context"
	"fmt"
	"log"
	"time"

	"go.mongodb.org/mongo-driver/mongo"
	"go.mongodb.org/mongo-driver/mongo/options"
)

// Глобальная переменная для клиента MongoDB
var client *mongo.Client

func init() {
	// 1. Устанавливаем Context с таймаутом
	ctx, cancel := context.WithTimeout(context.Background(), 10*time.Second)
	defer cancel() // Гарантируем отмену контекста

	// 2. Устанавливаем опции клиента (URI)
	clientOptions := options.Client().ApplyURI("mongodb://localhost:27017")

	// 3. Подключаемся к MongoDB
	var err error
	client, err = mongo.Connect(ctx, clientOptions)
	if err != nil {
		log.Fatal(err)
	}

	// 4. Проверяем соединение
	err = client.Ping(ctx, nil)
	if err != nil {
		log.Fatal("Не удалось подключиться к MongoDB:", err)
	}

	fmt.Println("Успешное подключение к MongoDB!")
}

func main() {
	// ... Ваш код будет здесь
}
```

#### 2. ### Работа с Базами Данных и Коллекциями

Драйвер работает через объекты **`Database`** и **`Collection`**, которые вы получаете от клиента.
##### Создание (Неявное)
В MongoDB база данных и коллекция **создаются неявно** при первой операции записи (например, `InsertOne`).

```go
func getCollection(dbName, colName string) *mongo.Collection {
	db := client.Database(dbName)
	return db.Collection(colName)
}

// Пример использования:
usersCollection := getCollection("mydb", "users") 
// Теперь usersCollection готова к использованию.
```

##### #### Удаление (Drop)
Удаление базы данных или коллекции выполняется явными методами.
```go
func dropDatabase(dbName string) {
	ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
	defer cancel()

	// Удаление всей базы данных
	err := client.Database(dbName).Drop(ctx)
	if err != nil {
		log.Println("Ошибка при удалении БД:", err)
	} else {
		fmt.Printf("База данных '%s' успешно удалена.\n", dbName)
	}
}

func dropCollection(dbName, colName string) {
	ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
	defer cancel()

	// Удаление коллекции
	err := client.Database(dbName).Collection(colName).Drop(ctx)
	if err != nil {
		log.Println("Ошибка при удалении коллекции:", err)
	} else {
		fmt.Printf("Коллекция '%s' успешно удалена.\n", colName)
	}
}
```


#### 3. CRUD-операции с Документами

Мы будем использовать простую структуру для документов:
```go
type User struct {
	ID    interface{} `bson:"_id,omitempty"` // interface{} или primitive.ObjectID
	Name  string      `bson:"name"`
	Email string      `bson:"email"`
	Age   int         `bson:"age"`
	IsActive bool     `bson:"is_active"`
}
```

##### Создание (Insert)
Используйте `InsertOne` или `InsertMany`.
```go
func createUser(collection *mongo.Collection, user User) {
	ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
	defer cancel()

	result, err := collection.InsertOne(ctx, user)
	if err != nil {
		log.Fatal(err)
	}
	fmt.Println("Добавлен новый пользователь с ID:", result.InsertedID)
}
```

##### Поиск (Find)
Используйте `FindOne` для одного документа и `Find` для нескольких. Критерии поиска задаются с помощью **`bson.D`** или **`bson.M`**.
```go
import "go.mongodb.org/mongo-driver/bson"

func findUsers(collection *mongo.Collection) {
	ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
	defer cancel()

	// 1. Фильтр: найти пользователей старше 25 лет
	filter := bson.D{{"age", bson.D{{"$gte", 25}}}} 

	// 2. Выполняем поиск
	cursor, err := collection.Find(ctx, filter)
	if err != nil {
		log.Fatal(err)
	}
	defer cursor.Close(ctx)

	// 3. Итерация по результатам
	var users []User
	if err = cursor.All(ctx, &users); err != nil {
		log.Fatal(err)
	}

	fmt.Println("\nНайденные пользователи:")
	for _, user := range users {
		fmt.Printf(" - %s (%d лет)\n", user.Name, user.Age)
	}
}
```


#####  Редактирование (Update)
Используется `UpdateOne` для изменения одного документа или `UpdateMany`.
```go
func updateUserName(collection *mongo.Collection, email string, newName string) {
	ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
	defer cancel()

	// 1. Фильтр: найти по email
	filter := bson.D{{"email", email}}

	// 2. Обновление: использовать оператор $set
	update := bson.D{{"$set", bson.D{{"name", newName}}}} 

	result, err := collection.UpdateOne(ctx, filter, update)
	if err != nil {
		log.Fatal(err)
	}

	fmt.Printf("\nОбновлено документов: %d\n", result.ModifiedCount)
}
```



##### Удаление (Delete)
Используйте `DeleteOne` или `DeleteMany`.
```go
func deleteUser(collection *mongo.Collection, email string) {
	ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
	defer cancel()

	// Фильтр: удалить по email
	filter := bson.D{{"email", email}}

	result, err := collection.DeleteOne(ctx, filter)
	if err != nil {
		log.Fatal(err)
	}

	fmt.Printf("Удалено документов: %d\n", result.DeletedCount)
}
```


#### 4. Продвинутые техники: Агрегация и Транзакции

##### Агрегация (Aggregation)
Для сложных запросов используется метод **`Aggregate`**, который принимает массив стадий (`pipeline`).
```go
func aggregateReport(collection *mongo.Collection) {
	ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
	defer cancel()

	// Pipeline: 
	// 1. $match: только активные пользователи
	// 2. $group: группировка по is_active и подсчет среднего возраста
	pipeline := []bson.D{
		{{"$match", bson.D{{"is_active", true}}}},
		{{"$group", bson.D{
			{"_id", "$is_active"},
			{"average_age", bson.D{{"$avg", "$age"}}},
			{"count", bson.D{{"$sum", 1}}},
		}}},
	}

	cursor, err := collection.Aggregate(ctx, pipeline)
	if err != nil {
		log.Fatal(err)
	}
	defer cursor.Close(ctx)

	var results []bson.M // Используем bson.M для гибкого вывода
	if err = cursor.All(ctx, &results); err != nil {
		log.Fatal(err)
	}

	fmt.Println("\nРезультат агрегации (средний возраст активных пользователей):", results)
}
```

##### Транзакции (Transactions)
Для обеспечения **атомарности** (гарантии выполнения всех или ни одной операции) при работе с несколькими документами или коллекциями используйте транзакции. Транзакции требуют использования **Replica Set** или **Sharded Cluster**.
```go
func performTransaction(client *mongo.Client, collection *mongo.Collection) {
	ctx := context.Background()

	// 1. Запускаем сессию
	session, err := client.StartSession()
	if err != nil {
		log.Fatal(err)
	}
	defer session.EndSession(ctx)

	// 2. Определяем функцию для транзакции
	_, err = session.WithTransaction(ctx, func(sessCtx mongo.SessionContext) (interface{}, error) {
		// --- Операция 1: Уменьшение баланса (гипотетический пример)
		filter1 := bson.D{{"email", "user_a@test.com"}}
		update1 := bson.D{{"$inc", bson.D{{"balance", -10}}}}
		_, err := collection.UpdateOne(sessCtx, filter1, update1)
		if err != nil {
			return nil, err
		}

		// --- Операция 2: Увеличение баланса (гипотетический пример)
		filter2 := bson.D{{"email", "user_b@test.com"}}
		update2 := bson.D{{"$inc", bson.D{{"balance", 10}}}}
		_, err = collection.UpdateOne(sessCtx, filter2, update2)
		if err != nil {
			return nil, err
		}

		fmt.Println("Транзакция успешно зафиксирована (Commit).")
		return nil, nil
	})

	if err != nil {
		fmt.Println("Транзакция отменена (Abort):", err)
	}
}
```


#### Краткое резюме

| Аспект                | Метод в Go                                           | Описание                                                           |
| --------------------- | ---------------------------------------------------- | ------------------------------------------------------------------ |
| **Соединение**        | `mongo.Connect()`                                    | Устанавливает связь с MongoDB.                                     |
| **Фильтр/Обновление** | `bson.D` / `bson.M`                                  | Используется для задания критериев запросов и обновлений.          |
| **Вставка**           | `collection.InsertOne()`                             | Добавление одного документа.                                       |
| **Поиск**             | `collection.Find()`, `collection.FindOne()`          | Извлечение данных с помощью `cursor`.                              |
| **Агрегация**         | `collection.Aggregate()`                             | Использование `pipeline` для сложного анализа данных.              |
| **Транзакции**        | `client.StartSession()`, `session.WithTransaction()` | Обеспечение атомарности операций (только для Replica Set/Cluster). |