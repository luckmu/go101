# memory layout

## Go 中类型对齐保证 (alignment guarantee)

字段对齐保证 & 一般对齐保证

编译时刻估值
+ unsafe.Alignof(t)
+ unsafe.Alignof(x.t)

运行时刻估值
+ reflect.TypeOf(t).Align()
+ reflect.TypeOf(t).FieldAlign()

[effective go size and alignment guarantees](https://golang.google.cn/ref/spec#Size_and_alignment_guarantees)
1. 任何类型 x, unsafe.Alignof(x) 的最小结果为 1
2. 结构体变量 x, unsafe.Alignof(x) 的结果为 x 的所有字段的对齐保证 unsafe.Alignof(x.f) 中的最大值 (但是最小为 1)
3. 对于一个数组类型的变量 x, unsafe.Alignof(x) 的结果和此数组的元素类型的一个变量的对齐保证相等

go 1.17
|types|guarantees|
|:-:|:-:|
|bool, uint8, int8|1|
|uint16, int16|2|
|uint32, int32|4|
|float32, complex64|4|
|array|element type|
|struct|field types|
|other types|size of a native word|

## 类型的尺寸和结构体字节填充 (structure padding)

|type|size (bytes)|
|:-:|:-:|
|uint8, int8|1|
|uint16, int16|2|
|uint32, int32, float32|4|
|uint64, int64|8|
|float64, complex64|8|
|complex128|16|
|uint, int|32bit 4, 64bit 8|
|uintptr||