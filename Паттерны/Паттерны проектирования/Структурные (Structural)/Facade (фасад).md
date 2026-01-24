Это структурный паттерн, который предоставляет **упрощённый интерфейс** к сложной подсистеме. Он **скрывает внутреннюю сложность**, позволяя клиенту взаимодействовать с системой через один понятный метод или API.

## Когда использовать
- У вас есть **сложная система** из множества взаимосвязанных классов/компонентов.
- Вы хотите **ограничить прямой доступ** клиента к деталям реализации подсистемы.
- Нужно **инкапсулировать инициализацию и взаимодействие** между компонентами подсистемы.

## Цель

- Упростить использование системы для клиента.
- Снизить связность между клиентом и подсистемой.
- Предоставить высокоуровневый интерфейс вместо низкоуровневых вызовов.

## Пример

Допустим, имеется система для запуска компьютера: BIOS, CPU, Memory, HardDrive. Каждый компонент имеет свой метод инициализации. Вместо того чтобы заставлять клиента вызывать их в правильном порядке, создаём фасад `Computer`.
Данный пример касается только упрощения инициализации целевого объекта. Однако данный паттерн можно использовать получения более расширенного функционала `Computer` на основании имеющихся у него компонентов.
```go
package main

import "fmt"

// Подсистемы
type BIOS struct{}
func (b *BIOS) Initialize() { fmt.Println("BIOS: initializing") }

type CPU struct{}
func (c *CPU) Freeze()   { fmt.Println("CPU: freeze") }
func (c *CPU) Jump(addr int) { fmt.Println("CPU: jump to", addr) }
func (c *CPU) Execute()  { fmt.Println("CPU: execute") }

type Memory struct{}
func (m *Memory) Load(addr int, data []byte) {
    fmt.Printf("Memory: load %d bytes at address %d\n", len(data), addr)
}

type HardDrive struct{}
func (h *HardDrive) Read(addr int, size int) []byte {
    fmt.Printf("HardDrive: read %d bytes from address %d\n", size, addr)
    return make([]byte, size)
}

// Фасад
type Computer struct {
    bios       *BIOS
    cpu        *CPU
    memory     *Memory
    hardDrive  *HardDrive
}

func NewComputer() *Computer {
    return &Computer{
        bios:      &BIOS{},
        cpu:       &CPU{},
        memory:    &Memory{},
        hardDrive: &HardDrive{},
    }
}

func (c *Computer) Start() {
    c.bios.Initialize()
    c.cpu.Freeze()
    data := c.hardDrive.Read(0, 1024)
    c.memory.Load(0, data)
    c.cpu.Jump(0)
    c.cpu.Execute()
}

// Клиент
func main() {
    pc := NewComputer()
    pc.Start() // один вызов вместо цепочки
}
```

