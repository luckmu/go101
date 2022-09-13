# concurrent common mistakes

[1. 不当使用-futurepromise](#不当使用-futurepromise)

[2. 不是最后活跃的-sender-关闭了通道](#不是最后活跃的-sender-关闭了通道)

[3. 过多-timeafter-消耗资源](#过多-timeafter-消耗资源)

## 不当使用 Future/Promise
```go
// 串行，两个实参顺序估值
aFunc(<-fa(), <-fb())
// 先估值，再用作实参
ca, cb := fa(), fb()
aFunc(<-ca, <-cb)
```

## 不是最后活跃的 sender 关闭了通道
最后1个活跃的发送者关闭通道（向已关闭的通道发送数据会 panic），见[如何优雅地关闭通道](https://gfw.go101.org/article/channel-closing.html)

## 过多 `time.After` 消耗资源
`time.After` 会声明1个 `time.Timer` 类型，多次调用会消耗大量资源，应该声明 `time.Timer` 变量，并在 loop 中 reset 使用

```go
// go longrunning(msgs)
// n loops in each goroutine,
// n timer.After() calls, that is n timers
// m goroutines -> m*n timers
func longrunning(msgs <-chan string) {
    for {
        select {
        case <-time.After(time.Minute):
            return
        case msg := <-msgs:
            fmt.Println(msg)
        }
    }
}

// correct using
// timer.C -> make(chan Time, 1)
// goroutine:timer == 1:1
func longrunningi(msgs <-chan string) {
    timer := time.NewTimer(time.Minute)
    defer timer.Stop()

    for {
        select {
        case <-timer.C:
            return
        case msg := <-msgs:
            fmt.Println(msg)
            // while processing, time.Time is sent into timer.C
            // must be expired rather than stopped
            if !timer.Stop() {
                // clear the cache
                <-timer.C
            }
        }
        timer.Reset(time.Minute)
    }
}
```
