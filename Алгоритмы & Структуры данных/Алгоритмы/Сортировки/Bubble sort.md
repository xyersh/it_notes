```go
package main

import "fmt"

func main() {
	sl := []int{1, 5, 2, 4, 9, 6, 7, 3}
	fmt.Println("Before sorting: ", sl)

	BubbleSort(sl)

	fmt.Println("After sorting: ", sl)
}

func BubbleSort(sl []int) {
	var swiped bool = true
	for swiped == true {
		swiped = false
		for i := 0; i < len(sl)-1; i++ {
			if sl[i] > sl[i+1] {
				sl[i], sl[i+1] = sl[i+1], sl[i]
				swiped = true
			}
		}
	}

}
```