# `nil`s in go

nil is **predeclared identifier without default type** (unlike other predeclared identifier, e.g., `true`/`false` is `bool`, `iota` is `int`)

你应该让编译器能成功推断出 nil 类型

```go
var _ = (*struct{})(nil) // 推断类型
var _ *struct{} = nil    // 声明类型
```

nil 不是一个 keyword `var nil = 5 // valid`

`nil` 有类型, 这导致了以下现象
+ 不同类型 `nil` 尺寸不一样 -> 等于类型尺寸
+ `nil` 的比较和类型比较表现相同
  + `([]int)(nil) == ([]int)(nil)` -> 不可比较类型
  + `(interface{})(nil) == (*int)(nil)` -> 可比较类型 & 可以隐式转换 (`*int` 实现了 `interface{}`)
  + `([]int)(nil) == nil`, especially, `slice`, `map`, `func` 可以和 不带类型的 `nil` 比较

特殊地, `(interface{})(nil) == (*int)(nil) // false`

+ non-interface -> interface, dynamic type is *int, dynamic value is nil
+ interface, dynamic type is nil, dynamic value is nil
