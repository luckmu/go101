# value parts

2 categories of Go types:

|types whose values each is only hosted *on one single memory block*|types whose values each may be hosted *on multiple memory blocks*|
|:-:|:-:|
|*solo direct value part*|*direct part* -> *underlying part*|
|boolean types<br/>numeric types<br/>pointer types<br/>unsafe poiter types<br/>struct types<br/>array types|slice types<br/>map types<br/>channel types<br/>function types<br/>interface types<br/>string types|

[internal definitions](https://gfw.go101.org/article/value-part.html#internal-definitions)

map, channel, function

```go
type _map *hashtableImpl
type _channel *channelImpl
type _function *functionImpl
```

slice

```go
type _slice struct {
    elements unsafe.Pointer // 引用底层元素
    len      int            // 当前元素个数
    cap      int            // 切片的容量
}
```

string

```go
type _string struct {
    elements *byte // 引用底层byte元素
    len      int   // 字符串长度
}
```

interface

```go
// blank interface
type _interface struct {
    dynamicType *_type          // 引用接口值的动态类型
    dynamicValue unsafe.Pointer // 引用接口值的动态值
}
// not-blank interface
type _interface struct {
    dynamicTypeInfo *struct {
        dynamicType *_type       // 引用着接口值的动态类型
        methods     []*_function // 引用着动态类型的对应方法列表
    }
    dynamicValue unsafe.Pointer // 引用着动态值
}
```

[about value copy](https://gfw.go101.org/article/value-part.html#about-value-copy)

Go 中, 每个赋值操作（包括函数调用传参等）都是值的浅拷贝过程（shallow copy）（源值和目标值的类型相同），如果源值含有间接部分，赋值操作后，目标值和源值将引用着相同的间接部分（src 和 dst 将共享底层的间接值部）

官方[FAQ](https://golang.google.cn/doc/faq#pass_by_value)指明接口值的赋值中，底层动态值将会复制到目标值。然而接口动态值是只读的，作为优化，官方标准编译器并没有复制底层动态值