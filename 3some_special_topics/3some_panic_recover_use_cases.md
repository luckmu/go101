# Some Panic/Recocer Use Cases

[1. 避免恐慌导致程序崩溃](#use-case-1-avoid-panics-crashing-programs)

[2. 自动重启因为恐慌而退出的协程](#use-case-2-automatically-restart-a-crashed-goroutine)

## use case 1: avoid panics crashing programs
线上环境是不允许程序崩溃的，所有功能，逻辑有交叉时用 `recover` 兜底，避免功能间相互影响
```go
func aHandler() {
    defer func(){
        if err := recover(); err != nil {
            // ...
        }
    }
}
```

## use case 2: automatically restart a crashed goroutine
```go
func neverExit() {
    defer func() {
        if err := recover(); err != nil {
            go neverExit()
        }
    }()
    ec := make(chan os.Signal, 1)
    signal.Notify(ec, syscall.SIGINT, syscall.SIGTERM)
    select {
    case <-ec:
        panic("restart.")
    }
}
func main() {
    neverExit()
    select{}
}
```
