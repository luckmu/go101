# structs

```golang
// 1. 字节填充(padding)和内存地址对齐(memory alignment)?
// 2. tag 不作注释; 减少匿名结构体
// 3. 具名, 无名结构体转换时候的差别
```

无名结构体字面形式: `struct`+`{}`+`fields`

## `tag`
Go中结构体字段(field)可以被指定1个标签(`tag`)

空格分隔的键值对
```golang
struct {
    X `myfmt:"b1"`
}
```
**不要将`tag`用作注释**

两个声明在不同代码包中的非导出字段将总被认为是不同的字段

**1个结构体不能(直接或间接)含有1个类型为此结构类型的字段**
```golang
// 不合法的
// type A struct {
//     Next A
// }

// 合法的
type B struct {
    Next *B
}
```

**结构体字面量宜使用的完全声明**
```golang
// 尽量用第2种方式, 在修改结构体源码后, 第1中方式可能编译不通过
S{0, false}
S{x: 0, y: false}
```

**指针将被隐式解引用**
```golang
(*book2).pages = (*book1).pages
// 这两个表达式是等效的
book2.pages = book1.pages
```

**S1和S2底层类型相同(忽略字段标签)则可以互相转换;</br>特别地, 若S1和S2底层类型相同(要考虑字段标签)并且只要它们其中1个为无名类型, 则这个转换可以是隐式的。**
```golang
type S0 struct {
    x int `foo:""`
    y int
}
type S1 struct {
    x int `bar:""`
    y int
}
type S2 = struct {
    x int `foo:""`
    y int
}
var s0, s1, s2 = S0{}, S1{}, S2{}
s0 = s2     // S2 是无名类型, 隐式转换
s0 = S0(s1) // S0, S1 都是具名类型, 需要显式转换
s1 = S1(s2) // tag 不同(foo, bar), 显式转换
_ = s0
```

**结构体字段中的匿名结构体;</br>通常来说, 为了代码可读性, 尽量少用匿名结构体。**
```golang
var aBook = struct {
    author struct {
        firstName, lastName string
        gender bool
    }
    title string
    pages int
}
```