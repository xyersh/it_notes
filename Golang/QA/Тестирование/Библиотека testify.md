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

### Функции уровня пакета
| Функция                                                              | Описание                                    |
| -------------------------------------------------------------------- | ------------------------------------------- |
| assert.New(*testing.T)  *assert.Assertions                           | Создание переменной типа *assert.Assertions |
| assert.NotEqual(t, expected, actual, "they should not be equal")     | Проверка на неравенство                     |
| assert.Equal(t *testing.T, expected, actual, "they should be equal") | Проверка на равенcтво                       |
| assert.NoError(t *testing.T, err)                                    | Проверка на отсутствие ошибки               |
| assert.Nil(t *testing.T, object)                                     | Проверка на nil                             |
| assert.NotNil(t *testing.T, object)                                  | Проверка на not nil                         |
|                                                                      |                                             |


### Методы assert.Assertions

| Методы                                                | Описание |
| ----------------------------------------------------- | -------- |
| assert.Equal(123, 123, "they should be equal")        |          |
| assert.NotEqual(123, 456, "they should not be equal") |          |
| assert.Nil(object)                                    |          |
| assert.NotNil(object)                                 |          |
|                                                       |          |


## `assert/require`

Пакет `require` имеет **идентичный API** с `assert`, но при неудачной проверке **немедленно прерывает** тест через `t.FailNow()`

