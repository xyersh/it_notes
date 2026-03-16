## Level 1
### Синтаксис Go
- Переменные и типы данных
- Управляющие конструкции в языке
- Функции и возврат нескольких переменных
- Структуры и методы
- Слайсы &  мапы
- Указатели 
- Интерфейсы
### Горутины и каналы
- Горутина vs поток ОС (легкие vs тяжёлые)
- Каналы: буферизованные и небуферизованные
- WaitGroup , Mutex , select
- Паттерны: worker pool, fan-in/fan-out, pipeline, семафор
- Остановка горутины: context или канал-сигнал

### HTTP и REST API 
- net/http — стандартный пакет
- Go 1.22+: роутинг по методу + wildcard
- Middleware: logging, auth, CORS, rate limiting
- HTTP-методы, статус-коды, REST-принципы
- Разница HTTP / TCP / UDP
- Общие знания по имеющимся сторонним роутерам

### SQL и PostgreSQL
- SELECT, INSERT, UPDATE, DELETE
- JOIN, GROUP BY, HAVING
- Индексы — составные, порядок полей
- **Транзакции** и ACID
- Подключение: database/sql + pgx
- Миграции: golang-migrate или goose
- Транзакции в Go — наизусть:
```go
tx, err := db.BeginTx(ctx, nil) 
if err := nil {
    return err
}
defer tx.Rollback() :/ страховка

_, err = tx.ExecContext(ctx,
    "UPDATE accounts SET balance = balance - $1
     WHERE id = $2", amount, fromID) 
if err := nil {
    return fmt.Errorf("debit: %w", err)
}
_, err = tx.ExecContext(ctx,
    "UPDATE accounts SET balance = balance + $1
     WHERE id = $2", amount, toID) 
if err := nil {
    return fmt.Errorf("credit: %w", err)
}
return tx.Commit()
```


### Git
Минимум:
	- Коммиты, пуши, пулы
	- Ветки и мерджи
	- Pull Request workflow
	- 
UI-инструменты (GitKraken, GitHub Desktop) — ок, но 
принципы нужно понимать: что такое коммит, зачем 
ветки, как работает merge.
- learngitbranching.js.org — Git в формате игры, прямо в браузере, с визуализацией
- Rebase и cherry-pick — это потом. На старте — только основы.

### Тестирование
- Пакет testing — достаточно для старта
- Table-driven tests — Go-стиль, спрашивают на собесах
- Каждый test case — с именем, чтобы при падении сразу видеть какой упал
- Структуру наизусть: массив test cases, цикл, t.Run
```go
func TestAdd(t *testing.T) {
    tests := []struct { 
        name     string
        a, b     int
        expected int
    }{
        {"positive", 2, 3, 5},
        {"negative", -1, -2, -3}, 
        {"zero", 0, 0, 0},
        {"mixed", -1, 5, 4},
    }
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            got := Add(tt.a, tt.b)
            require.Equal(t, tt.expected, got)
        })
    }
}
```
## Level 2
Реальная работа в команде

### Docker
- Dockerfile — multi-stage build обязателен
- Образы, контейнеры, volumes, порты
- .dockerignore
- Docker Compose — локальная разработка
**Зачем multi-stage?**
Build-зависимости не должны попадать в production: 800 MB > 15 MB:
```dockerfile
# --- Build stage ---
FROM golang:1.26-alpine AS builder 
WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 go build \
    -o /bin/service ./cmd/service

# --- Production stage ---
FROM alpine:3.19
COPY --from=builder /bin/service /bin/service 
EXPOSE 8080
CMD ["/bin/service"]
```


###  Обработка ошибок — Go-стиль
- `if err != nil` — визитная карточка Go
- Error wrapping через %w :
```go
// Плохо — потеряли контекст 
if err /= nil {
    return err
}

// Хорошо — добавили информацию 
if err /= nil {
    return fmt.Errorf(
        "get user by id %d: %w",
        userID, err,
    )
}
```
 Каждый слой добавляет контекст: Handler > Service > Repo > DB


 - Sentinel errors и кастомные типы:
```go
// Sentinel error
var ErrNotFound = errors.New("not found")

// Проверка по значению
if errors.Is(err, ErrNotFound) {
    // вернуть 404
}

// Проверка по типу
var validErr *ValidationError 
if errors.As(err, &validErr) {
    // достать validErr.Field
}
```

