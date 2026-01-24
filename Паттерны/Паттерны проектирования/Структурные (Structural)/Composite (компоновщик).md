Позволяет работать с древовидной структурой объектов так, будто это единый объект. Объединяет отдельные объекты и их композиции под общим интерфейсом.


**Когда использовать:**

- Есть иерархия «часть–целое» (например, файлы и папки, UI-элементы, DOM-дерево).
- Нужно выполнять одинаковые операции над отдельными объектами и группами объектов.
- Клиентский код не должен зависеть от того, работает ли он с листом или узлом дерева.

**Цель:**  
Упростить клиентский код, обстрагируясь от "древовидности" и "многоуровневости" предметных структур, и тем самым, упрощая API работы с ними. Яркий пример - файловая система. 

## Пример 
 файловая система
```go
package main

import "fmt"

// Component — общий интерфейс для файлов и папок
type Component interface {
	Name() string
	Size() int
}

// File — листовой элемент
type File struct {
	name string
	size int
}

func (f *File) Name() string { return f.name }
func (f *File) Size() int   { return f.size }

// Folder — составной элемент
type Folder struct {
	name     string
	children []Component
}

func (fd *Folder) Name() string { return fd.name }
func (fd *Folder) Size() int {
	total := 0
	for _, child := range fd.children {
		total += child.Size()
	}
	return total
}

func (fd *Folder) Add(c Component) {
	fd.children = append(fd.children, c)
}

// Пример использования
func main() {
	file1 := &File{name: "file1.txt", size: 100}
	file2 := &File{name: "file2.txt", size: 200}
	folder1 := &Folder{name: "docs"}
	folder1.Add(file1)
	folder1.Add(file2)

	file3 := &File{name: "image.png", size: 500}
	root := &Folder{name: "root"}
	root.Add(folder1)
	root.Add(file3)

	fmt.Printf("Root size: %d\n", root.Size()) // Root size: 800
}
```

- `File` — лист, реализует `Component`.
- `Folder` — контейнер, содержит другие `Component`, включая другие `Folder`.
- Клиент (`main`) работает с `root` как с единым объектом, не заботясь о внутренней структуре.