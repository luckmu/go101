# nil

## nil != nil
`nil` 是有类型的，只是没有默认类型，且需要 compiler 推断类型。所以不能将不同类型的值进行比较。

## explanation
> `nil` 预声明标识符，且**带有类型**

> 但 `nil` 没有默认类型（如 `true`，`false` -> `bool`；`iota` -> `int`），需要编译器通过上下文推断类型

> nil 不是 keyword，可以被同名标识符遮挡
```go
var nil int
prinln(nil)
```

> 可能与直观不符，对类型的`nil`操作往往是允许的
+ 访问 `nil` `map`
+ 可 `range` `nil chan(永久阻塞)`, `nil map`, `nil slice`, `nil slice pointer`

In Go, `nil` is the **zero value for pointers, interfaces, maps, slices, channels and function types**, representing an uninitialized values.

```go
_ = (*int)(nil) == nil
_ = (interface{})(nil) == nil
_ = (map[int]struct{})(nil) == nil
_ = ([]int)(nil) == nil
_ = (chan struct{})(nil) == nil
_ = (func())(nil) == nil
```
