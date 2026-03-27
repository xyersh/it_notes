URL: [github.com/stretchr/testify](https://github.com/stretchr/testify)

## Введение и установка
`testify` — это популярный инструментарий для написания тестов в Go, который предоставляет:

- Удобные функции для проверок (assertions)
- Механизм мокирования зависимостей
- Поддержку тестовых наборов (suites)
- Красивые и информативные сообщения об ошибках

**Установка:** 
```bash
go get github.com/stretchr/testify
```


## Пакет `assert`
Пакет `assert` предоставляет функции для проверок, которые **не прерывают** выполнение теста при неудаче — тест продолжается, а все неудачные проверки фиксируются
import "github.com/stretchr/testify/assert"

### Функции уровня пакета
#### Базовые сравнения
| Функция                                                                               | Описание                                    |
| ------------------------------------------------------------------------------------- | ------------------------------------------- |
| `assert.New(*testing.T)  *assert.Assertions`                                          | Создание переменной типа *assert.Assertions |
| `assert.NotEqual(t *testing.T, expected any, actual any, "they should not be equal")` | Ожидает на неравенство                      |
| `assert.Equal(t *testing.T, expected any, actual any, "they should be equal")`        | Ожидает  равенcтво                          |
| `assert.Nil(t *testing.T, object any, "the value should be nil")`                     | Ожидает nil                                 |
| `assert.NotNil(t *testing.T, object any, "the value shouldnt be nil")`                | Ожидает not nil                             |
| `assert.True(t *testing.T, value bool, "the value should be true")`                   | Ожидает true                                |
| `assert.False(t *testing.T, value bool, "the value should be false")`                 | Ожидает false                               |


#### Работа с коллекциями

| Функция                                                        | Описание                                                        |
| -------------------------------------------------------------- | --------------------------------------------------------------- |
| `assert.Len(t *testing.T, object, length)`                     | Проверка длины среза/карты/строки                               |
| `assert.Contains(t *testing.T, container, element)`            | Проверка наличия элемента                                       |
| `assert.NotContains(t *testing.T, container, element)`         | Проверка отсутствия элемента                                    |
| `assert.ElementsMatch(t  *testing.T, listA ant, listB any)`    | Ожидает равенство элементов списков без учета порядка элементов |
| `assert.Subset(t *testing.T, list any, subset any, "subset ")` | Ожидает, что `subset` является подмножеством `list`             |


#### Работа с ошибками
| Функция                                                         | Описание                     |
| --------------------------------------------------------------- | ---------------------------- |
| `assert.Error(t *testing.T, err error, "err shouldn't be nil")` | Ожидает наличие ошибки       |
| `assert.NoError(t *testing.T, err error, "err shouldn nil")`    | Ожидает  отсутствие ошибки   |
| `assert.EqualError(t *testing.T, err, expected)`                       | Проверка текста ошибки       |
| `assert.ErrorIs(t *testing.T, err, target)`                            | Проверка через `errors.Is()` |
| `assert.ErrorAs(t  *testing.T, err, &target)`                          | Проверка через `errors.As()` |
|                                                                 |                              |


####  Числовые сравнения
| Функция                                                     | Описание                  | Пример                                   |
| ----------------------------------------------------------- | ------------------------- | ---------------------------------------- |
| `assert.Greater(t *testing.T, e1, e2)`                      | e1 > e2                   | `assert.Greater(t, 10, 5)`               |
| `assert.Less(t *testing.T, e1, e2)`                         | e1 < e2                   | `assert.Less(t, price, 1000)`            |
| `assert.InDelta(t *testing.T, expected, actual, delta)`     | Разница в пределах delta  | `assert.InDelta(t, 3.14, pi, 0.01)`      |
| `assert.InEpsilon(t *testing.T, expected, actual, epsilon)` | Относительная погрешность | `assert.InEpsilon(t, 100, result, 0.05)` |
|                                                             |                           |                                          |


#### Проверка типов и интерфейсов

| Функция                               | Описание                       | Пример                                          |
| ------------------------------------- | ------------------------------ | ----------------------------------------------- |
| `IsType(t, expectedType, object)`     | Проверка типа                  | `assert.IsType(t, &User{}, obj)`                |
| `Implements(t, interfaceObj, object)` | Проверка реализации интерфейса | `assert.Implements(t, (*io.Reader)(nil), file)` |

#### Асинхронные и временные проверки
| Функция                                      | Описание                                      | Пример                                                                                             |
| -------------------------------------------- | --------------------------------------------- | -------------------------------------------------------------------------------------------------- |
| `Eventually(t, condition, waitFor, tick)`    | Условие должно стать true за время waitFor    | `assert.Eventually(t, func() bool { return db.Connected() }, 5*time.Second, 100*time.Millisecond)` |
| `Never(t, condition, waitFor, tick)`         | Условие не должно стать true за время waitFor | `assert.Never(t, func() bool { return job.Failed() }, 2*time.Second, 50*time.Millisecond)`         |
| `WithinDuration(t, expected, actual, delta)` | Временные метки в пределах delta              | `assert.WithinDuration(t, start, end, time.Minute)`                                                |

#### Проверка паник
| Функция                            | Описание                          | Пример                                                                |
| ---------------------------------- | --------------------------------- | --------------------------------------------------------------------- |
| `Panics(t, fn)`                    | Функция должна вызвать панику     | `assert.Panics(t, func() { panic("boom") })`                          |
| `NotPanics(t, fn)`                 | Функция не должна вызывать панику | `assert.NotPanics(t, func() { safeCall() })`                          |
| `PanicsWithValue(t, expected, fn)` | Паника с конкретным значением     | `assert.PanicsWithValue(t, "expected", func() { panic("expected") })` |


### Методы assert.Assertions

| Методы                                                | Описание                     |
| ----------------------------------------------------- | ---------------------------- |
| assert.Equal(123, 123, "they should be equal")        |                              |
| assert.NotEqual(123, 456, "they should not be equal") |                              |
| assert.Nil(object)                                    |                              |
| assert.NotNil(object)                                 |                              |


## Пакет require

Пакет `require` имеет **идентичный API** с `assert`, но при неудачной проверке **немедленно прерывает** тест через `t.FailNow()`
import "github.com/stretchr/testify/require"


## Пакет mock
Пакет `mock` позволяет создавать моки для интерфейсов и проверять, что методы вызываются с ожидаемыми аргументами
import "github.com/stretchr/testify/mock"

### Схема компонентов 
![[Drawing 2026-03-18 17.06.37.excalidraw.png]]

### Жизненный цикл мока:
1. Создаём мок → 
2. Настраиваем ожидания (On) → 
3. Вызываем тестируемый код → 
4. Проверяем результат → 
5. Верифицируем ожидания (AssertExpectations)


### Пошаговый процесс создания мока
#### Шаг 1: Определите интерфейс зависимости
```go
// repository/user_repository.go
package repository

type UserRepository interface {
    FindByID(id int) (*User, error)
    Save(user *User) error
    Delete(id int) error
}
```

#### Шаг 2: Создайте структуру мока с встраиванием `mock.Mock`
```go
// mocks/user_repository_mock.go
package mocks

import (
    "github.com/stretchr/testify/mock"
    "your-project/repository"
)

type MockUserRepository struct {
    mock.Mock  // ← Ключевой момент: встраиваем базовый Mock
}
```

#### Шаг 3: Реализуйте методы интерфейса
```go
// Для метода с одним возвращаемым значением и ошибкой:
func (m *MockUserRepository) Save(user *repository.User) error {
    // 1. Фиксируем вызов с аргументами
    args := m.Called(user)
    
    // 2. Возвращаем настроенное значение из первого индекса
    //    Если возвращаемых значений несколько — используем разные индексы
    return args.Error(0)
}

// Для метода с несколькими возвращаемыми значениями:
func (m *MockUserRepository) FindByID(id int) (*repository.User, error) {
    args := m.Called(id)
    
    // Get(0) возвращает interface{}, делаем type assertion
    // Для удобства есть типизированные методы: Int(), String(), Error() и т.д.
    return args.Get(0).(*repository.User), args.Error(1)
}

// Для метода без возвращаемых значений:
func (m *MockUserRepository) Delete(id int) error {
    args := m.Called(id)
    return args.Error(0)
}
```


#### Шаг 4: Настройте ожидания в тесте
```go
func TestUserService_Save(t *testing.T) {
    // Arrange
    mockRepo := new(mocks.MockUserRepository)
    service := service.NewUserService(mockRepo)
    
    testUser := &repository.User{ID: 42, Name: "Alice"}
    
    // Настраиваем: когда вызовут Save с testUser → верни nil
    mockRepo.
        On("Save", testUser).      // Метод + аргументы
        Return(nil).                // Что вернуть
        Once()                      // Сколько раз ожидать (опционально)
    
    // Act
    err := service.Register(testUser)
    
    // Assert
    assert.NoError(t, err)
    
    // Verify: проверяем, что все ожидания выполнены
    mockRepo.AssertExpectations(t)
}
```




### Методы mock.Mock
 **import "github.com/stretchr/testify/mock"**



| Метод                                 | Описание                                                       |
| ------------------------------------- | -------------------------------------------------------------- |
| `mock.On(methodName, args...)`        | Начинает описание ожидания вызова метода                       |
| `Return(values...)`                   | Указывает возвращаемые значения                                |
| `Times(n)` / `Once()` / `Twice()`     | Сколько раз ожидается вызов                                    |
| `Maybe()`                             | Вызов метода опционален                                        |
| `Run(fn)`                             | Выполняет функцию при вызова мока (для модификации аргументов) |
| `WaitUntil(channel)`                  | Блокирует возврат до получения сигнала                         |
| `Called(args...)`                     | Фиксирует вызов и возвращает настроенные значения              |
| `AssertExpectations(t)`               | Проверяет, что все ожидания выполнены                          |
| `AssertCalled(t, method, args...)`    | Проверяет, что метод был вызован                               |
| `AssertNotCalled(t, method, args...)` | Проверяет, что метод не вызывался                              |
