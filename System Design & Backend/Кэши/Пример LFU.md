
Для достижения  цели оптимальной реализации LFU  нужна структура данных, которая позволяет:
1. **Быстро найти элемент по ключу** (для операций `Get` и `Put`).
2. **Быстро узнать частоту использования элемента**.
3. **Быстро увеличить частоту элемента** и переместить его в "корзину" с новой частотой.
4. **Быстро найти элемент с наименьшей частотой** для вытеснения при переполнении.
    

Этот подход позволяет нам быстро находить наименее часто используемый элемент (он будет в первом списке частот), а также быстро перемещать элементы между списками частот при их использовании.

### Выбор Структур Данных для O(1) LFU

- **`map[K]*CacheEntry[K, V]` (карта `cache`):** Это наша основная хеш-таблица, которая отображает ключ (`K`) на сам объект `CacheEntry`. Это позволяет нам находить любой элемент по ключу за O(1) время.
    
- **`container/list.List` (список `freqList`):** Это **главный двусвязный список**, который хранит **узлы частот (`*FreqNode[K, V]`)**. Узлы в этом списке упорядочены по возрастанию их частоты. Самый первый узел (`freqList.Front()`) всегда будет представлять наименьшую частоту, присутствующую в кэше.
    
- **`container/list.List` (список `FreqNode.Entries`):** Каждый `FreqNode` внутри `freqList` содержит свой **собственный двусвязный список**, в котором хранятся **все `CacheEntry` с _одинаковой_ частотой**. Элементы внутри этого списка упорядочены по принципу **LRU**: недавно использованные находятся в начале, наименее давно использованные — в конце. Это обеспечивает "тай-брейк" (разрешение конфликтов) при одинаковой частоте.

### Определение Структур

- #### `CacheEntry[K comparable, V any]`
    - `Key K`: Ключ кэшируемого элемента. `comparable` означает, что его можно использовать как ключ в `map`.
    - `Value V`: Значение кэшируемого элемента. `any` означает любой тип.
    - `Freq int`: **Частота** использования данного элемента.
    - `parentFn *FreqNode[K, V]`: **Критически важное поле!** Это прямой указатель на **`FreqNode`**, которому в данный момент принадлежит этот `CacheEntry`. Позволяет нам мгновенно узнать, в каком списке частот находится `CacheEntry`.
    - `entryElem *list.Element`: **Также критически важное поле!** Это прямой указатель на **`*list.Element`** внутри `parentFn.Entries` (внутреннего списка `FreqNode`), который _содержит_ данный `CacheEntry`. Позволяет нам мгновенно удалить `CacheEntry` из `parentFn.Entries`.
        
- #### `FreqNode[K comparable, V any]`
    - `Frequency int`: Частота, которую представляет этот узел (например, 1, 2, 3 и т.д.).
    - `Entries *list.List`: Двусвязный список, хранящий **`*CacheEntry`**, у которых частота равна `Frequency`.
    - `freqListElem *list.Element`: **Критически важное поле!** Это прямой указатель на **`*list.Element`** в **главном списке `LFUCache.freqList`**, который _содержит_ данный `FreqNode`. Позволяет нам мгновенно удалить `FreqNode` из `freqList`, если он опустел.
        
- #### `LFUCache[K comparable, V any]`
    - `capacity int`: Максимальное количество элементов, которые кэш может хранить.
    - `minFreq int`: **Минимальная частота**, которая в данный момент присутствует в кэше. Это поле позволяет быстро найти `FreqNode` (который всегда будет `freqList.Front().Value.(*FreqNode).Frequency`), из которого нужно удалить элемент при переполнении.
    - `freqList *list.List`: Указатель на **главный двусвязный список `FreqNode`**.
    - `cache map[K]*CacheEntry[K, V]`: Указатель на **главную хеш-таблицу**, которая отображает ключи на объекты `*CacheEntry`.

### Разбор Методов
#### `NewLFUCache[K, V](capacity int) *LFUCache[K, V]`
- **Назначение:** Создает новый экземпляр LFU-кэша.
- **Что делает:**
    - Проверяет, что `capacity` больше нуля.
    - Инициализирует `capacity`, `minFreq` (в 0), `freqList` (новый `container/list.List`), и `cache` (новая `map`).
        

