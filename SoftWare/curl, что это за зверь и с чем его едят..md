`curl` (от англ. "Client URL") — это мощный инструмент командной строки для передачи данных с или на сервер с использованием различных протоколов, таких как HTTP, HTTPS, FTP, SFTP и других. Она часто используется для тестирования API, загрузки файлов, отправки данных и других задач, связанных с сетевыми запросами.

#### Основные флаги
- **--help**: Помощь

- **`-X` или `--request`**: Указывает метод HTTP-запроса (GET, POST, PUT, DELETE и т.д.).
```bash
curl -X POST https://example.com/api`
```

- **`-H` или `--header`**: Добавляет заголовок HTTP-запроса.
```bash
curl -H "Content-Type: application/json" https://example.com
```

- **`d` или `--data`**: Отправляет данные в теле запроса (обычно используется с POST).
```bash
curl -d "param1=value1&param2=value2" https://example.com
```

- **`-F` или `--form`**: Отправляет данные в виде формы (multipart/form-data).
```bash
curl -F "file=@file.txt" https://example.com/upload
```

- **`-o` или `--output`**: Сохраняет вывод в файл.
```bash
curl -o output.txt https://example.com
```

- **`-O`**: Сохраняет вывод в файл с именем, соответствующим имени файла на сервере.
```bash
curl -O https://example.com/file.txt
```

- **`-i` или `--include`**: Включает заголовки ответа в вывод.
```bash
curl -i https://example.com
```

- **`-I` или `--head`**: Отправляет HEAD-запрос и выводит только заголовки ответа.
```bash
curl -I https://example.com
```

- **`-L` или `--location`**: Следует за перенаправлениями (редиректами).
```bash
curl -L https://example.com
```

- **`-u` или `--user`**: Указывает имя пользователя и пароль для аутентификации.
```bash
curl -u username:password https://example.com
```

- **`-k` или `--insecure`**:  Игнорирует проверку SSL-сертификата (небезопасно, но полезно для тестирования).  
```bash
curl -k https://example.com
```

- **`-v` или `--verbose`**: Выводит подробную информацию о запросе и ответе.
```bash
curl -v https://example.com
```

- **`-s` или `--silent`**: Подавляет вывод прогресса и ошибок (тихий режим).
```bash
curl -s https://example.com 
```

- **`-A` или `--user-agent`**: Указывает пользовательский агент для запроса.
```bash
curl -A "MyUserAgent" https://example.com
```

- **`-b` или `--cookie`**: Отправляет cookie в запросе.
```bash
curl -b "name=value" https://example.com
```

- **`-c` или `--cookie-jar`**: Сохраняет cookie в файл после выполнения запроса.
```bash
curl -c cookies.txt https://example.com
```

- **`-x` или `--proxy`**: Использует прокси для запроса.
```bash
curl -x http://proxy.example.com:8080 https://example.com
```

- **`-T` или `--upload-file`**: Загружает файл на сервер (используется с FTP или SFTP).
```bash
curl -T file.txt ftp://example.com
```

- **`-C` или `--continue-at`**: Продолжает загрузку файла с определенного места (полезно для докачки).
```bash
curl -C - -O https://example.com/file.zip
```

- **`-m` или `--max-time`**: Устанавливает максимальное время выполнения запроса в секундах.
```bash
curl -m 10 https://example.com
```


### Примеры использования:
**Отправка GET-запроса:**
```bash
curl https://example.com
```

**Отправка POST-запроса с JSON-данными:**
```bash
curl -X POST -H "Content-Type: application/json" -d '{"key":"value"}' https://example.com/api
```

**Загрузка файла:**
```bash
curl -O https://example.com/file.zip
```

**Отправка файла на сервер:**
```bash
curl -F "file=@file.txt" https://example.com/upload
```

**Использование аутентификации:**
```bash
curl -u username:password https://example.com
```

**Сохранение cookie:**
```bash
curl -c cookies.txt https://example.com
```

**Использование cookie:**
```bash
curl -b cookies.txt https://example.com
```