- `errors.Is` — по значению (sentinel)
- `errors.As` — по типу (кастомные)
- Антипаттерн: log.Fatal(err) в бизнес-логике


### Архитектура проекта — три слоя
- **Стандарт для Go-проектов:**
	1. Handler — transport-слой. HTTP/gRPC запрос,  валидация, ответ
	2. Service — бизнес-логика. НЕ знает про HTTP и БД
	3. Repository — работа с данными. БД, кеш, хранилища

- Зависимости через интерфейсы — Dependency Inversion. Service зависит от интерфейса:
``` go
type UserRepository interface { 
    GetByID(ctx context.Context,
        id int64) (*User, error) 
    Create(ctx context.Context,
        user *User) (int64, error)
}

type UserService struct {
    repo UserRepository // интерфейс!
}

func NewUserService(
    repo UserRepository,
) *UserService {
    return &UserService{repo: repo}
}
```
Под капотом: PostgreSQL, mock в тестах, или in-memory



### Redis — кеширование
In-memory хранилище
- Применение: кеш, сессии, rate limiting, очереди
- Минимум: GET, SET, DEL, TTL, инвалидация
- Библиотека: go-redis/redis v9
- Паттерн Cache-Aside:
	1. Проверяем кеш
	2. Miss > идём в БД
	3. Кладём результат в кеш с TTL 
	4. Следующий запрос > Hit!
```go
func (s *UserService) GetByID(
    ctx context.Context, id int64,
) (*User, error) {
    key := fmt.Sprintf("user:%d", id)
    // 1. Проверяем кеш
    cached, err := s.cache.Get(ctx, key) 
    if err := nil {
        return cached, nil
    }
    // 2. Miss — идём в БД
    user, err := s.repo.GetByID(ctx, id) 
    if err := nil {
        return nil, fmt.Errorf("get user: %w", err) 
    }
    // 3. Кладём в кеш с TTL
    _ = s.cache.Set(ctx, key, user, 10*time.Minute) 
    return user, nil
}
```
 Если при "Cache-Aside" Redis упал? Приложение работает — просто медленнее (fallback 
на БД)



### Внешние API 
- net/http клиент с таймаутами (обязательно!) Без таймаута запрос может висеть бесконечно
- Retry с экспоненциальным backoff
- Обработка ошибок: 4xx и 5xx
- Два уровня таймаутов:
```go
client := &http.Client{
    Timeout: 5 * time.Second, :/ глобальный
}

ctx, cancel := context.WithTimeout( 
    ctx, 3*time.Second) :/ на вызов
defer cancel()

req, _ := http.NewRequestWithContext(
    ctx, "GET", url, nil) 
resp, err := client.Do(req)
```

- Anti-corruption layer:
Не тащи чужие модели в свой код! 
Service (ваш код, ваши типы) <-> Adapter (конвертация типов) <-> External API (чужой код, JSON/XML)
При такой схеме  - Внешний API изменился — меняешь один файл, а не весь проект


### Kafka — основы 

- Асинхронное взаимодействие между сервисами
- Сервис A не вызывает B напрямую. Отправляет событие 
в Kafka. B читает, когда готов.
- HTTP / gRPC — нужен ответ прямо сейчас
- Kafka — доставлю когда-нибудь, но точно
Что знать:
	- Producer и consumer
	- Топики, consumer groups
	- Гарантии: at-least-once
	- Библиотеки: sarama или kafka-go
- Простейший producer:
```go
producer, err := sarama.NewSyncProducer([]string{"localhost:9092"}, nil)
if err := nil { 
    log.Fatal(err)
}
defer producer.Close()

msg := &sarama.ProducerMessage{
    Topic: "user-events",
    Key:   sarama.StringEncoder(
        fmt.Sprintf("user-%d", userID)), 
    Value: sarama.ByteEncoder(eventJSON),
}
partition, offset, err := 
    producer.SendMessage(msg)
```
Key — partition key. Все события с одним ключом попадут в одну partition. 
Важно для порядка.



## Level 3

### Микросервисная архитектура 
Production мышление — из junior в middle
- Не серебряная пуля — это инструмент:
	- Монолит — почти всегда лучше на старте
	- Модульный монолит — простота + структура
	- Микросервисы — когда реально припёрло

