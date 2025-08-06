### Что это такое?
Паттерн **pipeline** (конвейер) в конкурентном программировании представляет собой цепочку последовательно соединенных этапов обработки данных. Каждый этап (stage) в конвейере выполняется одной или несколькими горутинами. Данные проходят через конвейер, обрабатываясь на каждом этапе и передаваясь на следующий. Этот паттерн особенно полезен для обработки больших объемов данных, где каждый шаг обработки может быть выполнен независимо и потенциально параллельно.
![[Pasted image 20250524230546.png]]

**Основные характеристики паттерна pipeline:**

- **Этапы (Stages):** Конвейер состоит из нескольких этапов обработки. Каждый этап выполняет определенную задачу над входящими данными.
- **Каналы (Channels):** Данные передаются между этапами конвейера через каналы Go. Выход одного этапа служит входом для следующего.
- **Параллелизм:** Каждый этап может выполняться параллельно с другими этапами, что позволяет повысить общую производительность обработки данных.
- **Направленность каналов:** Обычно используются направленные каналы (`<-chan` для входных и `chan<-` для выходных) для четкого определения потока данных и предотвращения случайных операций записи или чтения из неправильного конца канала.
- **Закрытие каналов:** Каждый этап, который отправляет данные в следующий этап, должен закрыть свой выходной канал после того, как все данные будут обработаны и отправлены. Это сигнализирует принимающему этапу об отсутствии новых данных и позволяет ему корректно завершить свою работу (например, через цикл `for range`).


**Пример реализации паттерна pipeline на Golang:**

Предположим, у нас есть задача обработать список строк:

1. **Этап 1 (generator):** Генерирует последовательность строк.
2. **Этап 2 (toUpper):** Преобразует каждую строку в верхний регистр.
3. **Этап 3 (filter):** Фильтрует строки, оставляя только те, которые содержат определенную подстроку.
4. **Этап 4 (printer):** Печатает отфильтрованные строки.


### Пример 1
```go
package main

import (
	"fmt"
	"sync"
	"time"
)

// generator генерирует последовательность строк и отправляет их в выходной канал.
func generator(strings []string) <-chan string {
	out := make(chan string)
	go func() {
		defer close(out) // Закрываем выходной канал после отправки всех строк
		for _, s := range strings {
			fmt.Printf("generator отправляет: %s\n", s)
			out <- s
			time.Sleep(time.Millisecond * 100) // Имитация работы
		}
	}()
	return out
}

// toUpper получает строки из входного канала, преобразует их в верхний регистр
// и отправляет в выходной канал.
func toUpper(in <-chan string) <-chan string {
	out := make(chan string)
	go func() {
		defer close(out) // Закрываем выходной канал после обработки всех строк
		for s := range in {
			upperS := fmt.Sprintf("%s (UPPER)", s)
			fmt.Printf("toUpper обработал: %s -> %s\n", s, upperS)
			out <- upperS
			time.Sleep(time.Millisecond * 150) // Имитация работы
		}
	}()
	return out
}

// filter получает строки из входного канала и отправляет в выходной канал
// только те строки, которые содержат подстроку "go".
func filter(in <-chan string) <-chan string {
	out := make(chan string)
	go func() {
		defer close(out) // Закрываем выходной канал после фильтрации всех строк
		for s := range in {
			if containsGo(s) {
				fmt.Printf("filter пропустил: %s\n", s)
				out <- s
				time.Sleep(time.Millisecond * 200) // Имитация работы
			} else {
				fmt.Printf("filter отфильтровал: %s\n", s)
			}
		}
	}()
	return out
}

// containsGo проверяет, содержит ли строка подстроку "go".
func containsGo(s string) bool {
	for i := 0; i < len(s)-1; i++ {
		if s[i:i+2] == "go" {
			return true
		}
	}
	return false
}

// printer получает строки из входного канала и печатает их.
func printer(in <-chan string, wg *sync.WaitGroup) {
	defer wg.Done()
	for s := range in {
		fmt.Printf("printer получил: %s\n", s)
		time.Sleep(time.Millisecond * 100) // Имитация работы
	}
	fmt.Println("printer завершил работу.")
}

func main() {
	strings := []string{"hello world", "golang is fun", "go routines", "concurrent programming", "no go here"}

	// Создаем этапы конвейера, соединяя их каналами.
	generated := generator(strings)
	upperCase := toUpper(generated)
	filtered := filter(upperCase)

	var wg sync.WaitGroup
	wg.Add(1)
	// Запускаем последний этап (printer) в отдельной горутине.
	go printer(filtered, &wg)

	// Ожидаем завершения работы последнего этапа.
	wg.Wait()

	fmt.Println("Конвейер завершил работу.")
}
```


