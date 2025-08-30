

### Параллельное программирование
#### основы 
- как происходит выполнение кода (регистры, стэки)
- Единицы исполнения кода: процесс, поток, корутина
- многозадачность: кооперативная, вытесняющая

#### операционные системы
- прерывания
- context switching
- виртуальная память
- адресное пространство процесса
- сегменты процесса: stack, heap, text, data/bss

#### Базовые примитивы синхронизации
- мьютексы
- семафоры
- условные переменные
Практика:
- написать приложение, которое будет параллельно работать с какой-нибудь структурой данных
- Написать приложение, которое будет ограничивать кол-во одновременных соединений для сервера
- написать приложение producer-consumer (reader-writer)

#### Базовая архитектура компьютера
- иерархия памяти
- разрядность процессора
- разрядность шины


#### Атомики
- Load
- Store
- CompareAndSwap (рассмотреть как паттерн)
- Атомики с указателями
Практика:
- написать приложение, которое будет считать RPS сервера
- Написать spin-lock

#### Проблемы параллельного программирования
- Data race
- Dead lock
- Live lock
- Starvation

Практика:
- Решить задачу об обедающих философах
- Решить задачу о спящем парикмахере
- Решить задачу о курильщиках

#### Продвинутые примитивы синхронизации
- Каналы 
- Мьютексы: read-write, recursive, timed, hierarchical
- Futures
- Promises
Практика:
- написать read-write мьютекс
- написать recursive мьютекс
- написать timed мьютекс
- написать hierarchical мьютекс

#### Паттерны
- Worker Pool
- Sheduler
- Batcher
Практика:
- Написать пул воркеров которые параллельно будут выполнять какие-то задачи
- написать планировщик задач, например setTimeout, setInterval из Js
- написать batcher

#### Ввод-вывод
- синхронный ввод-вывод
- мультиплексированный ввод-вывод
- асинхронный ввод вывод
Практика:
- написать TCP-сервер на epoll-ах

#### Барьеры памяти
- write-барьер
- Read-барьер
- Acquire-барьер
- Release-барьер

#### Различные внутренности
- Кэши процессора:
	- когерентность кэша (MESI)
	- Store buffer
	- Invalidation queue
	- False sharing
- Pipeline процессора
- Hyper-threading
- Алгоритмы планирования:
	- FIrst come First serve
	- Shortest job next
	- Round Robin
	- Приоритетное планирование
- Инверсия приоритетов
- Закон Амдала

#### Алгоритмы синхронизации
- Грубая синхронизация
- Тонкая синхронизация
- Оптимистичная синхронизация
- Ленивая синхронизация
- Партиционирование
Практика:
- написать несколько реализаций связанного списка используя грубую, тонкую, оптимистичную и ленивую синхронизацию
- написать партиционирование хэш-таблицы

#### Lock-free структуры
- ABBA проблема
- Стэк Трайбера
- Очередь Майкла-Скотта

#### Wait-free структуры данных



### Алгоритмы

#### Асимптотический анализ
 - Верхняя оценка (O-j)
 - Усредненная оценка (Ω-омега)
 - Нижняя оценка (ʘ- тета)

#### Базовые структуры данных
- Массив
- Список
- Двухсвязный список
- Деревья 
- Хэш-таблица
- Бинарная куча
- Очередь
- Двухсторонняя очередь
- Стек
- Граф

#### Бинарный поиск 
- first bad version
- valid perfect square
- search insert position
- sqrt(x)
- search in rotated sorted array
- peak index in a mountain array
- find first and last position of elements

#### Задачи
##### Два указателя
- remote duplecates from sortet array
- merge sorted array
- intersection of two arrays II
- two SUM
- 3Sum
- 4Sum
- sort colors
- move zeroes
- partition labels

##### Строки 
- Roman to integer
- Integer to roman
- Valid palindrome
- Valid anagram
- Reverse string 
- Zigzag conversion
- length of last word

##### Связанные списки
- Middle of the linked list
- Linkedlist cycle
- remote duplicatesfrom sorted list
- reverse linked list
- palindrome linked list
- merge two sorted lists
- merge K sorted lists
- DElete node in a linked list
- add two numbers
- sort list

