Паттерн **Worker Pool** используется для ограничения количества одновременно выполняющихся горутин, что позволяет более эффективно управлять ресурсами (например, CPU, память, сетевые соединения) при обработке большого количества задач.

Использование паттерна сводится к реализации функции (воркера) принимающей  в качестве параметра входящий и исходящий каналы, и в зависимости от реализации еще что-нибудь. Данные воркеры запускаются асинхронно в цикле принимают данные из входного канала, преобразуют данные, и возвращают преобразованные данные в выходной канал.


### Пример 1:
```go 
package main  
  
import (  
    "fmt"  
    "sync")  
  
func Pool(in <-chan int, numWorkers int, f func(int) int) <-chan int {  
    out := make(chan int)  
    wg := &sync.WaitGroup{}  
    wg.Add(1)  
    go func() {  
       defer wg.Done()  
       // запускается ограниченное количество воркеров согласно параметру numWorkers
       for range numWorkers {  
          go worker(in, out, f)  
       }  
    }()  
    wg.Wait()  
    return out  
}  


// воркер получает данные из in, преобразует, используя f(), кладет данные в out
func worker(in, out chan int, f func(int) int) {  
    for v := range in {  
       out <- f(v)  
    }  
    close(out)  
}  


// Вся логика преобразования данных - здеся
func sqr(v int) int {  
    return v * v  
}  
  
func main() {  
    in := make(chan int)  
	
	// генератор входных данных
    go func() {  
       for i := range 100 {  
          in <- i  
       }  
       close(in)  
    }()  
  
    // Запускается пул воркеров (Pool), преобразованные данные возвращаются в канал out для дальнейшей работы
    out := Pool(in, 10, sqr)  
    for v := range out {  
       fmt.Println(v)  
    }  
```

### Пример 2:
```go
package main

import (
	"fmt"
	"sync"
	"time"
)

// Task представляет собой задачу для выполнения
type Task struct {
	ID int
}

// Worker принимает задачи из канала и выполняет их
func workerPool(id int, jobs <-chan Task, wg *sync.WaitGroup) {
	defer wg.Done()
	fmt.Printf("Рабочий %d запущен.\n", id)
	for job := range jobs {
		fmt.Printf("Рабочий %d обрабатывает задачу %d\n", id, job.ID)
		time.Sleep(time.Millisecond * 500) // Имитация выполнения задачи
	}
	fmt.Printf("Рабочий %d завершил работу.\n", id)
}

func main() {
	numWorkers := 3
	numJobs := 10
	var wg sync.WaitGroup

	jobs := make(chan Task) // Канал для передачи задач рабочим

	// Отправляем задачи в канал 
	go func() {
		for i := 1; i <= numJobs; i++ {
			jobs <- Task{ID: i}
		}
		close(jobs) // Закрываем канал задач, чтобы сигнализировать рабочим об отсутствии новых задач
	}()

	// Запускаем пул рабочих
	for i := 1; i <= numWorkers; i++ {
		wg.Add(1)
		go workerPool(i, jobs, &wg)
	}

	wg.Wait() // Ожидаем завершения всех рабочих
	fmt.Println("Все задачи выполнены.")
}
```