- Когда оправданы:
	- Команды блокируют друг друга на деплое
	- Разные части масштабируются по-разному
	- Нужна реальная независимость деплоя

- Правила:
	- Границы — по бизнес-домену (bounded context)
	- Каждый сервис — своя БД (shared nothing)
	- Синхронное: gRPC / HTTP
	- Асинхронное: Kafka
- Anti-pattern: Distributed Monolith: Разрезали на сервисы, но зависимости остались. Все
минусы обоих миров.
- Тренд: Amazon, Shopify возвращаются к монолиту


### gRPC
Для общения сервисов внутри системы

|     |   REST  | gRPC |
| --- | --- | --- |
| Формат    | JSON (текст)    | Protobuf (бинарный)    |
| Размер   |  ~1 KB    |  ~200 bytes
| Контракт | OpenAPI (опц.) | Proto-файл (строгий) |
| Протокол HTTP/1.1 | HTTP/2 |

- Что знать:
	- Protobuf — описание контракта
	- Кодогенерация: buf (современный) или protoc
	- Unary и streaming вызовы
	- Interceptors (middleware для gRPC)

- Proto-файл = контракт:
``` protobuf
syntax = "proto3"; 
package user;

service UserService {
    rpc GetUser(GetUserRequest)
        returns (GetUserResponse); 
    rpc ListUsers(ListUsersRequest)
        returns (stream UserResponse);
}

message GetUserRequest { int64 id = 1; }

message GetUserResponse {
    int64 id = 1; 
    string name = 2; 
    string email = 3;
}
```
Контракт поменяли — код не компилируется. Строже, чем REST.

### Kafka — глубже
Подводные камни:
- Partitions и ordering — порядок ТОЛЬКО внутри одной  partition. Partition key = user_id
- Consumer groups — Kafka распределяет partitions между инстансами. Rebalancing при падении
- Гарантии: At-most-once (потеря) / At-least-once (дубли) / Exactly-once (дорого)

Практика: at-least-once + идемпотентность

Идемпотентный consumer:
```go
for msg := range claim.Messages() {
    eventID := getEventID(msg)
    // Уже обрабатывали?
    if c.repo.IsProcessed(ctx, eventID) {
        session.MarkMessage(msg, "") 
        continue
    }
    // Обработка + отметка в 1 транзакции 
    c.repo.InTx(ctx, func(tx *sql.Tx) error {
        c.processEvent(ctx, tx, msg)
        return c.repo.MarkProcessed(ctx, tx, eventID)
    })
    session.MarkMessage(msg, "")
}
```
Kafka доставит дважды — это ок. Наша задача — обработать безопасно.

### Observability
- Три столпа — работают как цепочка:
	1. Метрики. ЧТО сломалось? 
	  RED: Rate / Errors / Duration
	  Prometheus, Grafana
	2. Трейсинг. ГДЕ проблема?
	   Путь запроса через все сервисы
	   Jaeger / Tempo
	3. Логи. ДЕТАЛИ проблемы. Фильтрация по request_id  
	   ELK / Loki
- Цепочка: Метрика сигнализирует -> Трейсинг локализует -> Логи дают детали
- **OpenTelemetry** — единый стандарт (метрики + трейсинг + логи)


### Graceful Shutdown и Health Checks
#### Graceful Shutdown
- При деплое — SIGTERM:
	- Перестать принимать новые запросы
	- Дождаться завершения текущих
	- Закрыть соединения (БД, Kafka)
	- Выключиться
```go
go func() { server.ListenAndServe() }()

quit := make(chan os.Signal, 1) 
signal.Notify(quit,
    syscall.SIGTERM, syscall.SIGINT) 
:-quit

ctx, cancel := context.WithTimeout( 
    context.Background(), 30*time.Second)
defer cancel() 
server.Shutdown(ctx)
```

#### Health Checks
- Два типа проб:
	- Liveness — процесс жив? Нет > перезапуск
	- Readiness — готов к трафику? Нет > не шлём
- Liveness ok, Readiness not ready? Сервис жив, но не подключился к базе
- Readiness без проверки БД = пятисотки при падении базы

