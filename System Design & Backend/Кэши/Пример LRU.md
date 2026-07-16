Для эффективной реализации LRU-кэша нам потребуются две основные структуры данных:
1. **Двусвязный список (Doubly Linked List):** Будет хранить ключи кэша в порядке их использования. Самый недавно использованный элемент будет находиться в начале списка, а наименее давно использованный — в конце. Это позволяет быстро добавлять и удалять элементы из любого конца.
2. **Хеш-таблица (Map):** Будет отображать ключи на узлы двусвязного списка. Это позволяет осуществлять доступ к элементам кэша по ключу за O(1) время и получать доступ к соответствующему узлу в списке для обновления его позиции.

```go
package main

import (
	"container/list"
	"fmt"
)

// CacheEntry представляет элемент, хранящийся в кэше.
// K - тип ключа, V - тип значения.
type CacheEntry[K comparable, V any] struct {
	Key   K
	Value V
}

// LRUCache представляет LRU-кэш.
// K - тип ключа, V - тип значения.
type LRUCache[K comparable, V any] struct {
	capacity int                   // Максимальная вместимость кэша
	ll       *list.List            // Двусвязный список для отслеживания порядка LRU
	cache    map[K]*list.Element // Хеш-таблица для быстрого доступа к узлам списка по ключу
}

// NewLRUCache создает новый экземпляр LRU-кэша.
func NewLRUCache[K comparable, V any](capacity int) *LRUCache[K, V] {
	if capacity <= 0 {
		panic("Capacity must be a positive integer")
	}
	return &LRUCache[K, V]{
		capacity: capacity,
		ll:       list.New(),
		cache:    make(map[K]*list.Element),
	}
}

// Get извлекает значение по ключу.
// Если ключ найден, элемент перемещается в начало списка (самый недавно использованный)
// и возвращается его значение и true.
// В противном случае возвращается нулевое значение и false.
func (c *LRUCache[K, V]) Get(key K) (V, bool) {
	if element, found := c.cache[key]; found {
		// Перемещаем элемент в начало списка, так как он был только что использован
		c.ll.MoveToFront(element)
		// Распаковываем CacheEntry из элемента списка
		entry := element.Value.(*CacheEntry[K, V])
		return entry.Value, true
	}
	var zeroValue V // Возвращаем нулевое значение для типа V
	return zeroValue, false
}

// Put добавляет или обновляет элемент в кэше.
// Если ключ уже существует, его значение обновляется, и он перемещается в начало списка.
// Если кэш полон, и ключа нет, удаляется наименее давно используемый элемент
// (из хвоста списка), и новый элемент добавляется в начало.
func (c *LRUCache[K, V]) Put(key K, value V) {
	if element, found := c.cache[key]; found {
		// Ключ уже существует, обновляем значение и перемещаем в начало
		entry := element.Value.(*CacheEntry[K, V])
		entry.Value = value // Обновляем значение
		c.ll.MoveToFront(element)
		return
	}

	// Ключа нет в кэше. Проверяем вместимость.
	if c.ll.Len() >= c.capacity {
		// Кэш полон, удаляем наименее давно используемый элемент (из конца списка)
		oldestElement := c.ll.Back()
		if oldestElement != nil {
			c.ll.Remove(oldestElement)
			oldestEntry := oldestElement.Value.(*CacheEntry[K, V])
			delete(c.cache, oldestEntry.Key) // Удаляем из хеш-таблицы
		}
	}

	// Добавляем новый элемент в начало списка
	newEntry := &CacheEntry[K, V]{Key: key, Value: value}
	newElement := c.ll.PushFront(newEntry)
	c.cache[key] = newElement // Добавляем в хеш-таблицу
}

// Size возвращает текущее количество элементов в кэше.
func (c *LRUCache[K, V]) Size() int {
	return c.ll.Len()
}

// PrintCache выводит содержимое кэша для отладки.
func (c *LRUCache[K, V]) PrintCache() {
	fmt.Print("Cache (LRU -> MRU): ")
	for e := c.ll.Back(); e != nil; e = e.Prev() {
		entry := e.Value.(*CacheEntry[K, V])
		fmt.Printf("[%v:%v] ", entry.Key, entry.Value)
	}
	fmt.Println()
}

func main() {
	// Создаем кэш для строк и целых чисел с вместимостью 3
	lruCache := NewLRUCache[string, int](3)

	fmt.Println("Добавляем A:1")
	lruCache.Put("A", 1)
	lruCache.PrintCache() // A

	fmt.Println("Добавляем B:2")
	lruCache.Put("B", 2)
	lruCache.PrintCache() // A B

	fmt.Println("Добавляем C:3")
	lruCache.Put("C", 3)
	lruCache.PrintCache() // A B C

	fmt.Println("\nПолучаем B (MRU)")
	valB, foundB := lruCache.Get("B")
	if foundB {
		fmt.Printf("Получено B: %v\n", valB)
	}
	lruCache.PrintCache() // A C B (B становится MRU)

	fmt.Println("\nДобавляем D:4 (A должен быть вытеснен)")
	lruCache.Put("D", 4)
	lruCache.PrintCache() // C B D (A вытеснен, так как был LRU)

	fmt.Println("\nПолучаем A (промах)")
	_, foundA := lruCache.Get("A")
	if !foundA {
		fmt.Println("A не найдено в кэше.")
	}

	fmt.Println("\nОбновляем C:5")
	lruCache.Put("C", 5)
	lruCache.PrintCache() // B D C (C становится MRU)

	fmt.Println("\nТекущий размер кэша:", lruCache.Size())
}
```


