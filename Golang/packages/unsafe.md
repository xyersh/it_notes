### Что это такое?
Пакет `unsafe` в Go, как следует из его названия, предоставляет функциональность, которая выходит за рамки обычных правил безопасности типов в Go. Он позволяет вам выполнять низкоуровневые операции, такие как прямой доступ к памяти и преобразование типов, которые обычно запрещены в Go. Хотя `unsafe` может быть очень мощным инструментом для решения определенных задач, он также является источником потенциальных ошибок, если используется неправильно.

### Зачем нужен unsafe?
Пакет `unsafe` используется в очень специфических сценариях, где производительность является критически важной, или когда необходимо взаимодействовать с низкоуровневыми системами или сторонними библиотеками на C. Вот некоторые из практических задач, которые можно решать с помощью `unsafe`:

1. **Оптимизация производительности:** В редких случаях, когда стандартные операции Go оказываются недостаточно быстрыми, `unsafe` может быть использован для прямого манипулирования памятью, что потенциально может ускорить операции, например, при сериализации/десериализации данных или работе с большими массивами.
2. **Взаимодействие с кодом на C (FFI):** При работе с C-библиотеками через `cgo`, `unsafe` может быть необходим для преобразования типов Go в типы C и наоборот, а также для доступа к памяти, выделенной C.
3. **Реализация специализированных структур данных:** Иногда для очень специфических структур данных, которые требуют точного контроля над макетом памяти (например, для кэшей или пулов объектов), `unsafe` может быть использован для создания более эффективных реализаций.
4. **Рефлексия на низком уровне:** Хотя Go имеет свой собственный пакет `reflect`, `unsafe` может быть использован для более глубокого, но и более опасного, анализа и манипуляции типами и значениями во время выполнения.
5. **Доступ к неэкспортированным полям структур:** В очень редких случаях, когда необходимо получить доступ к неэкспортированным (приватным) полям структур из сторонних пакетов (чего обычно не следует делать!), `unsafe` может предоставить такую возможность.

Пакет `unsafe` предоставляет следующие ключевые элементы:
- `unsafe.Pointer`: Это универсальный указатель, который может указывать на любой тип в Go. Он аналогичен `void*` в C. `Pointer` не является типизированным, что означает, что компилятор не будет проверять его на безопасность типов.
- `unsafe.Alignof(v T) uintptr`: Возвращает выравнивание памяти для переменной `v` (в байтах).
- `unsafe.Offsetof(f T) uintptr`: Возвращает смещение поля `f` внутри структуры (в байтах).
- `unsafe.Sizeof(v T) uintptr`: Возвращает размер в байтах, который занимает переменная `v` в памяти (в байтах).
 - `unsafe.String(ptr *byte, len int) string`: Создает строку из указателя на байты и заданной длины. `unsafe.String` берет указатель на начальный байт (`*byte`) и длину (`int`) и конструирует новую строку, которая _ссылается на те же байты_ без их копирования.
 - `unsafe.StringData(str string) *byte`: Возвращает указатель на первый байт базового массива данных строки. `unsafe.StringData` предоставляет прямой доступ к внутренней памяти, где хранятся байты строки.
 - `unsafe.Slice(ptr *ArbitraryType, len int) []ArbitraryType`: Создает срез из указателя на первый элемент и заданной длины.  `unsafe.Slice` берет указатель на первый элемент (`ptr`) любого типа `ArbitraryType` и длину (`len`) и конструирует новый срез, который _ссылается на те же данные_ без копирования. Это обобщенная версия создания среза из массива или другой памяти.
 - `unsafe.SliceData(slice []ArbitraryType) *ArbitraryType`: Возвращает указатель на первый элемент базового массива среза. предоставляет прямой доступ к внутренней памяти, где хранятся элементы среза.
 - ``
#### аксиомы unsafe.Pointer
- Любой тип указателя `*T` может быть преобразован в `unsafe.Pointer`.
```go
	x := 123
	p := unsafe.Pointer(&x)
```
- `unsafe.Pointer` может быть преобразован в любой тип указателя `*T`.
 ```go
	newPtr := (*int)(p)
```
- `uintptr` может быть преобразован в `unsafe.Pointer`.
- `unsafe.Pointer` может быть преобразован в `uintptr`.



### Примеры

#### Значение  из unsafe.Pointer
`unsafe.Pointer` - можно преобразовать в любой \*ТИП
```go
	x:= uint(23423432)
	y := *(*int)(unsafe.Pointer(&x)) 
```

#### Размер переменной
`unsafe.Sizeof()` - получить размер переменной в байтах 
```go
	v := 111.555
	fmt.Printf("%d", unsafe.Sizeof(v))
```

#### Значение выравнивания структуры
`unsafe.Alignof()` - указывает по какому числу байт идет выравнивание структуры (выравнивание проводится по самому длинному полю структуры)
```go
type MyStruct struct {
	Field1 byte
	Field2 int64
	Field3 byte
	Field4 byte
}

s := MyStruct{}
fmt.Printf("%d", unsafe.Alignof(s)) //8 байт (по полю Field2)
```