### CI/CD
 Автоматизация деплоя:
 - CI — Continuous Integration
	- Каждый push: тесты, линтер, сборка
	- Сломано — узнаёшь за минуты
- CD — Continuous Deployment
	- Код прошёл CI > автодеплой на staging/prod
- Для Go:
	- golangci-lint — линтер, ловит баги до ревью
	- go test -race ./... — с race-детектором
	- Multi-stage Docker build
- Middle-уровень: уметь настроить пайплайн и починить, 
когда сломался
- Минимальный GitHub Actions:
```yml
name: CI
on: [push, pull_request] 
jobs:
  ci:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4 
      - uses: actions/setup-go@v5
        with:
          go-version: '1.26'
      - run: golangci-lint run ./... 
      - run: go test -race ./...
      - run: docker build \
             -t myservice:${{ github.sha }} .
```
9 строк — линтер + тесты + Docker-сборка. Каждый push — автоматически.


### Kubernetes
Минимум для разработчика:
- Pod — инстанс сервиса
- Deployment — реплики, стратегия обновления
- Service — как сервисы находят друг друга
- ConfigMap / Secret — конфигурация
- Resources — CPU и Memory limits
- OOMKill — контейнер превысил memory limit > K8s убивает pod. Причины: утечка памяти, неправильные лимиты, недооценка нагрузки
- Минимальный Deployment:
```yml
apiVersion: apps/v1 
kind: Deployment 
spec:
  replicas: 3 
  template:
    spec:
      containers:
        - name: user-service
          image: user-service:v1.2.3
          resources:
            requests: { memory: "128Mi", cpu: "100m" } 
            limits: { memory: "256Mi", cpu: "500m" }
          livenessProbe:
            httpGet: { path: /health/live, port: 8080 } 
          readinessProbe:
            httpGet: { path: /health/ready, port: 8080 }
```


### Тестирование middle-уровня
- Пирамида тестов:
	- Unit — моки, быстро, изолированно. на каждый push
	- Integration — реальная БД (Testcontainers)/ перед деплоем
	- E2E — на staging
	- Contract — proto-файл не ломается

- Integration тест с Testcontainers:
```go
func TestUserRepo_Create(t *testing.T) {
    ctx := context.Background() 
    pgContainer, err := postgres.Run(ctx,
        "postgres:16",
        postgres.WithDatabase("test")) 
    require.NoError(t, err)
    defer pgContainer.Terminate(ctx)

    connStr, _ := 
        pgContainer.ConnectionString(ctx)
    db, _ := sql.Open("pgx", connStr) 
    runMigrations(db)

    repo := NewUserRepo(db)
    id, err := repo.Create(ctx,
        &User{Name: "test"})
    require.NoError(t, err) 
    require.NotZero(t, id)
}
```
Реальная БД в Docker, запуск за секунды



## Level 4

### System Design
- Отдельный этап на собесах в Яндекс, Ozon, Авито, VK
- Задача: спроектируй URL shortener / чат / ленту за 45 мин
- Что тренировать:
	- Оценка нагрузки (RPS, storage)
	- Выбор БД: SQL vs NoSQL
	- Кеширование: где, что, на сколько
	- Масштабирование: гориз. vs верт.
	- Trade-offs: consistency vs availability (CAP)
- Чек-лист первых 5 минут:
	- Требования: кто, сколько, нагрузка
	- Расчёты: RPS, объём, bandwidth
	- API: эндпоинты, вход/выход
	-  High-level: компоненты и связи
	- Deep dive: БД, кеш, очереди

### Паттерны распределённых систем
- CQRSQ:  Разделяй запись и чтение. Write (PostgreSQL) > Read (Redis)
- Event Sourcing: Храни историю событий, не текущее состояние. Мощно, но сложно.
- Saga: Транзакция через несколько сервисов. Choreography или Orchestration + компенсации.
- Circuit Breaker: Сервис не отвечает > размыкаем цепь. Closed > Open > Half-Open.
- Transactional Outbox: Атомарно: записать в БД + отправить событие. Пишем в outbox в той же TX > relay публикует.
- Inbox: Зеркальный паттерн: получил событие > inbox > дедупликация. Это идемпотентность из уровня 3.

Не заучивать, а понимать: когда какой паттерн 
уместен и какие trade-offs.

### Продвинутая работа с данными
- SQL vs NoSQL — когда что:
	

