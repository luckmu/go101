# more about deferred function calls

[1. 很多有返回值的内置函数是不能被延迟调用的](#calls-to-many-built-in-functions-with-return-results-cant-be-deferred)

[2. 延迟函数的估值时刻](#the-evaluation-moment-of-deferred-function-values)

[3. 延迟方法的属主参数的估值时刻](#the-evaluation-moment-of-receiver-arguments-of-deferred-method-calls)

[4. 可能的资源泄漏](#kind-of-resource-leaking-by-deferring-function-calls)

## calls to many built-in functions with return results can't be deferred

自定义函数的结果都可以舍弃，大多数内置函数（除了 `copy` 和 `recover` ）的调用结果都不可舍弃，而延迟函数调用的所有结果都必须舍弃。
```go
// defer append(s[:1], "x", "y")
defer func() {
    _ = append(s[:1], "x", "y")
}()
```

## the evaluation moment of deferred function values
延迟函数的估值时刻

函数在进入延迟调用栈前被估值（指 `defer aFunc(...)` 这个本身函数的值，不是函数体中变量值），该函数值入栈时不会校验，结果是 `nil` 函数值会入栈，但实际调用时会 `panic`

```golang
aFunc := func() {
    fmt.Println(false)
}
defer aFunc()
aFunc = func() {
    fmt.Println(true)
}
// 结果是 false
```

## the evaluation moment of receiver arguments of deferred method calls
like this: `defer t.M().M()`

首先估值 `t.M() -> tm`, 然后将 `tm.M()` 压入栈中

## kind-of resource leaking by deferring function calls
及时释放掉资源
```go
// 普通情况
f, err := os.Open(file.path)
if err != nil {
    return err
}
// 用匿名函数包裹，缩短其生命周期
if err := func() error {
    f, err := os.Open(file.path)
    if err != nil {
        return err
    }
    defer f.Close()
}
```
