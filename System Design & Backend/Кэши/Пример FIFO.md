Для эффективной реализации FIFO-кэша нам потребуются две основные структуры данных:

1. **Двусвязный список (`container/list.List`):** Будет хранить ключи или сами элементы кэша в порядке их добавления. Самый старый (первый добавленный) элемент будет находиться в конце списка, а самый новый — в начале. Это позволяет легко удалять самый старый элемент из конца и добавлять новый в начало.
    
2. **Хеш-таблица (`map`):** Будет отображать ключи на узлы двусвязного списка (или на сами элементы кэша), чтобы обеспечить O(1) доступ к элементам по ключу.

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

// FIFOCache представляет FIFO-кэш.
// K - тип ключа, V - тип значения.
type FIFOCache[K comparable, V any] struct {
	capacity int                   // Максимальная вместимость кэша
	queue    *list.List            // Двусвязный список для отслеживания порядка FIFO (старейшие в конце)
	cache    map[K]*list.Element // Хеш-таблица для быстрого доступа к узлам очереди по ключу
}

// NewFIFOCache создает новый экземпляр FIFO-кэша.
func NewFIFOCache[K comparable, V any](capacity int) *FIFOCache[K, V] {
	if capacity <= 0 {
		panic("Capacity must be a positive integer")
	}
	return &FIFOCache[K, V]{
		capacity: capacity,
		queue:    list.New(),
		cache:    make(map[K]*list.Element),
	}
}

// Get извлекает значение по ключу.
// В FIFO-кэше операция Get не влияет на порядок элементов в очереди.
// Если ключ найден, возвращается его значение и true.
// В противном случае возвращается нулевое значение и false.
func (c *FIFOCache[K, V]) Get(key K) (V, bool) {
	if element, found := c.cache[key]; found {
		// Для FIFO-кэша Get не изменяет порядок.
		entry := element.Value.(*CacheEntry[K, V])
		return entry.Value, true
	}
	var zeroValue V // Возвращаем нулевое значение для типа V
	return zeroValue, false
}

// Put добавляет или обновляет элемент в кэше.
// Если ключ уже существует, его значение просто обновляется.
// Если кэш полон и ключа нет, удаляется самый старый элемент
// (из хвоста очереди), и новый элемент добавляется в начало.
func (c *FIFOCache[K, V]) Put(key K, value V) {
	if element, found := c.cache[key]; found {
		// Ключ уже существует, просто обновляем значение.
		// Порядок в FIFO не меняется при обновлении.
		entry := element.Value.(*CacheEntry[K, V])
		entry.Value = value
		return
	}

	// Ключа нет в кэше.
	// Проверяем вместимость.
	if c.queue.Len() >= c.capacity {
		// Кэш полон, удаляем самый старый элемент (первый вошел, первый вышел)
		// Самый старый элемент находится в конце двусвязного списка.
		oldestElement := c.queue.Back()
		if oldestElement != nil {
			c.queue.Remove(oldestElement)
			oldestEntry := oldestElement.Value.(*CacheEntry[K, V])
			delete(c.cache, oldestEntry.Key) // Удаляем из хеш-таблицы
		}
	}

	// Добавляем новый элемент в начало очереди.
	newEntry := &CacheEntry[K, V]{Key: key, Value: value}
	newElement := c.queue.PushFront(newEntry) // PushFront добавляет в начало
	c.cache[key] = newElement                 // Добавляем в хеш-таблицу
}

// Size возвращает текущее количество элементов в кэше.
func (c *FIFOCache[K, V]) Size() int {
	return c.queue.Len()
}

// PrintCache выводит содержимое кэша для отладки.
// Элементы выводятся от самого нового (MRU) до самого старого (FIFO/LRU).
func (c *FIFOCache[K, V]) PrintCache() {
	fmt.Print("FIFO Cache (Newest -> Oldest): ")
	for e := c.queue.Front(); e != nil; e = e.Next() {
		entry := e.Value.(*CacheEntry[K, V])
		fmt.Printf("[%v:%v] ", entry.Key, entry.Value)
	}
	fmt.Println()
}

