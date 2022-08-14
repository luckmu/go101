# string

```golang
// 移位操作符在编译阶段估值的条件 常量
_ = float64 = 1 << m // 根据 m 是否常量: float64(1) << m; float(1 << m)
// len(s) 是常量表达式, 但 len(s[:]) 不是
const s = "abc.def."
var a byte = 1 << len(s) / 128
var b byte = 1 << len(s[:]) / 128
fmt.Println(a, b) // 2, 0
// 等效于:
//   byte(1 << len(s) / 128)
//   byte(1) << len(s[:]) / 128, overflow so it's 0
// len(aString[start:end]) 和 aString 共享1部分底层 bytes

// 字符, 码点, 字节
// 1个字符由若干码点表示(如, 1英文字符对应1码点, 1中文字符对应1码点, ...)
// 1个码点由若干字节表示(如, 1英文码点对应1字节, 1中文码点对应3字节, ...)
// 英文: 1 - 1 - 1
// 中文: 1 - 1 - 3
// 码点在Go中用rune表示, rune是int32的别名
```

```go
type _string struct {
    elements *byte // underlying bytes
    len int // number of bytes
}
```

***编译时估值函数***
+ `unsafe.Sizeof`, `unsafe.Alignof`, `unsafe.Offsetof`
+ `len`, `cap`:
  + 字符串常量 `len(s)` 总是编译时估值
  + slice, array: 表达式不含数据接收操作和结果为非常量的函数调用, 总是在编译时估值
+ `real`, `imag`: ...
+ `complex`: ...

***字符串相关的类型转换***
1. 1个字符串值可以被显式转换为1个字节切片, 反之亦然
2. 1个字符串值可以被显式转换为1个码点切片, 反之亦然

***进行字符串和字节切片转换时, 两边都是深拷贝(意味着需要重新开辟1块同样大小的内存), 原因是字节切片是可以修改的, 字符串是不可修改的, 所以2者不能共享底层字节序列***

***字符串和字节切片转换的编译器优化***
```golang
// 1. for-range 中, 跟随 `range` 的 []byte -> string
for i, b := range []byte(str) {
    _, _ = i, b
}

// 2. map 中读取元素时, []byte -> string
m[string(key)] = "value"    // 写 map, 没有优化, 需要 deep copy
fmt.Println(m[string(key)]) // 读 map, 优化, 不需要 deep copy

// 3. 比较表达式中的 []byte -> string, 不需要 deep copy
if string(x) != string(y) {

    // 4. 字符串衔接(至少含1个非空字符串常量), []byte -> string, 不需要 deep copy
    s = (" " + string(x) + string(y))[1:] // 不需要 deep copy, 含至少1个非空字符串常量
    s = string(x) + string(y)             // 需要 deep copy
}
```

**`for-range`跟随字符串, 遍历其中的码点(int32, 而非字节元素)**

**那么如何遍历字符串中的字节元素呢?**
```golang
// 1. 
for i := 0; i < len(s); i++ {
    _ = s[i]
}
// 2. 官方编译器, 这种效率更高!!!
for _, b := range []byte(s) {
    _ = b
}
```

**得到 s 中的字节数和码点数**
```golang
// 1. `for-range`
// 2. `unicode/utf8`: utf8.RuneCountInString()
// 3. len([]rune(str))
//   法3编译器优化, 避免1个深拷贝, 使得以上3种方法效率相同, 时间复杂度均为O(n)
```

***语法糖: 将字符串当作字节切片使用***
```golang
var bs []byte
_ = append(bs, "sugar"...)
```

***字符串比较***
```golang
// 2个复杂度为O(1)的情况
// 1. string 长度不相等
// 2. 底层字节序列指针相等, 比较结果等于比较这两个字符串的长度
// 所以, 尽量避免比较2个很长的不共享底层字节序列的相等的(或者几乎相等的)字符串
```
