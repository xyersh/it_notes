
Основная идея состоит в том, чтобы не считать сбой окончательным, а дать операции еще один или несколько шансов на успех, предполагая, что причина сбоя может самоустраниться через короткое время.

## Важные моменты, которые нужно учитывать

- **Идемпотентность:** Повторять можно только те операции, многократное выполнение которых приводит к тому же результату, что и однократное (например, запрос на получение данных). Повторение неидемпотентных операций (как "списать $10 со счета") может привести к нежелательным последствиям.
- **Количество попыток:** Необходимо ограничивать максимальное число повторов, чтобы не попасть в бесконечный цикл, если ошибка является постоянной, а не временной.
- **Логирование:** Важно логировать неудачные попытки и окончательные сбои, чтобы иметь возможность анализировать проблемы в системе.
- **Комбинация с другими паттернами:** Паттерн Retry часто используется вместе с паттерном **Circuit Breaker ("Предохранитель")**. Если после нескольких попыток операция все равно завершается неудачей, Circuit Breaker "размыкает цепь" и на некоторое время полностью прекращает попытки вызова сбойного сервиса, чтобы не тратить ресурсы и дать сервису время на восстановление.

```go
type RetryConfig struct {  
    MaxAttempts  int  
    InitialDelay time.Duration  
    MaxDelay     time.Duration  
}  
  
func Retry[T any](  
    ctx context.Context,  
    config RetryConfig,  
    operation func() (T, error),  
) (T, error) {  
    var result T  
    var err error  
    currentDelay := config.InitialDelay  
  
    for attempt := 1; attempt <= config.MaxAttempts; attempt++ {  
       if ctx.Err() != nil {  
          return result, ctx.Err()  
       }  
  
       result, err = operation()  
       if err == nil {  
          return result, nil  
       }  
  
       if attempt == config.MaxAttempts {  
          return result, err  
       }  
  
       jitter := time.Duration(rand.Float64() * float64(currentDelay))  
       currentDelay += jitter  
       if currentDelay > config.MaxDelay {  
          currentDelay = config.MaxDelay  
       }  
  
       select {  
       case <-ctx.Done():  
          return result, ctx.Err()  
       case <-time.After(currentDelay):  
          currentDelay *= 2  
       }  
    }  
  
    return result, err  
}
```