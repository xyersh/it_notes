**Бинарное дерево (BT):** Это дерево, в котором у каждого узла может быть **не более двух** потомков (левый и правый). Никаких правил относительно того, какие значения и где должны находиться, нет.


### Термины и характеристики BT

- **Высота бинарного дерева** - самый длинный путь от корневой ноды дерева до ее листа.  
- **Сбалансированное дерево** - дерево в каждом узле которого разница в высоте  левого поддерева и правого поддерева не превышает 1.
- **Полное бинарное дерево** - это дерево , у которого каждый уровень(кроме последнего) является полностью заполненным. И заполнение крайних элементов должно прововодиться слева направо
- Высота бинарного дерева приблизительно равна логарифму от количества узлов(H = logN). 
- Следовательно, если нам известна высота сбалансированного дерева, то мы можем узнать приблизительно количество его нод (N = 2^H).
- **Бинарное дерево поиска (binary search tree)** - подмножество бинарных деревьев, у которых каждая левый потомок содержит значение меньше чем родитель, а каждый правый потомок - большее чем родитель.
  
### Bibary Search Tree
BST - это частный случай BT. В дополнение к требованиям BT, BST также должен уудовлетворять **правилу порядка**:
- Значение в **левом** поддереве всегда **меньше** значения родительского узла.
- Значение в **правом** поддереве всегда **больше** значения родительского узла.
- Это правило рекурсивно применяется ко всем узлам дерева.



#### Асимптотическая сложность проведения операций над BST