#### `updateFrequency(entry *CacheEntry[K, V])` (Вспомогательный метод)
- **Назначение:** Этот метод является "сердцем" LFU-кэша. Он вызывается каждый раз, когда элемент используется (получается или обновляется), чтобы увеличить его частоту и переместить в соответствующий `FreqNode`.
- **Что делает:**
    1. **Находит текущий `FreqNode`:** Использует `entry.parentFn` (прямая ссылка!).
    2. **Находит `*list.Element` в `freqList`:** Использует `currentFreqNode.freqListElem` (прямая ссылка!).
    3. **Удаляет `CacheEntry` из текущего `FreqNode.Entries`:** `currentFreqNode.Entries.Remove(entry.entryElem)`. Мы знаем, какой `*list.Element` нужно удалить, благодаря `entry.entryElem`.
    4. **Увеличивает частоту:** `entry.Freq++`.
    5. **Проверяет, опустел ли текущий `FreqNode`:**
        - Если `currentFreqNode.Entries.Len() == 0`:
            - Если частота этого `FreqNode` была равна `c.minFreq`, то `c.minFreq` увеличивается, так как теперь в кэше нет элементов с такой частотой (все были перемещены или удалены).
            - `currentFreqNode` удаляется из **главного списка `freqList`**: `c.freqList.Remove(currentFreqNodeElem)`.    
    6. **Находит или создает следующий `FreqNode` (для новой частоты `entry.Freq`):**
        - Он итерируется по `freqList` (начиная с места, где был `currentFreqNodeElem`) чтобы найти `FreqNode` с частотой `entry.Freq`.
        - Если такой `FreqNode` уже существует, он используется.
        - Если нет, создается новый `FreqNode`, вставляется в `freqList` в правильном месте (чтобы `freqList` оставался отсортированным по частоте), и сохраняется ссылка на себя (`newFreqNode.freqListElem`).
    7. **Добавляет `CacheEntry` в новый `FreqNode.Entries`:** `entry.entryElem = nextFreqNode.Entries.PushFront(entry)`. Новый `*list.Element` сохраняется в `entry.entryElem`.
    8. **Обновляет `parentFn`:** `entry.parentFn = nextFreqNode`. `CacheEntry` теперь принадлежит новому `FreqNode`.
        

#### `Get(key K) (V, bool)`
- **Назначение:** Извлекает значение по ключу.
- **Что делает:**
    1. Пытается найти `*CacheEntry` по `key` в `c.cache`.
    2. Если найден (`found`):
        - Вызывает `c.updateFrequency(entry)`: это увеличивает частоту элемента и перемещает его в соответствующий `FreqNode`.
        - Возвращает `entry.Value` и `true`.
    3. Если не найден:
        - Возвращает нулевое значение и `false`.
            

#### `Put(key K, value V)`
- **Назначение:** Добавляет новый элемент в кэш или обновляет существующий.
- **Что делает:**
    1. **Обработка нулевой вместимости:** Если `capacity` равно 0, просто выходит.
    2. **Обновление существующего элемента:**
        - Если `key` уже есть в `c.cache`:
            - Получает `entry`.
            - Обновляет `entry.Value = value`.
            - Вызывает `c.updateFrequency(entry)`: это обрабатывает увеличение частоты и перемещение в списке.
            - Выходит.
                
    3. **Добавление нового элемента (если `key` не найден):**
        - **Проверка переполнения:** Если `c.Size() >= c.capacity`:
            - Находит `FreqNode` с **минимальной частотой** (`c.freqList.Front()`).
            - Из этого `FreqNode` удаляется **наименее давно используемый** (`oldestEntryElem := minFreqNode.Entries.Back()`).
            - Соответствующий `CacheEntry` удаляется из `c.cache`.
            - Если `minFreqNode.Entries` опустел после удаления:
                - Удаляет `minFreqNode` из `c.freqList`.
                - Обновляет `c.minFreq` (переходит к частоте следующего `FreqNode` или 0, если кэш пуст).
        - **Создание и добавление нового `CacheEntry`:**
            - Создает новый `newEntry` с `Freq: 1`.
            - Находит или создает `FreqNode` для частоты 1 (он всегда должен быть в начале `c.freqList`).
            - Добавляет `newEntry` в `freq1Node.Entries` (в начало, т.е. LRU-поведение среди элементов с частотой 1).
            - Обновляет `newEntry.entryElem` и `newEntry.parentFn` соответствующими ссылками.
            - Сохраняет `newEntry` в `c.cache`.
            - Устанавливает `c.minFreq = 1`, так как теперь в кэше точно есть элемент с частотой 1.
                