##### Деревья
 - binary tree inorder traversal
 - binary tree preorder traversal
 - binary tree postorder traversal
 - binaty tree level order traversal
 - maximum depth of binary tree
 - minimum depth of binaty tree
 - path sum
 - symmetric tree
 - same tree
 - binary tree paths
 - validate binary search tree
 - Lowest common ancestor of a binary tree
 - binary tree right side view

##### Хэш-таблицы
- intersection of two arrays
- two sum
- isomorphic strings
- word pattern
- valid sudoku
- group anagrams
- LRU cache
- Top K frequent elements

##### Матрицы
- matrix diagonal sum
- valid sudoku
- 01 matrix
- Maximal square
- set matrix zeroes
- spiral matrix
- rotate image

##### Очередь и стек
- valid parentheses
- min stack
- daily temperatures
- implement queue using stack
- implement stack using queues
- minimum remove to make valid parentheses

##### Битовые операции 
- power of two
- poser of four
- counting bits
- reverse bits
- hamming distance
- total hamming distance

##### Скользящие окна
- longest substring withoutrepeating characters
- longest repeatingcharacter replacement
- fruit into backets
- find all anagrams in a string
- sliding window maximum

##### Поиск с возвратом
- permutations
- combination sum
- subsets
- generate parentheses 
- remove invalid parentheses
- N-Queens
- N-Queens II

##### Дополнительно
- DFS/BFS
- Префиксное дерево
- Суффиксное дерево
- Quad-дерево
- Дерево отрезков
- Система непересекающихся множеств
- Алгоритмы Ли, Дейкстры, Флойда-Уоршелла
- Топологическая сортировка
- Алгоритм кадане
- Алгоритм Кнута-Морриса-Пратта
- Динамическое программирование
- Quick select
- Дерево МЕркла
- Персистентные структуры данных
- Фильтр Блума
- Битовая карта 
- B/B+ дерево
- LSM дерево
- Splay дерево
- Жадные алгоритмы
- Кольцевой буффер
- Фибоначчиева куча




###  Бэкенд

#### Базы данных 

##### Виды БД
- Реляционные 
- Документоориентированные
- key-value
- временных рядов
- колоночные 
- blob store
- графовые

##### Индексы
- Hash
- Btree
- Spatial
- Bitmap
- Inverted

##### Транзакции
- ACID 
- BASE
- Уровни изоляции транзакций

##### SQL
- DML
- DDL

##### [[OLAP vs OLTP|OLAP/OLTP]]

#### Балансировка нагрузки
- Round Robin
- Weighted Round Robin
- Least Connections
- Sticky sessions

#### Проксирование
- Reverse proxy
- Forward proxy

#### Кэширование
##### Типы
- локальное кэширование
- внешнее кэширование
##### Виды
- Lazy caching
- Write-through
- Read-through
- Write-around
##### Алгоритмы вытеснения
- LRU, SLRU
- MRU
- LFU
- FIFO
- LIFO
- 2Q
- MQ
##### Тегирование и версионирование  кэша
#### Observability
- Логирование 
- мониторинг
- трейсинг
- профайлинг
  
#### Паттерны и подходы системного дизайна
- Шардирование: вертикальное горизонтальное
- CQRS
- backoff
- pub/sub
- circuit breaker
- gracefull degradation
- polling and streaming
- mapreduce
- servless
- trottling
- backpressure
- ##### Репликация
	 - Подходы: блочная, физическая, логическая
	 - Виды: синхронная, асинхронная
	 - Типы: с одним ведущим узлом, с несколькими ведущими узлами, без ведущих узлов
 - Толстый клиент
 - Идемпотентность

#### Алгоритмы
- Выбор лидера
- Распределенная блокировка
- Распределенная транзакция
- Консистентное хэширование
- rate limiting
- консенсус
- деплой

#### Архитектуры
- файл-сервер, клиент-сервер
- Монолитная/микросервисная
- Трехзвенная
- Событийно-ориентированная архитектура: 
	- Event notification
	- State transfer
	- Event collaboration

#### Инструменты
- Linux
- Docker
- Kubernates

#### Другое
- Брокеры сообщений
-  CAP-теорема
- Latency, availability, throughput
- SLA,  SLO, SLI
- API (SOAP, REST, gRPC, GraghQL)
- Масштабирование: горизонтальное, вертикальное
- Идентификация, аутентификация, авторизация