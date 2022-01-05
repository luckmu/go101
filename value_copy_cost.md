# value copy cost

值的尺寸只和直接部分（direct part）有关，简介部分（underlying part）没有贡献。

值复制的成本

in action 将 size 不大于4个原生字且字段数不超过4个的结构体看作是小尺寸值。复制小尺寸值的代价是比较小的。

对于标准编译器，除了大尺寸的结构体和数组类型，其他类型均为小尺寸类型。

in action 很少使用基类型为 slice、map、channel、function、string、interface 的指针类型，因为复制这些值的代价很小

```go
type S [12]int64
var sX = make([]S, 1000)
var sY = make([]s, 1000)
var sZ = make([]s, 1000)
var sumX, sumY, sumZ int64
// 2708 ns/op
for i := 0; i < b.N; i++ {
    sumX = 0
    for j := 0; j < len(sX); j++ {
        sumX += sX[j][0]
    }
}
// 2808 ns/op
for i := 0; i < b.N; i++ {
    sumY = 0
    for j := range sY {
        sumY += sY[j][0]
    }
}
// 5222 ns/op
for i := 0; i < b.N; i++ {
    sumZ = 0
    for _, v := range sZ {
        sumZ += v[0]
    }
}
```