### Объяснение реализации:

1. **`CacheEntry[K comparable, V any]`**:
    - Это вспомогательная дженерик-структура, которая будет храниться в узлах двусвязного списка. Она содержит собственно **ключ** (`Key`) и **значение** (`Value`) элемента кэша.
    - Ограничение `K comparable` означает, что тип ключа должен быть сравнимым (например, `int`, `string`, `bool`), чтобы его можно было использовать как ключ в `map`.
    - `V any` означает, что тип значения может быть любым.
        
2. **`LRUCache[K comparable, V any]`**:
    - **`capacity int`**: Максимальное количество элементов, которое может хранить кэш.
    - **`ll *list.List`**: Указатель на экземпляр `container/list.List`. Этот двусвязный список будет хранить элементы `CacheEntry`. **Начало списка** (`Front`) — это **MRU (Most Recently Used)** элемент, а **конец списка** (`Back`) — это **LRU (Least Recently Used)** элемент.
    - **`cache map[K]*list.Element`**: Хеш-таблица (карта), которая отображает ключ на указатель на соответствующий узел `*list.Element` в двусвязном списке. Это позволяет нам находить элементы в списке за O(1) время, что критически важно для производительности.
        
3. **`NewLRUCache[K, V](capacity int) *LRUCache[K, V]`**:
    - Функция-конструктор для создания нового экземпляра кэша. Инициализирует `capacity`, создает новый `list.List` и новую `map`.
        
4. **`Get(key K) (V, bool)`**:
    - Пытается найти элемент по `key` в `c.cache`.
    - Если найден (`found` равно `true`):
        - Мы **перемещаем соответствующий узел списка в начало** (`c.ll.MoveToFront(element)`). Это фундаментальное действие для LRU-политики: каждый раз, когда элемент используется (читается), он становится "самым свежим".
        - Извлекаем `CacheEntry` из `element.Value` и возвращаем его `Value` и `true`.
    - Если не найден:
        - Возвращаем нулевое значение для типа `V` и `false`.
            
5. **`Put(key K, value V)`**:
    - **Обновление существующего элемента:**
        - Сначала проверяем, существует ли `key` уже в `c.cache`.
        - Если да, обновляем `entry.Value` и **перемещаем элемент в начало списка** (`c.ll.MoveToFront(element)`), так как он только что был использован (обновлен).
            
    - **Добавление нового элемента:**
        - Если `key` не найден, это новый элемент.
        - Проверяем, **полон ли кэш** (`c.ll.Len() >= c.capacity`).
        - Если полон:
            - Получаем **наименее давно используемый элемент** (`oldestElement := c.ll.Back()`) — он находится в хвосте списка.
            - Удаляем его из списка (`c.ll.Remove(oldestElement)`).
            - Удаляем соответствующую запись из хеш-таблицы (`delete(c.cache, oldestEntry.Key)`).
        - Создаем новый `CacheEntry` и **добавляем его в начало списка** (`c.ll.PushFront(newEntry)`).
        - Сохраняем ссылку на новый узел списка в хеш-таблице (`c.cache[key] = newElement`).
            
6. **`Size() int`**:
    - Просто возвращает текущее количество элементов в списке, что соответствует размеру кэша.
        
7. **`PrintCache()`**:
    - Вспомогательная функция для отладки, которая выводит содержимое кэша от LRU до MRU (от конца списка к началу).