#### `Size() int`
- **Назначение:** Возвращает текущее количество элементов в кэше.
- **Что делает:** Просто возвращает `len(c.cache)`.

#### `PrintCache()` (Вспомогательный метод)
- **Назначение:** Выводит текущее состояние кэша для отладки.
- **Что делает:** Итерируется по `freqList`, а затем по `Entries` каждого `FreqNode`, выводя ключи и значения.

```go
package main

import (
	"container/list"
	"fmt"
)

// CacheEntry представляет элемент, хранящийся в кэше.
type CacheEntry[K comparable, V any] struct {
	Key      K
	Value    V
	Freq     int                  // Частота использования этого элемента
	parentFn *FreqNode[K, V]      // Указатель на FreqNode, которому принадлежит этот CacheEntry
	entryElem *list.Element       // Указатель на *list.Element в parentFn.Entries
}

// FreqNode представляет узел в списке частот.
type FreqNode[K comparable, V any] struct {
	Frequency int              // Частота, которую представляет этот узел
	Entries   *list.List       // Двусвязный список CacheEntry с этой частотой
	freqListElem *list.Element // Указатель на этот FreqNode в главном freqList
}

// LFUCache представляет LFU-кэш.
type LFUCache[K comparable, V any] struct {
	capacity int                         // Максимальная вместимость кэша
	minFreq  int                         // Минимальная частота в кэше
	freqList *list.List                  // Главный список FreqNode, отсортированный по частоте
	cache    map[K]*CacheEntry[K, V]     // Хеш-таблица для быстрого доступа к CacheEntry по ключу
	// !!! Ключевое изменение: cache[K] теперь хранит *CacheEntry
}

// NewLFUCache создает новый экземпляр LFU-кэша.
func NewLFUCache[K comparable, V any](capacity int) *LFUCache[K, V] {
	if capacity <= 0 {
		panic("Capacity must be a positive integer")
	}
	return &LFUCache[K, V]{
		capacity: capacity,
		minFreq:  0,
		freqList: list.New(),
		cache:    make(map[K]*CacheEntry[K, V]),
	}
}

// updateFrequency обновляет частоту CacheEntry и перемещает его в соответствующий FreqNode.
func (c *LFUCache[K, V]) updateFrequency(entry *CacheEntry[K, V]) {
	// Получаем текущий FreqNode, используя parentFn из CacheEntry
	currentFreqNode := entry.parentFn
	currentFreqNodeElem := currentFreqNode.freqListElem // Указатель на FreqNode в freqList

	// Удаляем CacheEntry из текущего списка частот (currentFreqNode.Entries)
	currentFreqNode.Entries.Remove(entry.entryElem)

	// Увеличиваем частоту entry
	entry.Freq++

	// Если текущий FreqNode.Entries теперь пуст
	if currentFreqNode.Entries.Len() == 0 {
		if c.minFreq == currentFreqNode.Frequency {
			c.minFreq++ // Увеличиваем minFreq, так как этот уровень частоты опустел
		}
		c.freqList.Remove(currentFreqNodeElem) // Удаляем пустой FreqNode из главного списка
	}

	// Находим или создаем следующий FreqNode для новой частоты entry.Freq
	// nextFreqListElem - это *list.Element в c.freqList, который будет содержать новый FreqNode
	var nextFreqListElem *list.Element
	
	// Ищем узел с нужной частотой. Если его нет, создаем и вставляем.
	// Начинаем поиск с currentFreqNodeElem
	tempNodeElem := currentFreqNodeElem
	if currentFreqNode.Entries.Len() == 0 { // Если currentFreqNode был удален, начинаем с его следующего
		tempNodeElem = currentFreqNodeElem.Next()
	}

	foundNextNode := false
	for e := tempNodeElem; e != nil; e = e.Next() {
		fn := e.Value.(*FreqNode[K, V])
		if fn.Frequency == entry.Freq {
			nextFreqListElem = e
			foundNextNode = true
			break
		}
		if fn.Frequency > entry.Freq {
			// Нашли узел с большей частотой, нужно вставить перед ним
			nextFreqListElem = c.freqList.InsertBefore(&FreqNode[K, V]{Frequency: entry.Freq, Entries: list.New()}, e)
			nextFreqListElem.Value.(*FreqNode[K, V]).freqListElem = nextFreqListElem
			foundNextNode = true
			break
		}
	}

	if !foundNextNode {
		// Если не нашли узел и не вставили, значит, нужно вставить в конец freqList
		newFreqNode := &FreqNode[K, V]{Frequency: entry.Freq, Entries: list.New()}
		nextFreqListElem = c.freqList.PushBack(newFreqNode)
		newFreqNode.freqListElem = nextFreqListElem
	}
	
	nextFreqNode := nextFreqListElem.Value.(*FreqNode[K, V])

	// Добавляем entry в новый список частот (nextFreqNode.Entries)
	entry.entryElem = nextFreqNode.Entries.PushFront(entry) // Обновляем ссылку на *list.Element
	entry.parentFn = nextFreqNode                            // Обновляем родительский FreqNode
}

// Get извлекает значение по ключу.
func (c *LFUCache[K, V]) Get(key K) (V, bool) {
	entry, found := c.cache[key] // entry - это *CacheEntry
	if !found {
		var zeroValue V
		return zeroValue, false
	}

	c.updateFrequency(entry) // Обновляем частоту и перемещаем элемент
	
	return entry.Value, true
}

// Put добавляет или обновляет элемент в кэше.
func (c *LFUCache[K, V]) Put(key K, value V) {
	if c.capacity == 0 { // Если вместимость 0, ничего не можем добавить
		return
	}

	if entry, found := c.cache[key]; found {
		// Ключ уже существует, обновляем значение и "получаем" его (чтобы обновить частоту)
		entry.Value = value // Обновляем значение
		c.updateFrequency(entry) // Обновляем частоту и перемещаем элемент
		return
	}

	// Ключа нет в кэше. Проверяем вместимость.
	if c.Size() >= c.capacity {
		// Кэш полон, нужно удалить LFU элемент.
		// Находим FreqNode с минимальной частотой.
		minFreqNodeElem := c.freqList.Front()
		if minFreqNodeElem == nil {
			// Это не должно произойти, если c.Size() > 0, но для безопасности
			return
		}
		minFreqNode := minFreqNodeElem.Value.(*FreqNode[K, V])

		// Удаляем самый старый (LRU) элемент из списка наименьшей частоты
		oldestEntryElem := minFreqNode.Entries.Back()
		if oldestEntryElem != nil {
			oldestEntry := oldestEntryElem.Value.(*CacheEntry[K, V])
			minFreqNode.Entries.Remove(oldestEntryElem)
			delete(c.cache, oldestEntry.Key) // Удаляем из хеш-таблицы

			// Если FreqNode с minFreq теперь пуст, удаляем его из freqList
			if minFreqNode.Entries.Len() == 0 {
				c.freqList.Remove(minFreqNodeElem)
				// Обновляем minFreq
				if c.freqList.Len() > 0 {
					c.minFreq = c.freqList.Front().Value.(*FreqNode[K, V]).Frequency
				} else {
					c.minFreq = 0 // Кэш стал пуст
				}
			}
		}
	}

	// Добавляем новый элемент.
	// Инициализируем его частоту в 1.
	newEntry := &CacheEntry[K, V]{Key: key, Value: value, Freq: 1}

	// Находим или создаем FreqNode для частоты 1.
	// Он должен быть в начале freqList, если он вообще существует.
	var freq1NodeElem *list.Element
	if c.freqList.Front() != nil && c.freqList.Front().Value.(*FreqNode[K, V]).Frequency == 1 {
		freq1NodeElem = c.freqList.Front()
	} else {
		// Создаем новый FreqNode для частоты 1 и вставляем его в начало freqList
		newFreqNode := &FreqNode[K, V]{Frequency: 1, Entries: list.New()}
		freq1NodeElem = c.freqList.PushFront(newFreqNode)
		newFreqNode.freqListElem = freq1NodeElem // Сохраняем ссылку на себя
	}
	
	freq1Node := freq1NodeElem.Value.(*FreqNode[K, V])

	// Добавляем новый entry в список Entries этого FreqNode.
	newEntry.entryElem = freq1Node.Entries.PushFront(newEntry) // Сохраняем *list.Element в CacheEntry
	newEntry.parentFn = freq1Node                            // Сохраняем родительский FreqNode
	c.cache[key] = newEntry // Сохраняем *CacheEntry в cache

	// Устанавливаем minFreq на 1, так как только что добавили элемент с частотой 1
	c.minFreq = 1
}

// Size возвращает текущее количество элементов в кэше.
func (c *LFUCache[K, V]) Size() int {
	return len(c.cache)
}

// PrintCache выводит содержимое кэша для отладки.
func (c *LFUCache[K, V]) PrintCache() {
	fmt.Print("LFU Cache (Freq 1 -> Max Freq): ")
	for freqNodeElem := c.freqList.Front(); freqNodeElem != nil; freqNodeElem = freqNodeElem.Next() {
		freqNode := freqNodeElem.Value.(*FreqNode[K, V])
		fmt.Printf("[Freq %d: ", freqNode.Frequency)
		for entryElem := freqNode.Entries.Front(); entryElem != nil; entryElem = entryElem.Next() {
			entry := entryElem.Value.(*CacheEntry[K, V])
			fmt.Printf("%v:%v ", entry.Key, entry.Value)
		}
		fmt.Print("] ")
	}
	fmt.Println()
}

func main() {
	// Создаем LFU-кэш для строк и целых чисел с вместимостью 3
	lfuCache := NewLFUCache[string, int](3)

	fmt.Println("--- Добавление элементов ---")
	fmt.Println("Добавляем A:1")
	lfuCache.Put("A", 1) // Freq A:1
	lfuCache.PrintCache()

	fmt.Println("Добавляем B:2")
	lfuCache.Put("B", 2) // Freq B:1
	lfuCache.PrintCache()

	fmt.Println("Добавляем C:3")
	lfuCache.Put("C", 3) // Freq C:1
	lfuCache.PrintCache()
	// Cache: [Freq 1: C B A ] (порядок внутри freq node - LRU)

	fmt.Println("\n--- Использование элементов (обновление частоты) ---")
	fmt.Println("Получаем A (Freq A:2)")
	valA, foundA := lfuCache.Get("A")
	if foundA {
		fmt.Printf("Получено A: %v\n", valA)
	}
	lfuCache.PrintCache()
	// Cache: [Freq 1: C B ] [Freq 2: A ]

	fmt.Println("Получаем B (Freq B:2)")
	valB, foundB := lfuCache.Get("B")
	if foundB {
		fmt.Printf("Получено B: %v\n", valB)
	}
	lfuCache.PrintCache()
	// Cache: [Freq 1: C ] [Freq 2: B A ] (порядок внутри Freq 2 изменился, B теперь MRU)

	fmt.Println("Получаем C (Freq C:2)")
	valC, foundC := lfuCache.Get("C")
	if foundC {
		fmt.Printf("Получено C: %v\n", valC)
	}
	lfuCache.PrintCache()
	// Cache: [Freq 2: C B A ] (все элементы теперь с freq 2)

	fmt.Println("\n--- Добавление нового элемента при заполненном кэше ---")
	fmt.Println("Добавляем D:4 (вытеснит LFU - A, B или C, который был добавлен раньше)")
	// В данном случае, так как A,B,C имеют одинаковую частоту 2,
	// будет удален тот, кто был в Freq 2 дольше всех (самый LRU в этом списке) - это A.
	lfuCache.Put("D", 4)
	lfuCache.PrintCache()
	// Cache: [Freq 1: D ] [Freq 2: C B ] (A вытеснен, D добавлен с freq 1)

	fmt.Println("\n--- Проверка отсутствующего элемента ---")
	_, notFoundA := lfuCache.Get("A") 
	if !notFoundA {
		fmt.Println("A не найдено в кэше.")
	}

	fmt.Println("\n--- Обновление существующего элемента ---")
	fmt.Println("Обновляем D:5 (Freq D:2)")
	lfuCache.Put("D", 5) // D становится Freq 2
	lfuCache.PrintCache()
	// Cache: [Freq 2: D C B ]

	fmt.Println("\nТекущий размер кэша:", lfuCache.Size())
}
```

### Объяснение реализации:
