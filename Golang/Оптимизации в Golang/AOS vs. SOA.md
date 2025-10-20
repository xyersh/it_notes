## Структуры данных: AOS vs. SOA

**Тезис:** При работе с коллекцией структур данных, часто более эффективным с точки зрения кэша является подход **Struct of Arrays (SOA)**, чем классический **Array of Structs (AOS)**, особенно если в цикле используется только подмножество полей структуры.
- **AOS (Array of Structs):** `[]struct { X, Y, Z float64 }`. Поля `X`, `Y`, `Z` хранятся вперемешку.
- **SOA (Struct of Arrays):** `struct { X, Y, Z []float64 }`. Поля каждого типа хранятся последовательно.
    
**Пример кода (AOS vs. SOA):**
```go
// ❌ Array of Structs (AOS) - Менее кэш-дружелюбно для частичного доступа
type PointAOS struct {
    X, Y, Z float64 // 24 байта. Если цикл использует только X, то Y и Z
}                   // бесполезно занимают место в кэш-линии.

// ✅ Struct of Arrays (SOA) - Более кэш-дружелюбно
type PointsSOA struct {
    X []float64
    Y []float64
    Z []float64
}

// Пример использования SOA:
func processX(points PointsSOA) float64 {
    var sum float64
    // При итерации здесь мы читаем только массив X, который полностью
    // последователен в памяти.
    for _, x := range points.X {
        sum += x
    }
    return sum
}
```