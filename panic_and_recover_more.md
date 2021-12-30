# panic and recover more

函数调用的退出阶段

一个函数调用可能通过三种途径进入它的退出阶段：

1. 此调用正常返回
2. 此调用中产生了一个恐慌
3. 当 runtime.Goexit 函数在此调用中被调用并且退出完毕

函数调用关联恐慌和Goexit信号

恐慌是可以恢复的，但Goexit信号是不可以取消的

在任何一个给定时刻，一个函数调用*最多只能和一个未恢复的恐慌相关联*。如果一个调用正和一个未恢复的恐慌相关联，则

+ 在此恐慌被*恢复之后，此调用不再和任何恐慌相关联*
+ 当在此函数调用中*产生了一个新的恐慌，此新恐慌将替换原来的未被恢复的恐慌作为和此函数调用相关联的恐慌*

```go
defer func() {
    fmt.Println(recover()) // 3
}()

defer panic(3) // 替换 panic 2
defer panic(2) // 替换 panic 1
defer panic(1) // 替换 panic 0
panic(0)
```

```go
func main() {
    go func() {
        // step4: panic2 替换 panic0
        defer func() {
            // step3: panic2 替换 panic1
            defer panic(2)
            func() {
                // step2: panic1 时，panic1 & panic0 共存
                panic(1)
            }()
        }()
        // step1: panic0
        panic(0)
    }()
    select{}
}
```

哪些 recover 调用会起作用

```go
// 直接外层调用是延迟调用，且和最新未恢复panic相关联
defer func() {
    recover()
}()
panic(0)
```