#### Значение смещения полей структуры
`unsafe.Offsetof()` - показывает смещение в байтах поля структуры относительно начального байта:
```go
type MyStruct struct {
	Field1 byte    // 0 байт смещение
	Field2 int64   //8 + 0 байт смещение
	Field3 byte    //8 + 8 = 16 байт смещение
	Field4 byte    // 16 + 1 = 17 байт смещение 
}

s := MyStruct{}
fmt.Printf("%d", unsafe.Offsetof(s.Field3)) // 16 байт 
fmt.Printf("%d", unsafe.Offsetof(s.Field4)) // 17 байт
```



#### Получение значений элементов массива с помощью адресной арифметики
Зная размер смещений мы можем получать определенные  значения, используя адресную арифметику:
```go
package main  
  
import (  
    "fmt"  
    "unsafe")  
  
func main() {  
    arr := [4]byte{0x00, 0x01, 0x02, 0x03}  
    arr_0 := (*byte)(unsafe.Pointer(uintptr(unsafe.Pointer(&arr)) + 0*unsafe.Sizeof(arr[0])))  
    arr_1 := (*byte)(unsafe.Pointer(uintptr(unsafe.Pointer(&arr)) + 1*unsafe.Sizeof(arr[0])))  
    arr_2 := (*byte)(unsafe.Pointer(uintptr(unsafe.Pointer(&arr)) + 2*unsafe.Sizeof(arr[0])))  
    arr_3 := (*byte)(unsafe.Pointer(uintptr(unsafe.Pointer(&arr)) + 3*unsafe.Sizeof(arr[0])))  
  
    fmt.Printf("arr[0] = %d\n", *arr_0)  
    fmt.Printf("arr[1] = %d\n", *arr_1)  
    fmt.Printf("arr[2] = %d\n", *arr_2)  
    fmt.Printf("arr[3] = %d\n", *arr_3)  
}
```

#### Ковыряем слайс при помощи unsafe
```go
package main  
  
import (  
    "fmt"  
    "unsafe")  
  
func main() {  
    s := make([]int, 10, 20)  
    s[4] = 4  
    s[5] = 5  
    fmt.Println(s)                         // [0 0 0 0 4 5 0 0 0 0]  
    fmt.Printf("addr of s: %p\n", &s)     // 0xc0000a8018  
    fmt.Printf("addr of s[0]: %p\n", &s[0])    // 0xc0000b2000  
    fmt.Printf("addr of s[1]: %p\n", &s[1])    // 0xc0000b2008  
    fmt.Printf("addr of s[2]: %p\n", &s[2])  // 0xc0000b2010  
  
    fmt.Println("\nget slice's info via unsafe:")  
    fmt.Printf("size of s: %d bytes\n", unsafe.Sizeof(s)) // size of s: 24 bytes  
    s_ptr := unsafe.Pointer(uintptr(unsafe.Pointer(&s)) + unsafe.Alignof(s)*0)  
    s_len := (*int)(unsafe.Pointer(uintptr(s_ptr) + unsafe.Alignof(s)*1))  
    s_cap := (*int)(unsafe.Pointer(uintptr(s_ptr) + unsafe.Alignof(s)*2))  
  
    fmt.Printf("s_ptr: %v value: %x\n", s_ptr, *(*[]int)(s_ptr)) // s_ptr: 0xc0000a8018 value: [0 0 0 0 4 5 0 0 0 0]  
    fmt.Printf("s_len: %v value: %v\n", s_len, *s_len) // s_len: 0xc0000a8020 value: 10 
    fmt.Printf("s_cap: %v value: %v\n", s_cap, *s_cap) // s_cap: 0xc0000a8028 value: 20 
    fmt.Printf("")  
}
```


