
### База

#### Что это такое?

Протокол **TCP (Transmission Control Protocol)** — это один из ключевых протоколов транспортного уровня в стеке **TCP/IP**. Его основная задача — обеспечить **надёжную, ориентированную на соединение** передачу данных между двумя устройствами. В отличие от **UDP**, который просто отправляет пакеты, TCP гарантирует, что все данные дойдут до адресата в правильном порядке и без потерь. Это достигается за счёт нескольких механизмов:
- **Установление соединения (Three-way Handshake):** Перед передачей данных отправитель и получатель обмениваются пакетами, чтобы убедиться, что они готовы к общению.
- **Подтверждение получения (ACK):** Получатель отправляет подтверждение о получении каждого пакета. Если отправитель не получает ACK, он повторно отправляет данные.
- **Нумерация пакетов:** Каждый пакет имеет порядковый номер, что позволяет получателю собрать данные в правильной последовательности.
- **Контроль потока:** Механизм, который не позволяет одному из участников перегружать другого, отправляя данные слишком быстро.


### Базовый API в Golang

**Общие функции**
- `(Conn)Write([]byte)(int, error)` - записать в TCP коннект
- `(Conn)Read([]byte)(int, error)` - читать из TCP коннекта
- `(Conn)Close()` - закрыть соединение

**Серверная часть:**
- `net.Listen(network, address string) (Listener , error)`- получить листенер
- `(Listener)Accept() (Conn, error)` -  ожидать коннекта от клиентов, блокирующая функция.
- `(Listener)Close()` - закрыть листенер



**Клиентская часть**
- `net.Dial(network, address string)(Conn, error)` - получить TCP -соединение с сервером. 



### Пример


#### Сервер
принимает сообщения  от клиетов, проводит широковещательную отправку сообдений по всем остальным клиентам 
```go
package main

  

import (
    "fmt"
    "net"
    "sync"
)

// Канал для трансляции сообщений.
var messages = make(chan string)

// Список активных клиентов. Используем sync.Map для безопасной работы с конкурентным доступом.
var clients = sync.Map{}

func main() {

    listener, err := net.Listen("tcp", "localhost:8866")
    if err != nil {
        fmt.Println("Ошибка при запуске сервера:", err)
        return
    }
    defer listener.Close()
    fmt.Println("Чат-сервер запущен...")

    // Запускаем горутину, которая будет слушать канал `messages` и транслировать сообщения.
    go broadcaster()
  
    for {
        conn, err := listener.Accept()
        if err != nil {
            fmt.Println("Ошибка при приёме соединения:", err)
            continue
        }
        go handleConnection(conn)
    }
}

  

// `broadcaster` — горутина, которая транслирует сообщения всем клиентам.
func broadcaster() {
    for {
        msg := <-messages // Блокируется, пока не получит сообщение из канала.
        fmt.Println("Трансляция:", msg)

        // Перебираем всех клиентов и отправляем им сообщение.
        clients.Range(func(key, value interface{}) bool {
            clientConn := value.(net.Conn)
            _, err := clientConn.Write([]byte(msg + "\n"))
            if err != nil {
                // Если ошибка, удаляем клиента.
                clients.Delete(key)
            }
            return true // Продолжаем итерацию.
        })
    }
}


func handleConnection(conn net.Conn) {
    defer conn.Close()


    // Добавляем клиента в список.
    clientAddr := conn.RemoteAddr().String()
    clients.Store(clientAddr, conn)
    fmt.Printf("Новый клиент: %s\n", clientAddr)
  
    // Отправляем сообщение о новом клиенте.
    messages <- fmt.Sprintf("К нам присоединился: %s", clientAddr)

    buffer := make([]byte, 1024)
    for {
        n, err := conn.Read(buffer)
        if err != nil {
            // Ошибка чтения означает, что клиент отключился.
            fmt.Printf("Клиент %s отключился.\n", clientAddr)
            clients.Delete(clientAddr)
            messages <- fmt.Sprintf("Клиент %s покинул чат.", clientAddr)
            return // Завершаем горутину.
        }

        receivedData := string(buffer[:n-1]) // -1 для удаления символа новой строки.
        messages <- fmt.Sprintf("[%s]: %s", clientAddr, receivedData)
    }
}
```