# channel use cases

future/promise

request/response

返回单向接收通道作为函数返回结果

```go
func req() <-chan int32 {
    r := make(chan int32)
    go func() {
        time.Sleep(time.Second * 3)
        r <- rand.Int31n(100)
    }
    return r
}
```

将单向发送通道类型用作函数实参

```go
func req(r chan<- int32) {
    time.Sleep(time.Second * 3)
    r <- rand.Int32n(100)
}
```

采用最快回应

```go
// N 个数据源，为了防止被舍弃response对应的协程永久阻塞，
// 传输数据必须使用一个容量至少为N-1的缓冲通道
func req(c chan <- int32) {
    ra, rb := rand.Int31(), rand.Intn(3) + 1
    time.Sleep(time.Duration(rb) * time.Second)
    c <- ra // 避免在这里阻塞
}

func main() {
    c := make(chan int32, N-1) // 避免loop时阻塞
    for i := 0; i < N; i++ {
        go req(c)
    }
    rnd := <- c // 采用了最快完成的结果
}
```

使用通道实现通知

发送值实现单对单通知
```go
done := make(chan struct{})

go func() {
    // do something...
    done <- struct{}{}
}()
// do other things...
<- done
```
接收值实现单对单通知
```go
done := make(chan struct{})

go func() {
    <- done
}()
done <- struct{}{}
```

多对单和单对多通知

```go
type T = struct{}

func worker(id int, ready <-chan T, done chan<- T) {
    <-ready
    // do works...
    done <- T{}
}

func main() {
    ready, done := make(chan T), make(chan T)
    go worker(0, ready, done)
    go worker(1, ready, done)
    go worker(2, ready, done)
    ready <- T{}; ready <- T{}; ready <- T{}
    // 关闭一个通道群发通知
    // close(ready) 来实现上个语句一样的效果
    <-done; <-done; <-done
}
```

无阻塞检查通道是否关闭

```go
func isclosed(c chan T) bool {
    select {
    case <- c:
        return true
    default:
    }
    return false
}
```

峰值限制（peak/burst limiting）

```go
limitch := make(chan struct{}, N)
// go func() {
//     limitch <- struct{}{}
// }()
for i := 0; ; i++ {
    select {
        case <- limitch:
            // do something...
        default:
            // abandon
    }
}
```

无阻塞“最快回应”

```go
func req(c chan<- itn32) {
    ch := rand.Int31()
    select {
        case c <- ch:
        default:
    }
}

func main() {
    c := make(chan int32, 1) // 容量至少为1
    for i := 0; i < 5; i++ {
        go req(c)
    }
    rnd := <-c
}
```
