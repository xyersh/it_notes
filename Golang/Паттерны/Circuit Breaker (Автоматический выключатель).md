### Что такое Circuit Breaker?

это критически важный шаблон проектирования, используемый в распределенных системах для повышения **отказоустойчивости** и **стабильности**. Он предотвращает постоянные сбои в системе, вызванные многократными неудачными вызовами к внешнему сервису или удаленной операции.

### Основные термины

При обсуждении и реализации паттерна "Circuit Breaker" используются следующие термины:
- **Circuit Breaker (Автоматический выключатель):** Сам объект или компонент, который отслеживает количество успешных и неудачных вызовов к защищаемой операции. Он управляет состоянием соединения.
- **Protected Operation (Защищаемая операция):** Операция, которая потенциально может завершиться неудачей из-за проблем с сетью, таймаутов, или недоступности внешнего сервиса (например, вызов к API или базе данных).
- **Failure Threshold (Порог отказа):** Максимально допустимое количество (или процент) неудачных вызовов, прежде чем Circuit Breaker перейдет в состояние **Open**.
- **Timeout (Таймаут):** Период, в течение которого Circuit Breaker ожидает перехода из состояния **Open** в **Half-Open**.
### **Сценарии использования**

Circuit Breaker следует применять в ситуациях, когда ваше приложение взаимодействует с внешними, потенциально нестабильными ресурсами:
- **Вызовы микросервисов:** Взаимодействие между сервисами в архитектуре микросервисов по HTTP/gRPC.
- **Взаимодействие с внешними API:** Вызовы сторонних сервисов (платежные шлюзы, почтовые сервисы, облачные хранилища).
- **Доступ к базе данных:** Запросы к удаленным базам данных, которые могут быть перегружены.
- **Любые удаленные или сетевые операции, которые могут зависнуть или завершиться ошибкой.**


###  Какие проблемы решает?

В распределенных системах (например, микросервисах) один сервис часто зависит от другого. Если один из сервисов (назовем его "Сервис Б") перестает отвечать, то "Сервис А", который к нему обращается, может столкнуться со следующими проблемами:

- **Быстрый отказ (Fail Fast):** Немедленно возвращать ошибку, если внешний сервис недоступен, вместо того чтобы заставлять вызывающий код ждать долгого таймаута.
- **Защита внешнего сервиса:** Предотвращать постоянное "наводнение" неисправного или перегруженного сервиса новыми запросами, давая ему возможность восстановиться. Это предотвращает **каскадные сбои**.

**Circuit Breaker решает эти проблемы,** позволяя системе быстро реагировать на сбои и изолировать их, не допуская коллапса всей архитектуры.

###  Состояния Circuit Breaker

Паттерн имеет три основных состояния:

1. **Closed (Замкнуто):** Нормальное состояние. Все запросы свободно проходят к удаленному сервису. "Предохранитель" подсчитывает количество неудачных вызовов. Если число ошибок превышает заданный порог за определенное время, он переходит в состояние `Open`.
2. **Open (Разомкнуто):** "Предохранитель" разорвал цепь. Все запросы к сервису немедленно блокируются и возвращается ошибка, без реальной попытки вызова. Через определенный таймаут "предохранитель" переходит в состояние `Half-Open`.
3. **Half-Open (Полуоткрыто):** Проверочное состояние. "Предохранитель" позволяет пройти одному или нескольким тестовым запросам к сервису.
    - **Если запрос успешен:** "Предохранитель" считает, что сервис восстановился, и переходит в состояние `Closed`.
    - **Если запрос неудачен:** "Предохранитель" возвращается в состояние `Open` и снова запускает таймер ожидания.

###  Нюансы использования

- **Настройка порогов (Thresholds):** Самая сложная часть — правильно настроить параметры:
    - **Порог ошибок:** Сколько неудачных вызовов должно произойти, чтобы "разомкнуть" цепь? Слишком низкое значение может привести к ложным срабатываниям, а слишком высокое — не защитит от реальных проблем.
    - **Таймаут в состоянии Open:** Как долго ждать перед переходом в `Half-Open`? Слишком короткий таймаут не даст сервису времени на восстановление, а слишком долгий увеличит время простоя.
    - **Количество тестовых запросов:** Сколько успешных вызовов в `Half-Open` состоянии считать достаточным для полного восстановления?
- **Логирование и мониторинг:** Критически важно отслеживать смену состояний "предохранителя". Это дает понимание о здоровье зависимых сервисов.
- **Fallback-логика:** Что делать, когда цепь разомкнута? Просто вернуть ошибку — это уже хорошо. Но еще лучше предоставить альтернативный результат (fallback): например, вернуть кэшированные данные или значение по умолчанию.
- **Один предохранитель на экземпляр или на тип сервиса?** Если у вас несколько экземпляров одного и того же сервиса, стоит подумать: использовать один общий Circuit Breaker для всех или отдельный для каждого экземпляра.



###  Примеры на Golang

