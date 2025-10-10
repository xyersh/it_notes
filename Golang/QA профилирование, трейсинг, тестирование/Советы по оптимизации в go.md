
#### 1. Используем профилирование.
Измеряем! Не гадаем!

#### 2. Проводим бенчмарки
```go
package main

import (
    "strings"
    "testing"
)

var testData = []string{"a", "b", "c", "d", "e", "f", "g"}

func BenchmarkStringPlus(b *testing.B) {
    b.ReportAllocs() // Reports memory allocations per operation
    for i := 0; i < b.N; i++ {
        var result string
        for _, s := range testData {
            result += s
        }
    }
}

func BenchmarkStringBuilder(b *testing.B) {
    b.ReportAllocs()
    for i := 0; i < b.N; i++ {
        var builder strings.Builder
        for _, s := range testData {
            builder.WriteString(s)
        }
        _ = builder.String()
    }
}
```


#### 3.  Преаллокация емкости при создании слайсов и карт
Если нам известные примерные размеры будущего слайса/карты - есть смысл сразу  указывть этот размер емкости. Так как  последующие аллокации при постепенный росте структры  являются довольно дорогими операциями.
```go
const count = 10000

// Bad practice: append() will trigger multiple reallocations
s := make([]int, 0)
for i := 0; i < count; i++ {
    s = append(s, i)
}

// Recommended practice: Allocate enough capacity at once
s := make([]int, 0, count)
for i := 0; i < count; i++ {
    s = append(s, i)
}

// The same logic applies to maps
m := make(map[int]string, count)
```

#### 4. Переиспользование часто аллоцируемых объектов `sync.Pool`
Использование `sync.Pool`  позволяет переиспользовать память, тем самим - сократить количество аллокаций объектов. таким образом, нам не придется перевыделять память для короткоживущих, временных обхектов.
Сборщик мусора может почистить объекты, хранимые в пуле. По сему, пул подходит лишь для хранения временных, объектов без состояния.

```go
import (
    "bytes"
    "sync"
)

var bufferPool = sync.Pool{
    New: func() interface{} {
        return new(bytes.Buffer)
    },
}

func ProcessRequest(data []byte) {
    buffer := bufferPool.Get().(*bytes.Buffer)
    defer bufferPool.Put(buffer) // defer ensures the object is always returned
    buffer.Reset()               // Reset the object's state before reuse

    // ... use the buffer ...
    buffer.Write(data)
}
```


#### 5. Использование `strings.Builder` вместо конкатенации строки

#### 6. Избегаем утечек памяти при получении саб-слайса из большого слайса

```go
// Potential memory leak
func getSubSlice(data []byte) []byte {
    // The returned slice still references the entire underlying array of data
    return data[:10]
}

// The correct approach
func getSubSliceCorrectly(data []byte) []byte {
    sub := data[:10]
    result := make([]byte, 10)
    copy(result, sub) // Copy the data to new memory
    // result no longer has any association with the original data
    return result
}
```

#### 7. Компромиссы между указателями и значениями
Большие объекты выгднее передавать по ссылке, так как это избалвяет нас от лишних копирований объектов
Мелкие объекты лучше передавть в функции  по значению, так как при этом копии паратеров помещаются в стек а не в кучу. 
Конечный выбор между использованием указателей и значений стоит за бенчарком.


#### 8. Настройка GOMAXPPROC
Как правило, настройки являются оптимальными.
Крайне рекомендуется использовать библиотеку `uber-go/automaxprocs`  для автоматического подбора оптимального значения
#### 9. Иcпользование  буферизированных каналов
Испольщование небуферизированных каналов приводит к синхроннной работе горутин пишущих и читающих в такие каналы. Например задержки в работе консьюмеров сразу же приводят к задаржкам в работе продьюсеров, так как пока консьюмер не прочитает значение из канала - продьюсер будет заблокирован.
Использование буферизированных каналов позволяет уменьшить влияние таких всплесков.

#### 10. Иcпользование `sync.Wait` для настройки ожидания окончания работы   конкурентных задач

#### 11. Уменьшение влияния блокировок в высококонкурентных заадачах
- Уменьшаем критическую секцию , требующую блокировок. 
- Используем `sync.RWMutex` в сценариях с высокой нагрузкой на чтение
- Испольуем пакет `sync/atomic` для простых счетчиков и флагов. 
- Шардирование - делим мапу на несколько частей, каждая из которых владеет своим мьютексом.

#### 12. Пул воркеров - эффективный паттерн для управления конкурентностью
Выделение новых горутин для каждой задачи является антипаттерном, который может мгновенно истощить системную  память. Чтобы решить данную проьблему -  применяем пул воркеров 
```go
func worker(jobs <-chan int, results chan<- int) {
    for j := range jobs {
        // ... process job j ...
        results <- j * 2
    }
}

func main() {
    jobs := make(chan int, 100)
    results := make(chan int, 100)

    // Start 5 workers
    for w := 1; w <= 5; w++ {
        go worker(jobs, results)
    }

    // ... send tasks to the jobs channel ...
    close(jobs)

    // ... collect results from the results channel ...
}
```

#### 13. Используем `map[key]struct{}` для множеств
```go
// More memory efficient
set := make(map[string]struct{})
set["apple"] = struct{}{}
set["banana"] = struct{}{}

// Check for existence
if _, ok := set["apple"]; ok {
    // exists
}
```

#### 14. Избегаем ненужных вычислений в горячих циклах 
```go
items := []string{"a", "b", "c"}

// Bad practice: len(items) is called in every iteration
for i := 0; i < len(items); i++ { /* ... */ }

// Recommended practice: Pre-calculate the length
length := len(items)
for i := 0; i < length; i++ { /* ... */ }
```

#### 15. Понимание стоимости использования интерфейсов
Использование интерфейсов влечет к дополнительным задержакам производительности.
В критическом по производительности  коде следует избегать использования интерфейсов в пользу прямого использования конкретных типов.
Если `pprof` показывает что показатели `runtime.convT2I` или `runtime.assertI2T` потребляют значительные ресурсы процессора - это явный сигнал к рефакторингу.

#### 16. Уменьшаем размер бинарников в продакшн
По умолчанию бинарники хранят дебажную информацию. Это полезно при разработке ,НО избыточно при  работе ПО при промышленной эксплуатации. Удаление дебажной инфы значительно уменьшает размер бинарника, что ускоряет  создание образа контейнера и его распространение.
Добиться уменьшения бинаря можно с помощью использования соответствующих  атрибутов при его создании:
```bash
go build -ldflags="-s -w" myapp.go
```
Где:
	- `-s:` Удаляет неиспользуемые символы из итогового файла, уменьшая его размер.
	- `-w:` Удаляет информацию об отладке.


#### 17. Понимание escape-анализа
Используем `-gcflags="-m"` при сборке бинаря:
```bash
go build -gcflags="-m"`
```



```go
func getInt() *int {
    i := 10
    return &i // &i "escapes to heap"
}
```
по данному куску кода компилятор отобразить решение escape-анализа: `escapes to heap`


#### 18. Оцениваем стоимость `cgo`-вызовов


https://dev.to/leapcell/20-go-performance-tricks-i-learned-the-hard-way-2h8h