#### Применение Sizeof, Offsetof, Alignof на примере работы со структурами
```go
package main

import (
	"fmt"
	"unsafe"
)

type MyStruct1 struct {
	Field1 byte
	Field2 int64
	Field3 byte
	Field4 byte
}

type MyStruct2 struct {
	Field1 int64
	Field2 byte
	Field3 byte
	Field4 byte
}

func main() {
	n := -123
	fmt.Printf("var n: Type = %[1]T,  value = %[1]v\n", n)
	fmt.Printf("var &n: Type = %[1]T,  value = %[1]v\n", &n)

	//получение unsafe.Pointer
	p := unsafe.Pointer(&n)
	fmt.Printf("unsafe ptr: Type = %[1]T,  value = %[1]v\n", p)

	//десятичное значение указателя
	p2 := uintptr(p)
	fmt.Printf("uintptr : Type = %[1]T,  value = %[1]v\n", p2)

	//новое число после преобразования unsafe.Pointer
	n2 := *(*uint)(p)
	fmt.Printf("var n2: Type = %[1]T,  value = %[1]v\n", n2)

	arr1 := [4]byte{}
	fmt.Printf("var arr1: Type = %[1]T,  value = %[1]v\n", arr1)
	fmt.Printf("Size arr1: %d bytes\n", unsafe.Sizeof(arr1))

	arr2 := []byte{0x00, 0x01, 0x02, 0x03, 0x04, 0x05, 0x06}
	fmt.Printf("var arr2: Type = %[1]T,  value = %[1]v\n", arr2)
	fmt.Printf("Size arr2: %d bytes\n", unsafe.Sizeof(arr2))

	s1 := MyStruct1{}
	fmt.Printf("Sizeof s1 = %d  alignof = %d \n", unsafe.Sizeof(s1), unsafe.Alignof(s1))
	fmt.Printf("offset of s1.Field1 = %d\n", unsafe.Offsetof(s1.Field1))
	fmt.Printf("offset of s1.Field2 = %d\n", unsafe.Offsetof(s1.Field2))
	fmt.Printf("offset of s1.Field3 = %d\n", unsafe.Offsetof(s1.Field3))
	fmt.Printf("offset of s1.Field4 = %d\n", unsafe.Offsetof(s1.Field4))

	s2 := MyStruct2{}
	fmt.Printf("Sizeof s2 = %d  alignof = %d\n", unsafe.Sizeof(s2), unsafe.Alignof(s2))
	fmt.Printf("offset of s2.Field1 = %d\n", unsafe.Offsetof(s2.Field1))
	fmt.Printf("offset of s2.Field2 = %d\n", unsafe.Offsetof(s2.Field2))
	fmt.Printf("offset of s2.Field3 = %d\n", unsafe.Offsetof(s2.Field3))
	fmt.Printf("offset of s2.Field4 = %d\n", unsafe.Offsetof(s2.Field4))
}
```

#### преобразование слайса байт в объект структуры, и наоборот
```go
package main  
  
import (  
    "bytes"  
    "fmt"    "unsafe")  
  
type ModbusHeader struct {  
    TransactionID uint16  
    ProtocolID    uint16  
    Length        uint16  
    DeviceAddress uint8  
    FunctionCode  uint8  
    ByteCount     uint8  
}  
  
func ModbusPDU(bytes []byte) (ModbusHeader, []uint16) {  
    // header  
    header := *(*ModbusHeader)(unsafe.Pointer(&bytes[0]))  
  
    //registers  
    offset := unsafe.Offsetof(header.ByteCount) + unsafe.Sizeof(header.ByteCount)  
    regAddr := unsafe.Pointer(uintptr(unsafe.Pointer(&bytes[0])) + offset)  
    bytesAddr := (*uint16)(regAddr)  
    registers := unsafe.Slice(bytesAddr, int(header.ByteCount)/2)  
  
    return header, registers  
}  
  
func BytesFromModbus(pdu ModbusHeader, registers []uint16) []byte {  
    if len(registers) < 1 {  
       return nil  
    }  
    headerSize := unsafe.Offsetof(pdu.ByteCount) + unsafe.Sizeof(pdu.ByteCount)  
    addrH := (*byte)(unsafe.Pointer(&pdu))  
    bytesHeader := unsafe.Slice(addrH, headerSize)  
  
    addrR := (*byte)(unsafe.Pointer(&registers[0]))  
    bytesRegisters := unsafe.Slice(addrR, pdu.ByteCount*uint8(unsafe.Sizeof(registers[0]))/2)  
  
    fmt.Printf("bytesHeader: %v \n", bytesHeader)  
    fmt.Printf("bytesRegisters: %v \n", bytesRegisters)  
  
    return append(bytesHeader, bytesRegisters...)  
}  
  
func main() {  
    msg := []byte{  
       0x00, 0x00, //Transaction ID  
       0x01, 0x00, //Protocol ID  
       0x09, 0x00, //Length  
       0x00,                                //Device address  
       0x03,                                //Function code  
       0x06,                                //Bytes count  
       0x0B, 0x02, 0x064, 0x00, 0x7F, 0x00, //Data registers  
    }  
    fmt.Println("msg:")  
    fmt.Println(msg)  
  
    Header, Registers := ModbusPDU(msg)  
    fmt.Printf("%+v \n %#b \n", Header, Registers)  
    fmt.Printf("size of header = %d \n", unsafe.Sizeof(Header))  
    fmt.Printf("offset of header.Bytecount = %d \n", unsafe.Offsetof(Header.ByteCount))  
    fmt.Printf("registers to bytes: %b \n", *(*[]byte)(unsafe.Pointer(&Registers)))  
  
    msg2 := BytesFromModbus(Header, Registers)  
    fmt.Println("msg2:")  
    fmt.Println(msg2)  
    fmt.Printf("msg is equal to  msg2:  %t", bytes.Equal(msg, msg2))  
}
```