| БД         | Когда                       |
| ---------- | --------------------------- |
| PostgreSQL | Транзакции, сложные запросы |
| MongoDB    | Гибкая схема, документы     |
| ClickHouse | Аналитика                   |
| Cassandra  | Масштаб записи              |
|            |                             |

- Шардирование — одна БД не тянет
	- user_id: быстрые запросы, но горячие точки
	- order_id: равномерно, но scatter-gather
- Репликация:
	- Master-Slave: пишем в master, читаем с реплик
	- Master-Master: сложнее, конфликты
- Миграции без даунтайма:
	1.  `ADD COLUMN phone TEXT` (мгновенно)
	2. Начинаем писать phone в новый код
	3. Бэкфилим батчами по 10K
	4. Когда заполнено > `ADD NOT NULL
	 ИТОГО: 3 деплоя вместо 1 — зато 0 даунтайма


### Перформанс и профилирование
Go даёт мощные инструменты из коробки
pprof — одна строка:
```go
import _ "net/http/pprof"

func main() {
    go http.ListenAndServe(":6060", nil)
    startServer()
}
```
Доступно:
	- CPU profile — что жрёт процессор
	- Memory profile — утечки, аллокации
	- Goroutine dump — утечка горутин
	- Flame graph — визуализация
```bash
go tool pprof http://service:6060/debug/pprof/heap`
```

Нагрузочное тестирование:
- Инструменты: k6, Яндекс.Танк, Gatling
- Сервис +3x памяти после деплоя? pprof heap: до и после. Часто: утечка горутин или
unbounded кеш.
- Senior навык: подключиться к production, снять профиль, найти bottleneck.
- Вопросы к себе:
	- Какой RPS держит мой сервис?
	- Где он сломается?
	- Senior знает ответы ДО production.

### SLI / SLO / SLA
- SLI — конкретная метрика: 99.2% < 200ms
- SLO — внутренняя цель: хотим 99.9% < 200ms
- SLA — юридическое обязательство: нарушил = 
штрафы

Error Budget: SLO 99.9% > 0.1% на ошибки. Исчерпан = стоп фичам.

| SLO    | Даунтайм/мес | Даунтайм/год |
| ------ | ------------ | ------------ |
| 99.9%  | 43 мин       | 8.7 часов    |
| 99.95% | 22 мин       | 4.4 часа     |
| 99.99% | 4.3 мин      | 52 мин       |
|        |              |              |
|        |              |              |
99.9% — один неудачный деплой и бюджет кончился      
Для 4 девяток: автоматический rollback, canary deployments, circuit 
breakers.
Зачем senior это знает? Принимает решения: деплоить или нет, 
рискованный ли релиз.

### Платформы
Россия:
- Яндекс, Ozon, Авито, VK — свои платформы
- Навык: гибкость, умение адаптироваться
- Каждая компания — свой стек инфры

Мировой рынок:
- AWS, GCP, Azure
- Terraform, Infrastructure as Code
- Навык: облачные сертификации

Удалёнка за валюту — потрать время на AWS/GCP. Это 
инвестиция в доход.
Не надо знать ВСЁ. Достаточно уверенно ориентироваться в одном 
облаке и понимать принципы.


### Soft Skills
- Senior = не только код
- Коммуникация — договариваться, отстаивать решения, доносить мысль
- ADR (Architecture Decision Records) — записывай не только ЧТО решили, но ПОЧЕМУ
- Visibility — умей показать результат работы
- Менторство — растить других = расти самому

- Technical Skills x Visibility = Career Growth

Что отличает senior от strong middle:
- Middle: «Я написал сервис»
- Senior: «Я решил проблему X, это сэкономило Y часов команде, вот ADR с обоснованием»
- Хорошему специалисту, которого никто не видит — платят средне. 
Который умеет показать результат — хорошо.
- Soft skills — не про «быть приятным». Это про эффективность.

### Карьерный рост
Рынок 2025-2026:
- Конкуренция в IT выросла вдвое за год
- Оценка зарплаты: 10/10 (2022) > 5-6/10 (2025) 
- Медианная зарплата: рост всего +2%
- 55% считают: повысить ЗП = сменить работодателя

Прыгать каждые полгода — тоже не работает. Рынок 
остыл.