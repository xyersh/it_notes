
### УСТАНОВКА/ЗАПУСК

#### docker
```shell

# установить образ
docker pull redis 

# создать контейнер redis-test, запустить
docker run -name redis-test -d -p 6379:6379 redis 

# проверить работу контейнера
docker ps

# зайти в cli редиса
docker exec -it redis-test redis-cli

# Выйти из оболочки REDIS: EXIT
```



###  ОСНОВНЫЕ КОНЦЕПЦИИ
- **Redis хранит данные в формате ключ-значение**, где ключ всегда является строкой, а значение может быть одним из различных типов данных.
- **Ключи (Keys):** Уникальные **строковые** идентификаторы для каждого значения. Рекомендуется использовать осмысленные и структурированные ключи (например, `user:123:name`).
- **Атомарность операций:** Большинство операций Redis являются атомарными, что гарантирует целостность данных при параллельном доступе.
- **Производительность:** Redis оптимизирован для очень быстрых операций, так как данные хранятся в оперативной памяти.


### ОСНОВНЫЕ КОМАНДЫ
- `PING` - проверка доступности REDIS ( ответ PONG)
- `select <index>` - выбрать БД по индексу 

### ТИПЫ ДАННЫХ (API)



#### Строки (Strings)
- Самый простой тип данных. Может хранить любую последовательность байтов (строки, целые числа, числа с плавающей запятой, **бинарные данные**).
- Максимальный размер: 512 МБ.
- **Основные операции:**
	- `SET key value`: Устанавливает значение для ключа.
	- `GET key`: Возвращает значение по ключу.
	- `DEL key`: Удаляет ключ.
	- `INCR key`: Увеличивает целочисленное значение на 1.
	- `DECR key`: Уменьшает целочисленное значение на 1.
	- `EXPIRE key seconds`: Устанавливает срок действия ключа в секундах.
	- `TTL key`: Возвращает оставшееся время жизни ключа.

#### Хэши (Hashes)
- Идеальны для хранения объектов. Представляют собой набор пар поле-значение.
- **Основные операции:**
    - `HSET key field value [field value ...]`: Устанавливает одно или несколько полей в хэше.
    - `HGET key field`: Возвращает значение поля из хэша.
    - `HGETALL key`: Возвращает все поля и их значения из хэша.
    - `HDEL key field [field ...]`: Удаляет одно или несколько полей из хэша.

#### Списки (Lists)
- Коллекция упорядоченных строк. Могут действовать как очереди или стеки.
- **Основные операции:**
    - `LPUSH key value [value ...]`: Добавляет одно или несколько значений в начало списка.
    - `RPUSH key value [value ...]`: Добавляет одно или несколько значений в конец списка.
    - `LPOP key`: Удаляет и возвращает первый элемент списка.
    - `RPOP key`: Удаляет и возвращает последний элемент списка.
    - `LRANGE key start stop`: Возвращает диапазон элементов списка.
    - `LLEN key`: Возвращает длину списка.

#### Множества (Sets)
- Неупорядоченная коллекция уникальных строк.
- **Основные операции:**
    - `SADD key member [member ...]`: Добавляет один или несколько элементов в множество.
    - `SMEMBERS key`: Возвращает все элементы множества.
    - `SISMEMBER key member`: Проверяет, является ли элемент членом множества.
    - `SREM key member [member ...]`: Удаляет один или несколько элементов из множества.
    - `SUNION key [key ...]`: Возвращает объединение нескольких множеств.

#### Упорядоченные Множества (Sorted Sets)
- Похожи на множества, но каждый элемент ассоциирован с числовым "очком" (score), что позволяет упорядочивать элементы.
- Идеальны для реализации списков лидеров, рейтингов.
- **Основные операции:**
    - `ZADD key score member [score member ...]`: Добавляет один или несколько элементов в упорядоченное множество.
    - `ZRANGE key start stop [WITHSCORES]`: Возвращает элементы в заданном диапазоне по индексу.
    - `ZSCORE key member`: Возвращает "очко" элемента.
    - `ZREM key member [member ...]`: Удаляет один или несколько элементов из упорядоченного множества.



### GOLANG feat REDIS

