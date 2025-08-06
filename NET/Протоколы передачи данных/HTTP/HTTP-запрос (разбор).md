
#### Структура HTTP-запроса
1. **Стартовая строка (Request Line)**:
        - Определяет метод запроса, URI и версию HTTP.
2. **Заголовки (Headers)**:
     Содержат метаданные о запросе, такие как тип содержимого, куки, авторизация и. Заголовки делятся на следующие группы:
        - **Общие заголовки (General Headers):** Применимы как к запросам, так и к ответам (`Date`, `Connection`).
		- **Заголовки запроса (Request Headers):** Применимы только к запросам (`User-Agent`, `Accept`).  
		- **Заголовки ответа (Response Headers):** Применимы только к ответам (`Server`, `Content-Type`).
		- **Заголовки сущности (Entity Headers):** Применимы к телу сообщения (`Content-Type`, `Content-Length`).
3. **Тело запроса (Body)**:
        - Содержит данные, передаваемые в запросе (например, данные формы или JSON).

#### Таблица сущностей HTTP-запроса

| **Сущность**                      | **Описание**                                                                                         |
| --------------------------------- | ---------------------------------------------------------------------------------------------------- |
| **Стартовая строка**              |                                                                                                      |
| Метод (Method)                    | HTTP-метод (GET, POST, PUT, DELETE и т.д.). Определяет действие запроса.                             |
| URI (Uniform Resource Identifier) | Путь к ресурсу на сервере (например, `/users/123`).                                                  |
| Версия HTTP (HTTP Version)        | Версия протокола HTTP (например, `HTTP/1.1`).                                                        |
| **Заголовки**                     |                                                                                                      |
| Host                              | Имя хоста и порт сервера (например, `example.com:8080`). (обязательный заголовок для HTTP/1.1)       |
| User-Agent                        | Информация о клиенте (браузере или программе), отправившем запрос.                                   |
| Accept-Encoding                   | предпочитаемые методы сжатия `Accept-Encoding: gzip, deflate, br`                                    |
| Accept                            | Типы содержимого, которые клиент ожидает для обработки (например, `application/json`).               |
| Content-Type                      | Тип содержимого тела запроса (например, `application/json` или `application/x-www-form-urlencoded`). |
| Content-Length                    | Длина тела запроса в байтах.                                                                         |
| Authorization                     | Данные для авторизации (например, токен или логин/пароль).                                           |
| Cookie                            | Куки, отправленные клиентом.                                                                         |
| Connection                        | управление соединением `keep-alive` , `close`                                                        |
| Referer                           | URL страницы, с которой был сделан запрос.                                                           |
| **Тело запроса**                  |                                                                                                      |
| Данные формы                      | Данные, отправленные через форму (например, `username=admin&password=123`).                          |
| JSON                              | Данные в формате JSON (например, `{"name": "John", "age": 30}`).                                     |
| Файлы                             | Данные файлов, отправленные через multipart/form-data.                                               |

```http
POST /users HTTP/1.1
Host: example.com
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.124 Safari/537.36
Accept: application/json
Content-Type: application/json
Content-Length: 45
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
Cookie: session=abc123; theme=dark

{
    "name": "John Doe",
    "email": "john.doe@example.com"
}
```

#### Разбор примера
1. **Стартовая строка**:  
    - Метод: `POST`
    - URI: `/users`
    - Версия HTTP: `HTTP/1.1`
        
2. **Заголовки**:
    - `Host: example.com` — указывает сервер, на который отправлен запрос.
    - `User-Agent` — информация о клиенте (браузере).
    - `Accept: application/json` — клиент ожидает ответ в формате JSON.
    - `Content-Type: application/json` — тело запроса содержит JSON.
    - `Content-Length: 45` — длина тела запроса (45 байт).
    - `Authorization: Bearer ...` — токен для авторизации.
    - `Cookie: session=abc123; theme=dark` — куки, отправленные клиентом.
        
3. **Тело запроса**:
    - JSON-данные:
```json
{
    "name": "John Doe",
    "email": "john.doe@example.com"
}
```


