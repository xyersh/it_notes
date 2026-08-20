```go
package tokenbucket

import (
	"context"
	"errors"
	"math"
	"sync"
	"time"
)

var (
	ErrInvalidRate     = errors.New("tokenbucket: rate must be positive and finite")
	ErrInvalidCapacity = errors.New("tokenbucket: capacity must be positive")
	ErrExceedsCapacity = errors.New("tokenbucket: requested tokens exceed capacity")
)

// TokenBucket — потокобезопасный ограничитель скорости.
type TokenBucket struct {
	mu       sync.Mutex
	capacity float64 // максимальный размер bucket, он же burst
	tokens   float64 // количество токенов в корзине
	rate     float64 // скорость пополнения, токенов в секунду
	last     time.Time
}

// New создаёт новый TokenBucket.
//
// Например:
//	New(100, 200) означает:
//	- средняя скорость 100 токенов/сек;
//	- максимально можно накопить 200 токенов.
func New(rate float64, capacity int64) (*TokenBucket, error) {
	if rate <= 0 || math.IsNaN(rate) || math.IsInf(rate, 0) {
		return nil, ErrInvalidRate
	}

	if capacity <= 0 {
		return nil, ErrInvalidCapacity
	}
    
	c := float64(capacity)

	return &TokenBucket{
		capacity: c,
		tokens:   c,
		rate:     rate,
		last:     time.Now(),
	}, nil
}

// MustNew — то же самое, что New, но паникует при некорректных параметрах.
func MustNew(rate float64, capacity int64) *TokenBucket {
	b, err := New(rate, capacity)
	if err != nil {
		panic(err)
	}
	return b
}

// refill обновляет количество токенов с учётом прошедшего времени.
// Вызывается только под заблокированным mutex.
func (b *TokenBucket) refill(now time.Time) {
	if !now.After(b.last) {
		return
	}

	elapsed := now.Sub(b.last).Seconds()

	b.tokens = math.Min(b.capacity, b.tokens+elapsed*b.rate)
	b.last = now
}

// Allow проверяет, можно ли выполнить один запрос.
func (b *TokenBucket) Allow() bool {
	return b.Take(1)
}

// Take пытается списать n токенов.
// Возвращает true, если токены были доступны, иначе false.
func (b *TokenBucket) Take(n int64) bool {
	if n <= 0 {
		return true
	}

	need := float64(n)

	// Быстрая проверка без mutex.
	if need > b.capacity {
		return false
	}

	b.mu.Lock()
	defer b.mu.Unlock()

	now := time.Now()
	b.refill(now)

	if b.tokens < need {
		return false
	}

	b.tokens -= need
	return true
}

// Wait блокируется до тех пор, пока не будет доступно n токенов,
// либо пока не отменится context.
func (b *TokenBucket) Wait(ctx context.Context, n int64) error {
	if n <= 0 {
		return nil
	}

	need := float64(n)

	if need > b.capacity {
		return ErrExceedsCapacity
	}

	const minWait = time.Microsecond

	for {
		if err := ctx.Err(); err != nil {
			return err
		}

		b.mu.Lock()

		now := time.Now()
		b.refill(now)

		if b.tokens >= need {
			b.tokens -= need
			b.mu.Unlock()
			return nil
		}

		waitDur := time.Duration((need - b.tokens) / b.rate * float64(time.Second))
		b.mu.Unlock()

		if waitDur < minWait {
			waitDur = minWait
		}

		// Если у context есть deadline, не ждём дольше него.
		if deadline, ok := ctx.Deadline(); ok {
			if d := time.Until(deadline); d < waitDur {
				waitDur = d
			}
		}

		if waitDur <= 0 {
			// Либо deadline уже прошёл, либо ждать почти нечего.
			// Следующая итерация проверит ctx.Err() или повторит попытку.
			continue
		}

		timer := time.NewTimer(waitDur)

		select {
		case <-ctx.Done():
			timer.Stop()
			return ctx.Err()
		case <-timer.C:
			// После таймера снова проверяем доступные токены,
			// потому что другие горутины могли их успеть забрать.
		}
	}
}

// Available возвращает приблизительное количество доступных токенов.
// Полезно для метрик и отладки.
func (b *TokenBucket) Available() float64 {
	b.mu.Lock()
	defer b.mu.Unlock()

	now := time.Now()
	b.refill(now)

	return b.tokens
}
```