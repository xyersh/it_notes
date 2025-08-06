
### 1. Компиляция
#### regexp.Compile 
- компиляция шаблона.
- если компиляция шаблона прошла с ошибкой  regexp.Compile  вернет ошибку:
```go
re, err := regexp.Compile(`[a-z]+`)if err != nil {    
	log.Fatal("Ошибка компиляции регулярного выражения:",err)
}
fmt.Println("Регулярка скомпилирована:", re)
```

####  regexp.MustCompile
- если компиляция с ошибкой - паника
- используется, если регулярка статическая и шибки быть не может

### 2. Поиск соответствий, методы Match, MatchString, MatchReader
Методы получают на вход содержимое в виде сторки, байтового среза, или потока Reader, возвращают ответ true-сопоставление с шаблоном найдено, или false - если нет. 

- `MatchString(s string) bool` — проверяет **строку**.
- `Match(b []byte) bool` — проверяет **массив байтов**. Быстрее чем `MatchString`
- `MatchReader(io.RuneReader) bool` — проверяет **потоковый источник (io.Reader)**, полезно для больших файлов и сетевых данных, так как в этом случае данные файла не прогружаются в память полностью, а постепенно.


#### func (re \*[Regexp](https://pkg.go.dev/regexp@go1.24.1#Regexp)) MatchString(s [string](https://pkg.go.dev/builtin#string)) [bool](https://pkg.go.dev/builtin#bool) - пример

Проверка, является ли строка **числом** (состоит только из цифр):
```go

package main
import (	
"fmt"	
"regexp")

func main() {	
	pattern := `^\d+$` // Только цифры от начала до конца строки	
	re := regexp.MustCompile(pattern)	
	fmt.Println(re.MatchString("12345"))  // true  (всё цифры) 
	fmt.Println(re.MatchString("12a45"))  // false (есть буква)	
	fmt.Println(re.MatchString("00123"))  // true  (ноль допустим)
	fmt.Println(re.MatchString(""))       // false (пустая строка)}
```

#### func (re \*[Regexp](https://pkg.go.dev/regexp@go1.24.1#Regexp)) Match(b \[\][byte](https://pkg.go.dev/builtin#byte)) [bool](https://pkg.go.dev/builtin#bool) - пример
работает аналогично `MatchString`, но принимает `[]byte`, а не `string`
Проверка, является ли cодержимое байтового массива **числом** (состоит только из цифр):
```go
package main
import (	
"fmt"	
"regexp")

func main() {	
re := regexp.MustCompile(`^\d+$`)	
fmt.Println(re.Match([]byte("12345")))  // true
fmt.Println(re.Match([]byte("12a45")))  // false}
```

Проверка, является ли значение переданного HTTP-запросом параметра "email" валидным 
```go

package main
import (	
"fmt"	
"net/http"	
"regexp")

var emailRegex = regexp.MustCompile(`^[a-z0-9._%+-]+@[a-z0-9.-]+\.[a-z]{2,}$`)
func handler(w http.ResponseWriter, r *http.Request) {	
	email := []byte(r.URL.Query().Get("email"))	
	if emailRegex.Match(email) {		
		fmt.Fprintln(w, "Email корректный")	
	} else {		
		fmt.Fprintln(w, "Некорректный email")	
	}
}

func main() {	
	http.HandleFunc("/", handler)	
	http.ListenAndServe(":8080", nil)
}
```


#### MatchReader - пример

Проверка, содержит ли строка из io.Reader какое либо число:
```go
package main

import (	
"fmt"	
"regexp"	
"strings")

func main() {	
	re := regexp.MustCompile(`\d+`)	
	reader := strings.NewReader("Значение: 42")
	fmt.Println(re.MatchReader(reader)) // true (число найдено)
	}
```


### 3. Поиск данных: Find, FindAll, FindIndex
#### FindString — найти первое совпадение:
```go
re := regexp.MustCompile(`\d+`)	
str := "Цена: 123 руб, скидка: 456 руб."	
fmt.Println(re.FindString(str)) // "123"
```

#### FindAllString — найти все совпадения
Если совпадений **нет**, вернётся **пустой срез** (`[]string{}`), а не `nil`.
```go
fmt.Println(re.FindAllString(str, -1)) // ["123", "456"]
```
где второй аргумент функции означает:
- `-1` — найти **всё**.
- `2` — вернуть **только два первых совпадения**.
- `1` — вернуть **только одно совпадение** (аналог `FindString`).

#### FindStringIndex — находит позиции первого совпадения
если совпадений нет - вернет `nil`

#### FindAllStringIndex - находит позиции всех совпадений
```go
fmt.Println(re.FindAllStringIndex(str, -1))  [[6 9] [18 21]]
```
где второй аргумент функции означает:
- `-1` — найти **всё**.
- `2` — вернуть **только два первых совпадения**.
- `1` — вернуть **только одно совпадение** (аналог `FindString`).


#### FindReaderIndex(io.Reader) - ищет первое соответствие из потока чтения (io.Reader)
```go
package main
import (	
"fmt"	
"regexp"	
"strings")

func main() {	
	re := regexp.MustCompile(`\d+`)	
	reader := strings.NewReader("Цена: 123 руб, скидка: 456 руб.")
	fmt.Println(re.FindReaderIndex(reader)) // [6 9]}
```



#### FindStringSubmatchIndex(s string, i int ) \[\]\[\]int - 
возвращает индексы всех соответствий, в том числе и подгрупп
если соответствия не найдены вернет пустой слайс
```go
re := regexp.MustCompile(`a(x*)b`)
	// Indices:
	//    01234567   012345678
	//    -ab-axb-   -axxb-ab-
	fmt.Println(re.FindAllStringSubmatchIndex("-ab-", -1)) //[[1 3 2 2]]
	fmt.Println(re.FindAllStringSubmatchIndex("-axxb-", -1)) //[[1 5 2 4]]
	fmt.Println(re.FindAllStringSubmatchIndex("-ab-axb-", -1))//[[1 3 2 2] [4 7 5 6]]
	fmt.Println(re.FindAllStringSubmatchIndex("-axxb-ab-", -1))//[[1 5 2 4] [6 8 7 7]]
	fmt.Println(re.FindAllStringSubmatchIndex("-foo-", -1))//[]
```



#### FindStringSubmatch(s string) []string - находит не только совпадения но и  подгруппы
```go
re := regexp.MustCompile(`(\d{3})-(\d{3})-(\d{4})`)
str := "Телефон: 123-456-7890"
matches := re.FindStringSubmatch(str)
fmt.Println(matches) // ["123-456-7890" "123" "456" "7890"]
```



#### FindAllStringSubmatch(s string) \[\]\[\]string - 

```go
    re := regexp.MustCompile(`(\d{3})-(\d{3})-(\d{4})`)
    str := "Телефоны: 123-456-7890, 987-654-3210"
    allMatches := re.FindAllStringSubmatch(str, -1)
    fmt.Println(allMatches) //[[123-456-7890 123 456 7890] [987-654-3210 987 654 3210]]
```


