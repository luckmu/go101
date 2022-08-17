# method

[1. 接收者类型, 接收者基类型?](#receiver-type-receiver-base-type)

[2. 方法对应**编译器创建的的隐式函数声明**的形式](#methods-and-implicit-functions-declared-by-compiler)

[3. 方法描述和方法集(`interface`)](#method-specification--method-set)

[4. Others](#others)

## ***receiver type? receiver base type?***
`T`, `*T`: **receiver type** of methods; `T`: **receiver base type** of methods
## ***methods, and implicit functions declared by compiler***
### examples, methods
```golang
type Book struct {
    Pages int
}
// Value Receiver
func (b Book) Pages() int {
    return b.Pages
}
// Pointer Receiver
func (b *Book) SetPages(pages int) {
    b.Pages = pages
}
```
### implicit functions decalred by Go compiler
```golang
// Step1. 编译器声明隐式函数
func Book.Pages(b Book) int {
    return b.pages
}
func (*Book).SetPages(b *Book, pages int) {
    b.pages = pages
}
// Step2. 改写显式方法
func (b Book) Pages() int {
    return Book.Pages(b)
}
func (b *Book) SetPages(pages int) {
    (*Book).SetPages(b, pages)
}
// Step3. 最终可直接调用函数
var b Book
var pages int
(*Book).SetPages(&b, pages)
(*Book).Pages(&b)
Book.Pages(b)
```
### more implicit method & function declared by compiler (**value receiver**)
```golang
// 语法糖行为, 值接收器(value receiver)多声明1个隐式方法和函数
// 所以, var b *Book, b.Pages 是合法的
func (b *Book) Pages() int {
    return (*Book).Pages(b)
}
func (*Book).Pages(b *Book) int {
    return Book.Pages(*b)
}
```

## ***method specification & method set***
```golang
func (t T) funcname(parameters...) (rets...) {
  // ...
}
// 函数声明: `func` + receiver + 方法描述 + 函数体
// 即, 方法描述 = funcname(parameters...) (rets...)
// e.g.
// Pages() int
// SetPages(pages int)
```

## ***others***

[method as function](https://gfw.go101.org/article/method.html#method-as-function)

in a way, *method* is just *function*, compiler will declare coressponding functions for user defined methods

*method is special function, called member function*

类型的零值也有和方法同名的成员函数，注意 pointer receiver 的成员函数解引用时会 panic

[method value normalization](https://gfw.go101.org/article/method.html#method-value-normnalization)

+ v T
  + v.m -> (&v).m (pointer receiver, *implicit function*)
  + v.m -> v.m (value receiver)
+ p *T
  + p.m -> (*p).m (value receiver, *syntactic sugar*)
  + p.m -> p.m (pointer receiver)