### Пример 2 (с разделение потока выполнения на несколько горутин)
![[Pasted image 20250525115434.png]]
```go
package main  
  
import (  
    "fmt"  
    "sync")  
  
func parse(inputCh <-chan string) <-chan string {  
    outputCh := make(chan string)  
  
    go func() {  
       defer close(outputCh)  
       for data := range inputCh {  
          outputCh <- fmt.Sprintf("parsed - %s", data)  
       }  
    }()  
  
    return outputCh  
}  
  
func send(inputCh <-chan string, num int) <-chan string {  
    outputCh := make(chan string)  
    wg := sync.WaitGroup{}  
    wg.Add(num)  
  
    for i := 0; i < num; i++ {  
       go func() {  
          defer wg.Done()  
          for data := range inputCh {  
             outputCh <- fmt.Sprintf("sent - %s", data)  
          }  
       }()  
    }  
  
    go func() {  
       wg.Wait()  
       close(outputCh)  
    }()  
  
    return outputCh  
}  
  
func main() {  
    channel := make(chan string)  
    go func() {  
       defer close(channel)  
       for i := 0; i < 10; i++ {  
          channel <- "value"  
       }  
    }()  
  
    for value := range send(parse(channel), 2) {  
       fmt.Println(value)  
    }  
}
```


### Пример 3 (комбинирование pipeline + fan-out)

![[Pasted image 20250525115720.png]]

```go
package main  
  
import (  
    "fmt"  
    "sync")  
  
// split() - реализация паттерна "fan-out"  
func split[T any](inputCh <-chan T, n int) []chan T {  
    outputChans := make([]chan T, n)  
    for i := 0; i < n; i++ {  
       outputChans[i] = make(chan T)  
    }  
  
    go func() {  
       idx := 0  
       for value := range inputCh {  
          outputChans[idx] <- value  
          idx = (idx + 1) % n  
       }  
  
       for _, ch := range outputChans {  
          close(ch)  
       }  
    }()  
  
    return outputChans  
}  
  
func parse(inputCh <-chan string) <-chan string {  
    outputCh := make(chan string)  
  
    go func() {  
       defer close(outputCh)  
       for data := range inputCh {  
          outputCh <- fmt.Sprintf("parsed - %s", data)  
       }  
    }()  
  
    return outputCh  
}  
  
func send(inputCh <-chan string, num int) <-chan string {  
  
    wg := sync.WaitGroup{}  
    wg.Add(num)  
  
    //применим разделение канала на несколько каналов  
    splittedChans := split(inputCh, num)  
    outputCh := make(chan string)  
  
    for i := 0; i < num; i++ {  
       go func(idx int) {  
          defer wg.Done()  
          for data := range splittedChans[idx] {  
             outputCh <- fmt.Sprintf("sent - %s", data)  
          }  
       }(i)  
    }  
  
    go func() {  
       wg.Wait()  
       close(outputCh)  
    }()  
  
    return outputCh  
}  
  
func main() {  
    channel := make(chan string)  
    go func() {  
       defer close(channel)  
       for i := 0; i < 10; i++ {  
          channel <- "value"  
       }  
    }()  
  
    for value := range send(parse(channel), 2) {  
       fmt.Println(value)  
    }  
}
```