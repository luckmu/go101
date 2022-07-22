# basic control flows

3 kinds of basic control flow code blocks:
+ `if-else` 条件分支代码
+ `for` 循环代码块
+ `switch-case` 多条件分支代码

control code blocks related to certain kinds of types in go:
+ 容器类型(container types)相关的 `for-range`
+ 接口类型(interface types)相关的 `type-switch`
+ 通道类型(channel types)相关的 `select-case`

除 `if-else` 外, 其它5种为可跳出代码块(`break`, `continue`, `goto`)

```golang
// InitSimpleStatement 必须为简单语句
// 常常为1条变量短声明语句
if InitSimpleStatement; Condition {
    // ...
} else {
    // ...
}
// 注意 InitSimpleStatement 中新声明变量的生命周期
// 该变量被声明在外层的隐式代码块中
// e.g.
if h := time.Now().Hour(); h < 12 {
    println("morning")
} else if h > 19 {
    println("evening")
} else {
    println("afternoon")
    h := h
    _ = h
}
// 上面2个h在此处都不可见
```

```golang
for InitSimpleStatement; Condition; PostSimpleStatement {
    // ...
}

for i := 0; i < 3; i++ {
    println(i)
    i := i // 遮挡上面声明的i
    i = 10 // 新声明的i被更改了
    _ = i
}
```

```go
// switch-case
switch InitSimpleStatement; CompareOperand0 {
case CompareOperandList1:
    // ...
case CompareOperandList2:
    // ...
case CompareOperandListN:
    // ...
default:
    // ...
}

switch n := rand.Intn(100)%5; n {
case 0,1,2,3,4:
    println(n)
case 5,6,7,8:
    n := 5 // 仅当前case代码块中可见n
    println(n)
    fallthrough
default:
    println(n) // 和第1个分支中的n是同1个变量
}
```