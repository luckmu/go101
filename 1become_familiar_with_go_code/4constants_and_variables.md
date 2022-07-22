# constants and variables

## untyped values and typed values
+ nil 是没有默认类型的 (nil is untyped)
+ 默认类型: 字符串(string), 布尔(bool), 整形(int), rune字面量(rune), 浮点数(float), 虚部字面量(complex128)


## Explicit conversions of untyped constants

`T(v)`: 将值 `v` 转换为类型 `T`

```golang
maxuint = ^uint(0)
maxint  = int(^uint(0)>>1)
```

每个短声明语句中至少有1个新声明变量

```golang
lang, year := "golang", 2007
year, createdBy := 2009, "Google Search"
// createdBy is a new variable
```

+ 1个非常量浮点数和整数可以显式转换到其他任何1个浮点数和整数类型
+ 1个非常量复数可以显式转化到其他任何1个复数类型

非常量数字值转换
+ 比特位数多->比特位数少, 高位比特舍弃, 低位比特保留
+ 非常量浮点数->整型, 浮点数小数部分舍弃
+ 非常量整数或浮点数->浮点数类型, 精度丢失是可能的
+ 非常量复数->复数类型, 精度丢失是可能的
+ 显示转换涉及非常量浮点数或复数数字, 原值溢出了目标类型的表示范围, 行为未定义


**类型不确定常量表示的值可以溢出其默认类型**

```golang
const n = 1 << 64 // 默认类型为 int
const r = 'a' + 0x7FFFFFFF // 默认类型为 rune
const x = 2e+308 // 默认类型为 float64
_ = n >> 2
_ = r - 0x7FFFFFFF
_ = x/2

// 区别于以下3个语句, 指定类型后 overflow 错误
// const n int = 1 << 64
// const r rune = 'a' + 0x7FFFFFFF
// const x float64 = 2e+308
```
**每个常量标识符将编译的时候被其绑定的字面量替代(可以看作C中的#define), 如果运算中的所有运算符都为常量, 则此运算的结果也为常量**
