# type embedding

+ Part1 
  + 类型内嵌的意义(和继承的区别)?
+ Part2
  + 选择器遮挡 & 碰撞?
  + 可以省省略嵌字段的情况？(`a.B.C -> a.C`)
  + 不可以省略内嵌字段的情况？(多个选择器相同深度有同名`field`)

[1. 可内嵌类型](#embeddable-types)

[2. 选择器遮挡和碰撞](#selector-shadowing-and-colliding)

## embeddable types
[embeddable-types](https://gfw.go101.org/article/type-embedding.html#embeddable-types)

+ type T, 本身不是定义的指针类型且基类型不是指针类型或接口类型
+ type *T, T 是类型名且 T 不是指针类型或接口类型

### Meaningfulness of Type Embedding
+ 继承: `T`继承另1类型, 则获取另1类型的能力, 同时`T`可以当作另1类型使用。
+ 内嵌: `T`内嵌另1类型, 另1类型变成`T`的1部分, `T`获取另1类型的能力, 但不能当作另1类型使用。

选择器的缩写

如果一个选择器中某中间名对应一个内嵌字段, 则此项可以忽略, 因此内嵌字段也被称为匿名字段

## selector shadowing and colliding
[selector shadowing and colliding](https://gfw.go101.org/article/type-embedding.html#selector-shadow-and-collide)

+ the full-form selector with the shortest path (assume it is the only one) can be shortened as x.y other full-form selectors are **shadowed by** the one with the shallowest depth.
+ if there're more than one full-form selectors with the same shallowest depth, then none of those full-form selectors can be shortened as x.y. those full-form selectors with the shallowest depth are **colliding with** each other

```go
type A struct {
    Name string
}
type B struct {
    Name string
}
type C struct {
    A
    B
}
// C.Name // ambiguous selector, connot compile
// C.A.Name, C.B.Name
```