func main() {
	// Создаем FIFO-кэш для строк и целых чисел с вместимостью 3
	fifoCache := NewFIFOCache[string, int](3)

	fmt.Println("Добавляем A:1")
	fifoCache.Put("A", 1)
	fifoCache.PrintCache() // A

	fmt.Println("Добавляем B:2")
	fifoCache.Put("B", 2)
	fifoCache.PrintCache() // B A

	fmt.Println("Добавляем C:3")
	fifoCache.Put("C", 3)
	fifoCache.PrintCache() // C B A

	fmt.Println("\nПолучаем B (не влияет на порядок FIFO)")
	valB, foundB := fifoCache.Get("B")
	if foundB {
		fmt.Printf("Получено B: %v\n", valB)
	}
	fifoCache.PrintCache() // C B A (порядок тот же)

	fmt.Println("\nДобавляем D:4 (A должен быть вытеснен, так как он самый старый)")
	fifoCache.Put("D", 4)
	fifoCache.PrintCache() // D C B (A вытеснен)

	fmt.Println("\nПолучаем A (промах)")
	_, foundA := fifoCache.Get("A")
	if !foundA {
		fmt.Println("A не найдено в кэше.")
	}

	fmt.Println("\nОбновляем C:5 (не влияет на порядок FIFO)")
	fifoCache.Put("C", 5)
	fifoCache.PrintCache() // D C B (порядок тот же, значение C изменилось)

	fmt.Println("\nДобавляем E:6 (B должен быть вытеснен)")
	fifoCache.Put("E", 6)
	fifoCache.PrintCache() // E D C (B вытеснен)

	fmt.Println("\nТекущий размер кэша:", fifoCache.Size()) // 3
}
```


###  Основные Принципы FIFO
- **"Первый вошёл — первый вышел"**: Элемент, который был добавлен в кэш раньше всех, будет вытеснен первым, когда кэш достигнет своей максимальной вместимости и потребуется место для нового элемента.
- **Использование не влияет на порядок**: В отличие от LRU или LFU, обращение к элементу (операция `Get`) не изменяет его позицию в очереди кэша.
    
###  Выбор Структур Данных
- **`CacheEntry[K comparable, V any]`**:
    - `Key K`: Ключ элемента.
    - `Value V`: Значение элемента.
    - Эта структура просто инкапсулирует данные, которые мы храним в кэше.
- **`FIFOCache[K comparable, V any]`**:
    - `capacity int`: Целое число, определяющее максимальное количество элементов, которое кэш может одновременно хранить. Это фиксированный размер.
    - `queue *list.List`: Это указатель на экземпляр двусвязного списка из стандартной библиотеки Go (`container/list`).
        - **Назначение**: `queue` хранит `*CacheEntry` в порядке их добавления. Мы будем добавлять новые элементы в _начало_ списка (`PushFront`) и удалять самые старые элементы из _конца_ списка (`Back()`, затем `Remove`). Это создает эффект очереди "первый вошёл — первый вышел".
    - `cache map[K]*list.Element`: Это хеш-таблица (Go `map`).
        - **Назначение**: `cache` отображает ключ (`K`) на `*list.Element` из `queue`, который содержит соответствующий `CacheEntry`. Это позволяет нам выполнять операции `Get` и `Put` за O(1) время, так как мы можем мгновенно найти элемент по ключу.
            

### Разбор Методов
####  `NewFIFOCache[K, V](capacity int) *FIFOCache[K, V]`
- **Назначение**: Конструктор для создания нового экземпляра FIFO-кэша.
- **Что делает**:
        - Проверяет, что `capacity` положительно, чтобы избежать некорректных состояний.
        - Инициализирует `capacity`.
        - Создаёт новый пустой двусвязный список для `queue`.
        - Создаёт новую пустую `map` для `cache`.
            
#### `Get(key K) (V, bool)`
- **Назначение**: Получить значение, связанное с данным ключом.
- **Что делает**:
        - Использует `c.cache[key]` для быстрого поиска `*list.Element`, связанного с `key`. Это занимает O(1) время.
        - Если `element` найден (`found` равно `true`):
            - Извлекает `CacheEntry` из `element.Value` (поскольку `list.Element.Value` имеет тип `interface{}`, требуется приведение типа).
            - Возвращает `entry.Value` и `true`.
        - Если `element` не найден (`found` равно `false`):
            - Возвращает нулевое значение для типа `V` (например, `0` для `int`, `""` для `string`, `nil` для указателей) и `false`, указывая, что элемент отсутствует в кэше.
        - **Важная особенность FIFO**: Операция `Get` **не изменяет** порядок элементов в `queue`. Элемент остаётся на своём месте, независимо от того, как часто к нему обращаются.
            
####  `Put(key K, value V)`
- **Назначение**: Добавить новый элемент в кэш или обновить значение существующего элемента.
- **Что делает**:
        1. **Проверка существования ключа**:
            - Сначала проверяет, существует ли `key` уже в `c.cache`.
            - Если `element` найден (`found` равно `true`):
                - Извлекает `CacheEntry`.
                - Просто **обновляет `entry.Value = value`**.
                - Выходит из метода. Порядок в `queue` не меняется, так как это FIFO.
        2. **Добавление нового элемента (если `key` не найден)**:
            - **Проверка на переполнение**: Сравнивает текущий размер очереди (`c.queue.Len()`) с `c.capacity`.
            - Если `c.queue.Len() >= c.capacity` (кэш полон):
                - Находит самый старый элемент: `oldestElement := c.queue.Back()`. В двусвязном списке, элемент, добавленный первым, будет находиться в конце, когда новые добавляются в начало.
                - Удаляет этот `oldestElement` из `queue`: `c.queue.Remove(oldestElement)`.
                - Извлекает `oldestEntry` из удалённого `oldestElement`.
                - Удаляет `oldestEntry.Key` из `c.cache`: `delete(c.cache, oldestEntry.Key)`. Это синхронизирует `map` с `queue`.
            - **Добавление нового элемента**:
                - Создаёт новый `CacheEntry` с заданными `key` и `value`.
                - Добавляет этот `newEntry` в _начало_ `queue`: `newElement := c.queue.PushFront(newEntry)`. `PushFront` возвращает `*list.Element`, который был добавлен.
                - Сохраняет этот `newElement` в `c.cache` по `key`: `c.cache[key] = newElement`. Это связывает ключ с его положением в очереди.
                    
####  `Size() int`
- **Назначение**: Возвращает текущее количество элементов, хранящихся в кэше.
- **Что делает**: Просто возвращает длину двусвязного списка (`c.queue.Len()`).
        
####  `PrintCache()`
 **Назначение**: Вспомогательная функция для отладки, которая выводит содержимое кэша.
 **Что делает**: Итерируется по `queue` от начала (`Front()`) до конца (`Next()`), выводя ключи и значения элементов. Это показывает порядок "от самого нового до самого старого".