Необходимо использование пакета   `github.com/go-redis/redis/v8`

#### Акценты на самых важных концепциях применения Redis в Golang

При использовании Redis с Golang стоит уделить внимание нескольким ключевым аспектам:

- **Обработка ошибок:**
    - Всегда проверяйте ошибки, возвращаемые функциями `go-redis`. Redis — это сетевая база данных, и могут возникать проблемы с подключением, таймауты и другие ошибки.
    - `redis.Nil` - это специальная ошибка, указывающая на то, что ключ не найден. Важно ее обрабатывать отдельно от других ошибок (как показано в примере с `GET`).

- **Использование `context.Context`:**
    - Пакет `go-redis` широко использует `context.Context` для управления жизненным циклом запросов, отмены и таймаутов.
    - Рекомендуется передавать соответствующий контекст в каждую операцию Redis. В простом консольном приложении `context.Background()` или `context.TODO()` достаточны, но в реальных веб-приложениях или микросервисах необходимо использовать полноценный контекст запроса.

- **Управление подключениями (Connection Pooling):**
    - Клиент `go-redis` по умолчанию имеет встроенный пул соединений, что очень важно для производительности. **Не следует** создавать новый `redis.Client` для каждого запроса.
    - Настройте параметры пула (максимальное количество соединений, таймауты) в `redis.Options` для оптимизации производительности в зависимости от вашей нагрузки.

- **Сериализация данных:**
    - Redis хранит данные как байты. Когда вы сохраняете структуры Go (Go structs) в Redis (например, в виде JSON), **необходимо** сериализовать их в строку или байты перед записью и десериализовать после чтения.
    - Для хэшей можно использовать `HSet` с отдельными полями или сериализовать всю структуру в JSON и сохранить ее как строку.
	```go
	// Пример сериализации JSON для строки
	type User struct {
	    Name string
	    Age  int
	}
	
	// ...
	user := User{Name: "Bob", Age: 25}
	jsonData, err := json.Marshal(user)
	if err != nil {
	    log.Fatal(err)
	}
	rdb.Set(ctx, "user:200", jsonData, 0)
	
	// ...
	rawJson, err := rdb.Get(ctx, "user:200").Bytes()
	if err != nil {
	    log.Fatal(err)
	}
	var loadedUser User
	json.Unmarshal(rawJson, &loadedUser)
	fmt.Printf("Загруженный пользователь: %+v\n", loadedUser)
	```

- **Пайплайнинг и Транзакции (Pipelines & Transactions):**
	- **Пайплайнинг:** Позволяет отправить несколько команд Redis за одну операцию сетевого взаимодействия, что значительно сокращает задержки. `go-redis` поддерживает пайплайнинг через `Pipeline()`.
```go
pipe := rdb.Pipeline()
incr := pipe.Incr(ctx, "pipeline_counter")
set := pipe.Set(ctx, "pipeline_key", "value", 0)
_, err := pipe.Exec(ctx) // Выполнить все команды в пайплайне
if err != nil {
	log.Fatal(err)
}
fmt.Println("Инкремент пайплайна:", incr.Val())
fmt.Println("Установка пайплайна:", set.Val())
```

- **Транзакции (MULTI/EXEC):** Гарантируют атомарное выполнение группы команд. `go-redis` поддерживает их через `Watch` и `TxPipelined`.
```go
// Пример WATCH/MULTI/EXEC - осторожно, это более продвинутая концепция
key := "my_balance"
// Установим начальный баланс
rdb.Set(ctx, key, 100, 0)

err = rdb.Watch(ctx, func(tx *redis.Tx) error {
    n, err := tx.Get(ctx, key).Int()
    if err != nil && err != redis.Nil {
        return err
    }

    // Имитируем некоторую логику
    newVal := n - 10 // Вычитаем 10

    _, err = tx.TxPipelined(ctx, func(pipe redis.Pipeliner) error {
        pipe.Set(ctx, key, newVal, 0)
        return nil
    })
    return err
}, key) // WATCH за ключом "my_balance"

if err != nil {
    fmt.Printf("Транзакция завершилась ошибкой: %v\n", err)
} else {
    finalBalance, _ := rdb.Get(ctx, key).Int()
    fmt.Printf("Финальный баланс: %d\n", finalBalance)
}
```

