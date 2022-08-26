# Explain Panic/Recover Mechanism in Detail

1. 函数的退出阶段（exiting phase）？进入退出阶段的方式？
2. Goexit 可以覆盖 panic 导致类似 recover 的现象发生
3. `panic()` & `runtime.GoExit()` & `os.Exit()` 使程序（协程）退出的区别？

[1. 函数的退出阶段](#exiting-phases-of-function-calls)

[2. 恐慌和Goexit信号](#associating-panics-and-goexit-signals-of-function-calls)

## Exiting Phases of Function Calls

退出阶段执行`deferred functions`，执行完毕后退出阶段结束（函数调用退出完毕）。

进入`exiting phase (or exit directly)`的3种方式:
1. 函数正常 return 后
2. 调用产生 panic
3. 当`runtime.Goexit`被调用且退出完毕

***1般`runtime.Goexit`不在主协程中调用，会等待其它goroutines执行结束然后panic.***

***`os.Exit()`会使程序立即结束，不执行`deferred function`，即跳过函数退出阶段。***


## Associating Panics and Goexit Signals of Function Calls

Goexit 会直接进入函数`exiting phase`，不可恢复。不会导致程序崩溃。
> Goexit terminates the goroutine that calls it. No other goroutine is affected.

***非`main goroutine`中的panic会使得程序crash（实际上是，最终有还未），而Goexit不会。***

***Goexit 可以覆盖 panic***

Goexit覆盖panic，最终程序不会crash，作用类似1个无body的`recover()`。
```go
func main() {
    go func() {
        defer runtime.Goexit()
        panic("g panic.")
    }()
    for runtime.NumGoroutine() > 1 {
        runtime.Gosched()
    }
}
```

> 一个recover调用只有在它的直接外层调用（即recover调用的父调用）是一个延迟调用，并且此延迟调用（即父调用）的直接外层调用（即recover调用的爷调用）和当前协程中最新产生并且尚未恢复的恐慌相关联并且此尚未恢复的恐慌不为一个Goexit信号时才起作用。 一个有效的recover调用将最新产生并且尚未恢复的恐慌和与此恐慌相关联的函数调用（即爷调用）剥离开来，并且返回当初传递给产生此恐慌的panic函数调用的参数。

参照这样的用法即可
```go
func() {
    // 爷调用
    defer func() {
        // 父调用
        if e := recover(); e != nil {s
            // ...
        }
    }()
    // 最新且未恢复的恐慌和爷调用关联
    panic("g panic.")
}
```