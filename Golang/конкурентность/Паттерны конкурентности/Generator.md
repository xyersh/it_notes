используется для генерации последовательности данных

```go 
func Generator() <-chan int {
	ch := make(chan int)
	go func() {
		for val := range 10 {
			ch <- val
		}
		close(ch)
	}()
	return ch
}
```