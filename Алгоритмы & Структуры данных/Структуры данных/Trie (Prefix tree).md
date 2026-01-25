
**Trie** (или **префиксное дерево**) — это древовидная структура данных, предназначенная для эффективного хранения и поиска строк по их префиксам.

Каждый узел в Trie представляет символ строки. Путь от корня до любого узла образует префикс (или полную строку), а флаг в узле указывает, является ли этот путь завершённым словом.


## Как работает Trie?

- **Вставка**: Строка разбивается на символы. Для каждого символа создаётся (или используется существующий) дочерний узел.
- **Поиск**: Поочерёдно проходим по символам строки, проверяя наличие соответствующих узлов.
- **Удаление**: Более сложная операция; требует проверки, не используются ли узлы другими словами.

Пример: Для слов `["cat", "car", "cart"]` дерево будет выглядеть так:
```go
        (root)
         /
        c
       /
      a
     / \
    t   r
         \
          t
```

## Мотивация использования Trie

- **Быстрый поиск по префиксу** (например, автозаполнение в поисковике).
- **Эффективное хранение множества строк с общими префиксами** (экономия памяти по сравнению с хеш-таблицами при большом количестве общих префиксов).
- **Проверка существования слова** за O(m), где m — длина слова (независимо от количества слов в структуре).

## Где применяется Trie?

- **Автозаполнение** (Google Search, IDE подсказки)
- **Словари и проверка орфографии**
- **Маршрутизация IP-адресов** (в сетях)
- **Игры** (например, поиск слов в Boggle)
- **Сжатие данных** (алгоритм LZW использует похожую идею)
## Преимущества Trie

- Время поиска/вставки — **O(m)**, где m — длина строки  
- Поддержка **поиска по префиксу**  
- егко реализовать **автозаполнение**, **предложения**, **проверку орфографии** 
- Не требует хеширования

## Недостатки Trie

- Может потреблять много памяти (особенно если алфавит большой, например, Unicode).
- Неэффективна для коротких или редко пересекающихся строк.

## ## Полная реализация Trie на Go
```go
package main

// TrieNode — узел дерева
type TrieNode struct {
	children map[rune]*TrieNode
	isEnd    bool
}

// NewTrieNode создаёт новый узел
func NewTrieNode() *TrieNode {
	return &TrieNode{
		children: make(map[rune]*TrieNode),
		isEnd:    false,
	}
}

// Trie — структура дерева
type Trie struct {
	root *TrieNode
}

// NewTrie создаёт новое дерево
func NewTrie() *Trie {
	return &Trie{root: NewTrieNode()}
}

// Insert добавляет слово в дерево
func (t *Trie) Insert(word string) {
	node := t.root
	for _, ch := range word {
		if _, exists := node.children[ch]; !exists {
			node.children[ch] = NewTrieNode()
		}
		node = node.children[ch]
	}
	node.isEnd = true
}

// Search проверяет, существует ли слово целиком
func (t *Trie) Search(word string) bool {
	node := t.root
	for _, ch := range word {
		if _, exists := node.children[ch]; !exists {
			return false
		}
		node = node.children[ch]
	}
	return node.isEnd
}

// StartsWith проверяет наличие хотя бы одного слова с данным префиксом
func (t *Trie) StartsWith(prefix string) bool {
	node := t.root
	for _, ch := range prefix {
		if _, exists := node.children[ch]; !exists {
			return false
		}
		node = node.children[ch]
	}
	return true
}

// Delete удаляет слово из дерева (если оно существует)
// Возвращает true, если слово было удалено
func (t *Trie) Delete(word string) bool {
	if !t.Search(word) {
		return false // Слово не существует
	}
	t.deleteHelper(t.root, word, 0)
	return true
}

// deleteHelper — рекурсивная вспомогательная функция
func (t *Trie) deleteHelper(node *TrieNode, word string, index int) bool {
	if index == len(word) {
		// Мы на последнем символе
		if !node.isEnd {
			return false // Не должно произойти при корректном вызове
		}
		node.isEnd = false
		// Можно ли удалить этот узел? Только если нет потомков.
		return len(node.children) == 0
	}

	ch := rune(word[index])
	child := node.children[ch]

	shouldDeleteChild := t.deleteHelper(child, word, index+1)

	if shouldDeleteChild {
		delete(node.children, ch)
		// Удаляем текущий узел, только если он не конец другого слова и не имеет других детей
		return !node.isEnd && len(node.children) == 0
	}

	return false
}

// GetAllWordsWithPrefix возвращает все слова, начинающиеся с prefix
func (t *Trie) GetAllWordsWithPrefix(prefix string) []string {
	node := t.root
	for _, ch := range prefix {
		if _, exists := node.children[ch]; !exists {
			return nil // Префикс не найден
		}
		node = node.children[ch]
	}

	var results []string
	t.collectWords(node, prefix, &results)
	return results
}

// collectWords — рекурсивный обход поддерева для сбора слов
func (t *Trie) collectWords(node *TrieNode, current string, results *[]string) {
	if node.isEnd {
		*results = append(*results, current)
	}
	for ch, child := range node.children {
		t.collectWords(child, current+string(ch), results)
	}
}
```