В Go нет встроенного Circuit Breaker, но есть множество отличных библиотек. Рассмотрим популярную `sony/gobreaker`.
Установка
```bash
go get github.com/sony/gobreaker
```

### ПРИМЕРЫ

В Go можно реализовать паттерн "Circuit Breaker" с использованием стандартных пакетов `time` и `sync`. Для данной реализации будем использовать интерфейс `Circuit` для определения поведения и `circuitBreaker` как его конкретную реализацию.

```go
package main

import (
	"errors"
	"fmt"
	"sync"
	"time"
)

// State определяет текущее состояние Circuit Breaker.
type State int

const (
	Closed State = iota // Вызовы разрешены
	Open                // Вызовы блокированы
	HalfOpen            // Разрешены тестовые вызовы
)

// Circuit определяет контракт паттерна Circuit Breaker
type Circuit interface {
	Execute(request func() error) error
}

// circuitBreaker является конкретной реализацией Circuit Breaker.
type circuitBreaker struct {
	state             State         // Текущее состояние (Closed, Open, HalfOpen)
	failureCount      int           // Текущее количество последовательных сбоев
	failureThreshold  int           // Порог для перехода в Open
	timeout           time.Duration // Время, после которого Open переходит в HalfOpen
	lastFailureTime   time.Time     // Время последнего перехода в Open
	halfOpenSuccesses int           // Количество успешных вызовов в HalfOpen
	maxHalfOpenTests  int           // Необходимое число успехов для перехода в Closed
	mu                sync.Mutex    // Мьютекс для безопасного изменения состояния
}

// NewCircuitBreaker создает новый экземпляр Circuit Breaker.
func NewCircuitBreaker(threshold int, timeout time.Duration, maxTests int) Circuit {
	return &circuitBreaker{
		state:             Closed,
		failureThreshold:  threshold,
		timeout:           timeout,
		maxHalfOpenTests:  maxTests,
		halfOpenSuccesses: 0,
	}
}

// Execute выполняет защищаемую операцию с логикой Circuit Breaker.
func (cb *circuitBreaker) Execute(request func() error) error {
	cb.mu.Lock()
	defer cb.mu.Unlock()

	// 1. Проверка состояния Open
	if cb.state == Open {
		// Проверяем, истекло ли время таймаута
		if time.Since(cb.lastFailureTime) > cb.timeout {
			cb.toHalfOpen() // Переход в Half-Open, если таймаут истек
		} else {
			// Быстрый отказ: если в Open и таймаут не истек
			return errors.New("circuit breaker is open: external service is unavailable")
		}
	}

	// 2. Выполнение операции
	err := request()

	// 3. Обработка результата
	if err != nil {
		cb.handleFailure()
		return err
	}

	// Успешное выполнение
	cb.handleSuccess()
	return nil
}

// handleFailure обрабатывает неудачный вызов и управляет переходами Closed -> Open и Half-Open -> Open.
func (cb *circuitBreaker) handleFailure() {
	if cb.state == HalfOpen {
		cb.toOpen() // Неудача в Half-Open -> немедленный возврат в Open
		return
	}

	// В состоянии Closed: инкремент счетчика
	cb.failureCount++
	if cb.failureCount >= cb.failureThreshold {
		cb.toOpen() // Превышен порог -> переход в Open
	}
}

// handleSuccess обрабатывает успешный вызов и управляет переходом Half-Open -> Closed.
func (cb *circuitBreaker) handleSuccess() {
	if cb.state == HalfOpen {
		cb.halfOpenSuccesses++
		if cb.halfOpenSuccesses >= cb.maxHalfOpenTests {
			cb.toClosed() // Достаточно успехов -> переход в Closed
		}
	} else if cb.state == Closed {
		cb.failureCount = 0 // Сброс счетчика успехов в Closed
	}
}

// toClosed переводит состояние в Closed и сбрасывает счетчики.
func (cb *circuitBreaker) toClosed() {
	cb.state = Closed
	cb.failureCount = 0
	cb.halfOpenSuccesses = 0
	fmt.Println("Circuit Breaker: CLOSED (Service recovered)")
}

// toOpen переводит состояние в Open, фиксирует время сбоя и сбрасывает счетчики.
func (cb *circuitBreaker) toOpen() {
	cb.state = Open
	cb.lastFailureTime = time.Now()
	cb.failureCount = 0
	cb.halfOpenSuccesses = 0
	fmt.Printf("Circuit Breaker: OPEN (Failure threshold reached, trip time: %s)\n", cb.lastFailureTime.Format(time.RFC3339))
}

// toHalfOpen переводит состояние в Half-Open для тестовых вызовов.
func (cb *circuitBreaker) toHalfOpen() {
	cb.state = HalfOpen
	cb.halfOpenSuccesses = 0
	fmt.Println("Circuit Breaker: HALF-OPEN (Testing service recovery)")
}

func main() {
	// Инициализация Circuit Breaker: 3 сбоя, 5 секунд таймаут, 2 успешных теста в Half-Open
	cb := NewCircuitBreaker(3, 5*time.Second, 2)
	requestCounter := 0

	// Эмуляция защищаемой операции
	mockServiceCall := func() error {
		requestCounter++
		// Эмуляция сбоев: сбои с 1 по 4, успех на 5, сбои с 6 по 8, успех на 9 и т.д.
		if requestCounter%5 != 0 {
			// Имитация сбоя сети/таймаута
			return errors.New("mock service call failed")
		}
		// Имитация успешного ответа
		return nil
	}

	fmt.Println("--- START SIMULATION ---")

	// 1. Сбои, ведущие к Open
	for i := 0; i < 5; i++ {
		fmt.Printf("Attempt %d: ", i+1)
		err := cb.Execute(mockServiceCall)
		if err != nil {
			fmt.Println("Error:", err)
		} else {
			fmt.Println("Success!")
		}
		time.Sleep(500 * time.Millisecond)
	}
	fmt.Println("--- Open state activated ---")
	time.Sleep(1 * time.Second) // Задержка

	// 2. Вызовы в состоянии Open (быстрый отказ)
	for i := 0; i < 3; i++ {
		fmt.Printf("Attempt %d (Open): ", i+6)
		err := cb.Execute(mockServiceCall)
		if err != nil {
			fmt.Println("Error:", err)
		} else {
			fmt.Println("Success!")
		}
		time.Sleep(500 * time.Millisecond)
	}
	fmt.Println("--- Waiting for Timeout ---")
	time.Sleep(4 * time.Second) // Ждем истечения таймаута (Open + 4с + 1с = 5с)

	// 3. Переход в Half-Open и тестирование
	for i := 0; i < 4; i++ {
		fmt.Printf("Attempt %d (Test): ", i+9)
		err := cb.Execute(mockServiceCall)
		if err != nil {
			fmt.Println("Error:", err)
		} else {
			fmt.Println("Success!")
		}
		time.Sleep(500 * time.Millisecond)
	}
	fmt.Println("--- Simulation End ---")
}
```


#### Пример : Простой вызов

Этот пример имитирует вызов к внешнему сервису, который иногда возвращает ошибку.

```go
package main

import (
	"errors"
	"fmt"
	"log"
	"time"

	"github.com/sony/gobreaker"
)

var cb *gobreaker.CircuitBreaker

func init() {
	// Настройки нашего "предохранителя"
	var settings gobreaker.Settings
	settings.Name = "HTTP-GET-Service"
	// Порог для открытия цепи - 5 последовательных ошибок
	settings.MaxRequests = 1
	// Таймаут перед переходом в Half-Open
	settings.Timeout = 10 * time.Second
	// Порог для открытия цепи
	settings.ReadyToTrip = func(counts gobreaker.Counts) bool {
		return counts.ConsecutiveFailures > 5
	}
	// Логирование смены состояний
	settings.OnStateChange = func(name string, from gobreaker.State, to gobreaker.State) {
		log.Printf("CircuitBreaker '%s' changed state from '%s' to '%s'\n", name, from, to)
	}

	cb = gobreaker.NewCircuitBreaker(settings)
}

// Имитация нестабильного внешнего сервиса
var requestCounter = 0

func getUnstableData() (string, error) {
	requestCounter++
	// Первые 6 вызовов будут неудачными
	if requestCounter <= 6 {
		return "", errors.New("service is currently unavailable")
	}
	// После этого сервис "восстанавливается"
	return "Hello from remote service!", nil
}

func main() {
	for i := 0; i < 15; i++ {
		body, err := cb.Execute(func() (interface{}, error) {
			// Оборачиваем наш вызов в Execute
			return getUnstableData()
		})

		if err != nil {
			log.Printf("Попытка %d: Ошибка: %v", i+1, err)
		} else {
			log.Printf("Попытка %d: Успех: %s", i+1, body.(string))
		}
		time.Sleep(1 * time.Second)
	}
}
```

**Что происходит в коде:**

1. Мы создаем `CircuitBreaker` с настройками: он перейдет в состояние `Open` после 5 последовательных ошибок.
2. В цикле мы 15 раз пытаемся выполнить функцию `getUnstableData` через `cb.Execute`.
3. Первые 6 вызовов `getUnstableData` возвращают ошибку. После 5-й ошибки "предохранитель" сработает и перейдет в `Open`.
4. 6-й вызов вернет ошибку немедленно, без реального вызова функции (это и есть работа Circuit Breaker).
5. Далее "предохранитель" будет в состоянии `Open` в течение 10 секунд. Все запросы в этот период будут мгновенно отклонены.
6. По истечении таймаута он перейдет в `Half-Open`. Следующий вызов будет тестовым. Поскольку `getUnstableData` уже "восстановился", вызов будет успешным.
7. "Предохранитель" вернется в состояние `Closed`, и последующие запросы снова будут проходить нормально.