- **Redis как кэш:**
    - Используйте `EXPIRE` или `SETEX` для установки срока жизни ключей. Это критически важно при использовании Redis в качестве кэша, чтобы избежать устаревания данных и переполнения памяти.
    - Реализуйте стратегию кэширования (например, "cache-aside") в своем приложении.
    
- **Удаление ключей:**
    - Помните, что данные в Redis постоянны, пока вы их не удалите или пока не истечет их срок жизни.
    - Используйте `DEL` для удаления ненужных ключей.




#### ОБЩИЙ API РАБОТЫ С REDIS
Представлен в виде рабочего приложения:
```go
package main

import (
	"context"
	"fmt"
	"log"
	"time"

	"github.com/go-redis/redis/v8"
)

var ctx = context.Background() // Контекст для операций Redis

func main() {
	// 1. Инициализация клиента Redis
	// Акцент: Подключение к Redis. Используем порт 6379, который мы пробросили из контейнера.
	rdb := redis.NewClient(&redis.Options{
		Addr:     "localhost:6379", // Адрес нашего Redis-контейнера
		Password: "",               // Пароль, если ваш Redis защищен (по умолчанию нет)
		DB:       0,                // Используемая база данных (по умолчанию 0)
	})

	// Проверка соединения
	// Акцент: Важность проверки соединения при запуске приложения.
	pong, err := rdb.Ping(ctx).Result()
	if err != nil {
		log.Fatalf("Не удалось подключиться к Redis: %v", err)
	}
	fmt.Printf("Успешное подключение к Redis! Ответ: %s\n\n", pong)

	// 2. Работа со строками (Strings)
	fmt.Println("--- Работа со Строками (Strings) ---")
	// SET
	err = rdb.Set(ctx, "mykey", "Hello from Go!", 0).Err()
	if err != nil {
		log.Fatalf("Ошибка при SET: %v", err)
	}
	fmt.Println("Установлено: mykey = 'Hello from Go!'")

	// GET
	val, err := rdb.Get(ctx, "mykey").Result()
	if err != nil {
		log.Fatalf("Ошибка при GET: %v", err)
	}
	fmt.Printf("Получено: mykey = '%s'\n", val)

	// INCR
	err = rdb.Set(ctx, "counter", "10", 0).Err()
	if err != nil {
		log.Fatalf("Ошибка при SET counter: %v", err)
	}
	newVal, err := rdb.Incr(ctx, "counter").Result()
	if err != nil {
		log.Fatalf("Ошибка при INCR: %v", err)
	}
	fmt.Printf("Значение counter после INCR: %d\n", newVal)

	// EXPIRE
	err = rdb.Set(ctx, "temporary_key", "I will expire soon!", 10*time.Second).Err() // Ключ истечет через 10 секунд
	if err != nil {
		log.Fatalf("Ошибка при SET с EXPIRE: %v", err)
	}
	fmt.Println("Установлен temporary_key со сроком действия 10 секунд.")
	ttl, err := rdb.TTL(ctx, "temporary_key").Result()
	if err != nil {
		log.Fatalf("Ошибка при TTL: %v", err)
	}
	fmt.Printf("Время жизни temporary_key: %v\n", ttl)
	fmt.Println()

	// 3. Работа с хэшами (Hashes)
	fmt.Println("--- Работа с Хэшами (Hashes) ---")
	// HSET
	userKey := "user:100"
	err = rdb.HSet(ctx, userKey, "name", "Alice", "email", "alice@example.com", "age", 30).Err()
	if err != nil {
		log.Fatalf("Ошибка при HSET: %v", err)
	}
	fmt.Printf("Создан хэш %s с полями name, email, age.\n", userKey)

	// HGET
	name, err := rdb.HGet(ctx, userKey, "name").Result()
	if err != nil {
		log.Fatalf("Ошибка при HGET name: %v", err)
	}
	fmt.Printf("Имя пользователя (HGET name): %s\n", name)

	// HGETALL
	userData, err := rdb.HGetAll(ctx, userKey).Result()
	if err != nil {
		log.Fatalf("Ошибка при HGETALL: %v", err)
	}
	fmt.Println("Все данные пользователя (HGETALL):", userData)
	fmt.Println()

	// 4. Работа со списками (Lists)
	fmt.Println("--- Работа со Списками (Lists) ---")
	listKey := "messages"
	// RPUSH (добавление в конец)
	rdb.RPush(ctx, listKey, "msg1", "msg2", "msg3").Err()
	fmt.Printf("Добавлены элементы в %s: msg1, msg2, msg3\n", listKey)

	// LRANGE (получение диапазона)
	messages, err := rdb.LRange(ctx, listKey, 0, -1).Result()
	if err != nil {
		log.Fatalf("Ошибка при LRANGE: %v", err)
	}
	fmt.Printf("Все сообщения в списке: %v\n", messages)

	// LPOP (удаление из начала)
	firstMsg, err := rdb.LPop(ctx, listKey).Result()
	if err != nil {
		log.Fatalf("Ошибка при LPOP: %v", err)
	}
	fmt.Printf("Удалено первое сообщение: %s\n", firstMsg)

	// LLEN (длина списка)
	listLen, err := rdb.LLen(ctx, listKey).Result()
	if err != nil {
		log.Fatalf("Ошибка при LLEN: %v", err)
	}
	fmt.Printf("Длина списка %s: %d\n", listKey, listLen)
	fmt.Println()

	// 5. Работа со множествами (Sets)
	fmt.Println("--- Работа со Множествами (Sets) ---")
	setKey := "tags"
	// SADD
	rdb.SAdd(ctx, setKey, "golang", "redis", "docker", "golang").Err() // "golang" будет добавлен только один раз
	fmt.Printf("Добавлены теги в %s: golang, redis, docker\n", setKey)

	// SMEMBERS
	tags, err := rdb.SMembers(ctx, setKey).Result()
	if err != nil {
		log.Fatalf("Ошибка при SMEMBERS: %v", err)
	}
	fmt.Printf("Все теги в множестве: %v\n", tags)

	// SISMEMBER
	isMember, err := rdb.SIsMember(ctx, setKey, "redis").Result()
	if err != nil {
		log.Fatalf("Ошибка при SISMEMBER: %v", err)
	}
	fmt.Printf("Является ли 'redis' членом множества? %t\n", isMember)
	fmt.Println()

	// 6. Работа с упорядоченными множествами (Sorted Sets)
	fmt.Println("--- Работа с Упорядоченными Множествами (Sorted Sets) ---")
	leaderboardKey := "leaderboard"
	// ZADD
	rdb.ZAdd(ctx, leaderboardKey, &redis.Z{Score: 100, Member: "PlayerA"},
		&redis.Z{Score: 200, Member: "PlayerB"},
		&redis.Z{Score: 150, Member: "PlayerC"}).Err()
	fmt.Printf("Добавлены игроки в %s с очками.\n", leaderboardKey)

	// ZRANGE (по возрастанию очков)
	topPlayers, err := rdb.ZRangeWithScores(ctx, leaderboardKey, 0, -1).Result()
	if err != nil {
		log.Fatalf("Ошибка при ZRANGE: %v", err)
	}
	fmt.Println("Игроки в порядке возрастания очков:")
	for _, z := range topPlayers {
		fmt.Printf("  %s: %.0f\n", z.Member, z.Score)
	}

	// ZREM
	rdb.ZRem(ctx, leaderboardKey, "PlayerA").Err()
	fmt.Println("Удален PlayerA из таблицы лидеров.")

	remainingPlayers, err := rdb.ZRangeWithScores(ctx, leaderboardKey, 0, -1).Result()
	if err != nil {
		log.Fatalf("Ошибка при ZRANGE после ZREM: %v", err)
	}
	fmt.Println("Оставшиеся игроки:")
	for _, z := range remainingPlayers {
		fmt.Printf("  %s: %.0f\n", z.Member, z.Score)
	}
	fmt.Println()

	// Очистка данных (опционально)
	fmt.Println("--- Очистка данных ---")
	rdb.Del(ctx, "mykey", "counter", "temporary_key", userKey, listKey, setKey, leaderboardKey).Err()
	fmt.Println("Все созданные ключи удалены.")
}
```