#### Пример HTTP-запроса с файлом (multipart/form-data)
```http
POST /upload HTTP/1.1
Host: example.com
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.124 Safari/537.36
Content-Type: multipart/form-data; boundary=----WebKitFormBoundary7MA4YWxkTrZu0gW
Content-Length: 345

------WebKitFormBoundary7MA4YWxkTrZu0gW
Content-Disposition: form-data; name="username"

John Doe
------WebKitFormBoundary7MA4YWxkTrZu0gW
Content-Disposition: form-data; name="file"; filename="example.txt"
Content-Type: text/plain

(Содержимое файла example.txt)
------WebKitFormBoundary7MA4YWxkTrZu0gW--
```

### Разбор примера с файлом

1. **Стартовая строка**:
    - Метод: `POST`
    - URI: `/upload`
    - Версия HTTP: `HTTP/1.1`
        
2. **Заголовки**:
    - `Host: example.com` — указывает сервер.
    - `User-Agent` — информация о клиенте.
    - `Content-Type: multipart/form-data; boundary=...` — тип содержимого и разделитель для частей тела запроса.
    - `Content-Length: 345` — длина тела запроса.
        
3. **Тело запроса**:
    - Часть 1: Текстовое поле `username` со значением `John Doe`.
    - Часть 2: Файл `example.txt` с содержимым.


#### boundary
**boundary**- это уникальная строка, которая служит разделителем между частями тела запроса. Она указывается в заголовке `Content-Type` запроса, например:
```http
Content-Type: multipart/form-data; boundary=----WebKitFormBoundary7MA4YWxkTrZu0gW
```
Эта строка должна быть уникальной и не встречаться в самих данных запроса.

##### Как работает boundary?
**Начало boundary**:
- Каждая часть тела запроса начинается с boundary, перед которым стоят два дефиса (`--`).
- Пример:
```http
------WebKitFormBoundary7MA4YWxkTrZu0gW
```
**Заголовки части**:
- После boundary идут заголовки, описывающие текущую часть (например, имя поля или файла).
- Пример:
```http
Content-Disposition: form-data; name="username"
```
**Тело части**:
- После заголовков следует пустая строка, а затем данные части (текстовое значение или содержимое файла).    
- Пример:
```http
John Doe
```
**Конец boundary**:
- После последней части boundary завершается двумя дефисами в конце (`--`).
- Пример:
```http
------WebKitFormBoundary7MA4YWxkTrZu0gW--
```

#### Пример разбора запроса с boundary
Рассмотрим пример запроса:

```http
POST /upload HTTP/1.1
Host: example.com
Content-Type: multipart/form-data; boundary=----WebKitFormBoundary7MA4YWxkTrZu0gW
Content-Length: 345

------WebKitFormBoundary7MA4YWxkTrZu0gW
Content-Disposition: form-data; name="username"

John Doe
------WebKitFormBoundary7MA4YWxkTrZu0gW
Content-Disposition: form-data; name="file"; filename="example.txt"
Content-Type: text/plain

(Содержимое файла example.txt)
------WebKitFormBoundary7MA4YWxkTrZu0gW--
```

#### Разбор:
1. **Первая часть**: 
    - Boundary: `------WebKitFormBoundary7MA4YWxkTrZu0gW`
    - Заголовок: `Content-Disposition: form-data; name="username"`
    - Тело: `John Doe`
    Эта часть передает текстовое поле с именем `username` и значением `John Doe`.

2. **Вторая часть**:    
    - Boundary: `------WebKitFormBoundary7MA4YWxkTrZu0gW`
    - Заголовок: `Content-Disposition: form-data; name="file"; filename="example.txt"`
    - Заголовок: `Content-Type: text/plain`
    - Тело: `(Содержимое файла example.txt)`
    Эта часть передает файл с именем `example.txt` и его содержимым.
    
3. **Завершение**:
    - Boundary: `------WebKitFormBoundary7MA4YWxkTrZu0gW--`
    Два дефиса в конце указывают на завершение тела запроса.