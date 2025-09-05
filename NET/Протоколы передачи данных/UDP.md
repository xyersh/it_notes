### База

#### Чё это такое?
UDP (User Datagram Protocol) - это **простой** протокол передачи данных в сети. Его главное отличие от TCP - это **отсутствие гарантии доставки**. 🚫📦
- **Быстрый**: UDP не тратит время на установку соединения, подтверждение получения или повторную отправку потерянных пакетов.
- **Без гарантий**: Пакеты (дейтаграммы) могут приходить не по порядку, быть потеряны или дублироваться.

#### Когда использовать UDP?
UDP подходит для приложений, где **скорость важнее надёжности**: 🚀
- **Онлайн-игры**: потеря нескольких пакетов не критична.
- **Видео- и аудио-стриминг**: лучше небольшой сбой в картинке, чем задержка.
- **DNS-запросы**: быстрые и небольшие.






### Пример

#### Сервер
Простой эхо-сервер. Принимает от клиента сообщение и возвращает его же с дополнительной информацией 
```go
package main

import (
    "fmt"
    "net"
    "time"
)

func main() {

    addr, err := net.ResolveUDPAddr("udp", ":8899")
    if err != nil {
        panic(err)
    }

    servConn, err := net.ListenUDP("udp", addr)
    if err != nil {
        panic(err)
    }

    defer servConn.Close()


    fmt.Println("UDP-запущен на порту :8899")
    data := make([]byte, 1024)

    for {
        n, clientAddr, err := servConn.ReadFromUDP(data)
        if err != nil {
            fmt.Printf("ошибка чтения: %v\n", err)
        }
        go handleMessage(n, clientAddr, servConn, data[:n])
    }
}

// обработчик запросов от клиента. Сюда вся логика работы с непосредственными сообщениями
func handleMessage(n int, clientAddr *net.UDPAddr, servConn *net.UDPConn, data []byte) {
    // симуляция долгой обработки
    time.Sleep(1 * time.Second)

    //собираем ответ от эхо-сервера
    message := fmt.Sprintf("[%v; %v]:  %s", clientAddr.IP, clientAddr.Port, string(data))

    //отправка сообщения клиенту
    _, err := servConn.WriteToUDP([]byte(message), clientAddr)
    fmt.Printf("оправлено: %s\n", message)
    if err != nil {
        fmt.Printf("ошибка отправки: %v\n", err)
    }
}
```


#### Клиент
```go
package main

import (
    "bufio"
    "fmt"
    "net"
    "os"
    "strings"
)

func main() {

    //создаем коннект с UDP-сервером
    conn, err := net.Dial("udp", "localhost:8899")
    for err != nil {
        panic(err)
    }
    defer conn.Close()
  
    //обработка ответов от сервера пишем асинхронную - чтобы красиво
    go func(conn net.Conn) {
        data := make([]byte, 1024)
        for {
            n, err := conn.Read(data)
            if err != nil {
                fmt.Printf("ошибка чтения ответа: %v\n", err)
                break
            }
            fmt.Printf(" Ответ от сервера: %s\n", string(data[:n]))
        }
  
    }(conn)
  
    for {
        reader := bufio.NewReader(os.Stdin)
        message, err := reader.ReadString('\n')
        message = strings.Trim(message, "\n")

        if err != nil {
            fmt.Printf("ошибка чтения: %v\n", err)
        }

        //отправляем сообщение на сервер
        _, err = conn.Write([]byte(message))
        if err != nil {
            fmt.Printf("ошибка отправки: %v\n")
        }
    }
}
```