Порядок следования элементов  в BST позволяет проводить операции вставки, поиска, удаления элементов со сложностью O(logN)  при условии что дерево является сбалансированным (в худшем случае сложность составит O(N)

##### Средний случай: $O(\log n)$

Если дерево сбалансировано, то при каждом шаге поиска (или вставки) мы отсекаем ровно половину оставшихся вариантов. Это работает точно так же, как бинарный поиск в отсортированном массиве.

Например, в сбалансированном дереве из 1 000 000 узлов путь от корня до любого элемента составит всего около 20 шагов.


##### Худший случай: $O(n)$

Это происходит, когда данные вставляются в уже отсортированном порядке (например, 1, 2, 3, 4, 5). В таком случае каждый новый узел становится правым потомком предыдущего, и дерево превращается в связный список (так называемое «вырожденное» дерево).

Чтобы найти элемент в конце такой «палки», нам придется пройти через все $n$ узлов.




```go
package main

import (
	"fmt"
	"container/list"
)

// Node представляет узел в бинарном дереве.
type Node struct {
	Value int
	Left  *Node
	Right *Node
}

// Tree представляет бинарное дерево.
type Tree struct {
	Root *Node
}

// insertRecursive рекурсивно вставляет новый узел.
func (t *Tree) insertRecursive(currentNode *Node, value int) *Node {
	// Базовый случай: если текущий узел nil, создаем и возвращаем новый узел.
	if currentNode == nil {
		return &Node{Value: value, Left: nil, Right: nil}
	}

	// Рекурсивный шаг: сравниваем значение и идем дальше.
	if value < currentNode.Value {
		// Если значение меньше, идем в левое поддерево.
		currentNode.Left = t.insertRecursive(currentNode.Left, value)
	} else if value > currentNode.Value {
		// Если значение больше, идем в правое поддерево.
		currentNode.Right = t.insertRecursive(currentNode.Right, value)
	}
	// Если значение равно, ничего не делаем, так как дубликаты не допускаются.

	return currentNode
}

// Insert - публичный метод для вставки нового значения в дерево.
func (t *Tree) Insert(value int) {
	t.Root = t.insertRecursive(t.Root, value)
}

// searchRecursive рекурсивно ищет узел с заданным значением.
func (t *Tree) searchRecursive(currentNode *Node, value int) *Node {
	// Базовый случай: если узел не найден или мы достигли nil.
	if currentNode == nil || currentNode.Value == value {
		return currentNode
	}

	// Рекурсивный шаг: определяем, куда идти дальше.
	if value < currentNode.Value {
		// Если значение меньше, ищем в левом поддереве.
		return t.searchRecursive(currentNode.Left, value)
	}
	// Если значение больше, ищем в правом поддереве.
	return t.searchRecursive(currentNode.Right, value)
}

// Search - публичный метод для поиска значения.
func (t *Tree) Search(value int) bool {
	node := t.searchRecursive(t.Root, value)
	return node != nil
}




// findMin находит узел с минимальным значением в поддереве.
func (t *Tree) findMin(node *Node) *Node {
	current := node
	// Идем по левым потомкам, пока не найдем самый левый узел.
	for current.Left != nil {
		current = current.Left
	}
	return current
}



// deleteRecursive рекурсивно удаляет узел с заданным значением.
func (t *Tree) deleteRecursive(currentNode *Node, value int) *Node {
	// Базовый случай: если дерево пустое или узел не найден.
	if currentNode == nil {
		return nil
	}

	// Рекурсивный шаг: ищем узел для удаления.
	if value < currentNode.Value {
		// Идем в левое поддерево.
		currentNode.Left = t.deleteRecursive(currentNode.Left, value)
	} else if value > currentNode.Value {
		// Идем в правое поддерево.
		currentNode.Right = t.deleteRecursive(currentNode.Right, value)
	} else {
		// Узел найден.
		// Случай 1: у узла нет потомков или один потомок.
		if currentNode.Left == nil {
			return currentNode.Right
		}
		if currentNode.Right == nil {
			return currentNode.Left
		}

		// Случай 2: у узла два потомка.
		// Находим наименьший узел в правом поддереве.
		minRight := t.findMin(currentNode.Right)
		// Заменяем значение текущего узла на значение найденного минимума.
		currentNode.Value = minRight.Value
		// Рекурсивно удаляем этот наименьший узел из правого поддерева.
		currentNode.Right = t.deleteRecursive(currentNode.Right, minRight.Value)
	}

	return currentNode
}

// Delete - публичный метод для удаления значения.
func (t *Tree) Delete(value int) {
	t.Root = t.deleteRecursive(t.Root, value)
}


// DFS PreOrderTraversal (Прямой обход: корень -> левый -> правый)
func (t *Tree) PreOrderTraversal(node *Node) {
	if node != nil {
		fmt.Printf("%d ", node.Value)
		t.PreOrderTraversal(node.Left)
		t.PreOrderTraversal(node.Right)
	}
}

// DFS InOrderTraversal (Симметричный обход: левый -> корень -> правый)
// Этот обход возвращает отсортированный список элементов для бинарного дерева поиска.
func (t *Tree) InOrderTraversal(node *Node) {
	if node != nil {
		t.InOrderTraversal(node.Left)
		fmt.Printf("%d ", node.Value)
		t.InOrderTraversal(node.Right)
	}
}

// DFS PostOrderTraversal (Обратный обход: левый -> правый -> корень)
func (t *Tree) PostOrderTraversal(node *Node) {
	if node != nil {
		t.PostOrderTraversal(node.Left)
		t.PostOrderTraversal(node.Right)
		fmt.Printf("%d ", node.Value)
	}
}


// BFS LevelOrderTraversal (Обход в ширину )
func (t *Tree) LevelOrderTraversal() {
	if t.Root == nil {
		return
	}

	// Используем двусвязный список (container/list) как очередь.
	queue := list.New()
	queue.PushBack(t.Root)

	for queue.Len() > 0 {
		element := queue.Front()
		queue.Remove(element)

		node := element.Value.(*Node)
		fmt.Printf("%d ", node.Value)

		if node.Left != nil {
			queue.PushBack(node.Left)
		}
		if node.Right != nil {
			queue.PushBack(node.Right)
		}
	}
}



func main() {
	// Создаем новое дерево.
	tree := &Tree{}

	// Вставляем элементы в дерево.
	fmt.Println("Вставка элементов:")
	tree.Insert(50)
	tree.Insert(30)
	tree.Insert(70)
	tree.Insert(20)
	tree.Insert(40)
	tree.Insert(60)
	tree.Insert(80)
	tree.Insert(10)
	tree.Insert(90)
	

	// Выполняем обходы.
	fmt.Println("\nПрямой обход (Pre-order):")
	tree.PreOrderTraversal(tree.Root) // Ожидаемый вывод: 50 30 20 10 40 70 60 80 90

	fmt.Println("\nСимметричный обход (In-order):")
	tree.InOrderTraversal(tree.Root) // Ожидаемый вывод: 10 20 30 40 50 60 70 80 90 (отсортированный)

	fmt.Println("\nОбратный обход (Post-order):")
	tree.PostOrderTraversal(tree.Root) // Ожидаемый вывод: 10 20 40 30 60 90 80 70 50

	fmt.Println("\nОбход в ширину (Level-order):")
	tree.LevelOrderTraversal() // Ожидаемый вывод: 50 30 70 20 40 60 80 10 90

	// Поиск элементов.
	fmt.Println("\n\nПоиск элемента 40:", tree.Search(40)) // Ожидаемый вывод: true
	fmt.Println("Поиск элемента 99:", tree.Search(99)) // Ожидаемый вывод: false

	// Удаление элемента.
	fmt.Println("\nУдаление элемента 30.")
	tree.Delete(30)

	fmt.Println("Симметричный обход после удаления:")
	tree.InOrderTraversal(tree.Root) // Ожидаемый вывод: 10 20 40 50 60 70 80 90
}
```