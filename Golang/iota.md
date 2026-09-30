`iota` — предопределённый счетчик который используется внутри блока **const**
Свойства `iota`:
- начинается с нуля
- в рамках одного блока **const** можно использовать только 1 явный iota. все остальные константы при этом приобретают значения  с учетом плавающего инкремента на единицу ()
- в каждом новом блоке **const** `iota` обнуляется

> Главное правило: **`iota` живёт только внутри блока `const`** и обнуляется при его открытии. Всё остальное — комбинация этого факта с арифметикой и побитовыми операциями.

## Примеры  использования iota

### Базовая нумерация (замена хардкода)
```go
const (
    StatusNew = iota      // 0
    StatusActive          // 1
    StatusArchived        // 2
    StatusDeleted         // 3
)
```

### Пропуск значений через `_`
```go
const (
    _           = iota // 0 — пропущено
    PriorityLow        // 1
    PriorityMedium     // 2
    _                  // 3 — пропущено
    PriorityHigh       // 4
    PriorityCritical   // 5
)
```

### Побитовые флаги (bitmask)
Один из самых мощных приёмов — каждый флаг занимает свой бит:
```go
const (
    PermRead    = 1 << iota // 1  (0b0001)
    PermWrite               // 2  (0b0010)
    PermExecute             // 4  (0b0100)
    PermAdmin               // 8  (0b1000)
)

// Комбинирование:
var role = PermRead | PermWrite // 3 (чтение + запись)

func hasPermission(perm, flag int) bool {
    return perm&flag != 0
}
```

### Единицы измерения (степени 1024)
```go
const (
    _  = iota             // 0 — пропускаем
    KB = 1 << (10 * iota) // 1 << 10 = 1024
    MB                    // 1 << 20 = 1 048 576
    GB                    // 1 << 30 = 1 073 741 824
    TB                    // 1 << 40 = 1 099 511 627 776
)

func main() {
    fmt.Printf("1 GB = %d байт\n", 1*GB)
}
```

### Type-safe enum (кастомный тип + Stringer)
Предотвращает случайное присваивание «чужих» чисел
```go
type Color int

const (
    Red Color = iota
    Green
    Blue
)

func (c Color) String() string {
    return [...]string{"Red", "Green", "Blue"}[c]
}

func main() {
    var c Color = Green
    fmt.Println(c) // Green
}
```

### Валидация enum (Min / Max / Count)
```go
type Weekday int

const (
    Monday Weekday = iota
    Tuesday
    Wednesday
    Thursday
    Friday
    Saturday
    Sunday
    weekdayCount // 7 — не экспортируется, служит счётчиком
)

func IsValid(w Weekday) bool {
    return w >= Monday && w < weekdayCount
}
```

### Арифметические выражения с `iota`
```go
const (
    Even0 = iota * 2 // 0
    Even2            // 2
    Even4            // 4
    Even6            // 6
)

const (
    Offset10 = iota*10 + 10 // 10
    Offset20                // 20
    Offset30                // 30
)
```

### Повторение последнего выражения (без `iota`)
Если в строке нет `iota`, Go **повторяет выражение** из предыдущей строки, но `iota` внутри него всё равно растёт:
```go
const (
    A = iota + 1 // 1  (выражение: iota + 1)
    B            // 2  (повтор: iota + 1, iota=1)
    C            // 3
    D = 100      // 100 (явное значение, iota=3, но не используется)
    E            // 100 (повтор выражения "100", iota=4)
    F = iota     // 5  (iota возобновляет счёт)
)
```

### Несколько «столбцов» в одном блоке
```go
const (
    Read, Write   = 1 << iota, 1 << iota // Read=1,  Write=1  ← оба 1<<0
    Read2, Write2                        // Read2=2, Write2=2 ← оба 1<<1
)
```
> ⚠️ Это **редкий и запутанный** паттерн. На практике лучше разнести по разным строкам.

### Сброс `iota` в новом блоке
```go
const (
    X = iota // 0
    Y        // 1
)

const (
    A = iota // 0 ← заново!
    B        // 1
)
```

### HTTP-статусы / коды ошибок с шагом
```go
const (
    ErrBadRequest    = iota*100 + 400 // 400
    ErrUnauthorized                   // 500 ← iota=1 → 500? Нет!
)
// Правильнее для HTTP — явные значения, но для внутренних кодов:

const (
    InternalErrBase = iota*10 + 1000 // 1000
    InternalErrDB                    // 1010
    InternalErrCache                 // 1020
    InternalErrQueue                 // 1030
)
```

### Матрица прав доступа (комбинация bitmask + iota)
```go
type Role uint8

const (
    RoleGuest Role = 1 << iota // 1
    RoleUser                   // 2
    RoleModerator              // 4
    RoleAdmin                  // 8
)

type Permission uint8

const (
    CanView Permission = 1 << iota
    CanEdit
    CanDelete
    CanManageUsers
)

// Таблица: роль → набор разрешений
var rolePermissions = map[Role]Permission{
    RoleGuest:     CanView,
    RoleUser:      CanView | CanEdit,
    RoleModerator: CanView | CanEdit | CanDelete,
    RoleAdmin:     CanView | CanEdit | CanDelete | CanManageUsers,
}
```

### Генерация `String()` через `go generate` (паттерн)
```go
//go:generate stringer -type=Direction

type Direction int

const (
    North Direction = iota
    East
    South
    West
)
// После `go generate` автоматически создастся файл direction_string.go
```