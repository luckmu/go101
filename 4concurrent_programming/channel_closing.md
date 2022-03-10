# channel closing

不要在**数据接收方**或者在有**多个发送者**的情况下关闭 channel，只应该让一个通道唯一的发送者关闭此通道

官方原则是不能关闭一个已经关闭的 channel

## Close Channels Rudely

```go
func safeClose(ch chan T)(justClosed bool) {
    defer func() {
        if recover() != nil {
            // 函数的返回结果可以在 defer 中修改
            justClosed = false
        }
    }()

    // assume ch != nil
    close(ch) // 如果 ch closed, panic
    return true // <=> justClosed = true; return
}
```

## Close Channels Politely

### sync.Once

```go
type mychannel struct {
    c chan struct{}
    once sync.Once
}

func (mc *mychannel) safeclose() {
    mc.once.Do(func() {
        close(mc.c)
    })
}

func newmychannel() *mychannel {
    return &mychannel{c: make(chan struct{})}

}
```

### sync.Mutex

```go
type mychannel struct {
	c      chan struct{}
	closed bool
	mutex  sync.Mutex
}

func newmychannel() *mychannel {
	return &mychannel{c: make(chan struct{})}
}

func (mc *mychannel) safeclose() {
	mc.mutex.Lock()
	defer mc.mutex.Unlock()
	if !mc.closed {
		close(mc.c)
		mc.closed = true
	}
}

func (mc *mychannel) isclosed() bool {
	mc.mutex.Lock()
	defer mc.mutex.Unlock()
	return mc.closed
}
```

## Close Channels Gracefully

### M receivers 1 sender, sender says "no more sends" by *closing the channel*

```go
ch := make(chan struct{})

// the sender
go func() {
    for {
        if value := rand.Intn(Max); value == 0 {
            // the only sender can close
            // the channel at any time safely
            close(ch)
            return
        } else {
            ch <- struct{}
        }
    }
}()

// the receivers
for i := 0; i < N; i++ {
    go func() {
        // receive values until ch is closed and
        // the value buffer queue of ch becomes empty
        for s := range ch {
            fmt.Printf("received\n")
        }
    }()
}
```

### 1 receiver N senders, the only receiver says "plz stop sending more" by *closing additional signal channel*

```go

datach := make(chan struct{})
stopch := make(chan struct{})

// the senders
for i := 0; i < S; i++ {
    go func() {
        for {
            select {
            case <- stopch:
                return
            default:
            }
            
            select {
            // not absolutely
            // after `<- stopch:` receiver close the channel
            // but sender still `datach <- struct{}`
            case <- stopch:
                return
            case datach <- struct{}{}:
            }
        }
    }()
}

// the receiver
go func() {
    for s := range datach {
        // 满足某个条件时终止通信
        if ... {
            close(stopch)
            return
        }
        fmt.Printf("received\n")
    }
}()
```

### M receivers N senders, any of them says "end" by *notifying a moderator to close an additional signal channel*

```go
datach := make(chan struct{})
stopch := make(chan struct{})
tostop := make(chan string, 1)

var stoppedby string

// the broker
go func() {
    stoppedby = <- tostop
    close(stopch)
}()

// the senders
for i := 0; i < S; i++ {
    go func(id string) {
        for {
            if ... {
                // 满足某个条件，从 sender#i 终止通信
                select {
                case tostop <- "sender#"+id:
                default:
                }
                return
            }

            // stop == return
            select {
            case <- stopch:
                return
            default:
            }

            // datach <- struct{}{}
            select {
            case <- stopch:
                return
            case datach <- struct{}{}:
            }
        }
    }(strconv.Itoa(i))
}

// the receivers
for i := 0; i < R; i++ {
    go func(id string) {
        for {
            select {
            case <- stopch:
                return
            default:
            }

            select {
            case <- stopch:
                return
            case s := <- datach:
                if ... {
                    select {
                    case tostop <- "receiver#"+id:
                    default:
                    }
                    return
                }
                fmt.Printf("received\n")
            }
        }
    }(strconv.Itoa(i))
}
```

## M receiver 1 sender mutation: stop signal sent by a third-party

```go
datach := make(chan int)
closing := make(chan struct{})
closed := make(chan struct{})

stop := func() {
    select {
    case closing <- struct{}{}:
        <-closed
    case <-closed:
    }
}

// the third-parties
for i := 0; i < T; i++ {
    go func() {
        r := 1 + rand.Intn(3)
        time.Sleep(time.Duration(r) * time.Second)
        stop()
    }()
}

// the senders
go func() {
    defer func() {
        close(closed)
        close(datach)
    }()

    for {
        select {
        case <- closing: return
        default:
        }

        select {
        case <-closing: return
        case datach <- rand.Intn(Max):
        }
    }
}()

for i := 0; i < R; i++ {
    go func() {
        for value := range datach {
            fmt.Printf("%d", value)
        }
    }()
}
```

## N senders mutation: data channel must be closed

```go
datach := make(chan int)
middlech := make(chan int)
closing := make(chan string)
closed := make(chan struct{})

var stoppedby string

stop := func(by string) {
    select {
    case closing <- by: <-closed
    case <-closed
    }
}

// middleware
go func() {
    exit := func(v int, needsend bool) {
        close(closed)
        if needsend {
            datach <- v
        }
        close(datach)
    }
    for {
        select {
        case stoppedby = <-closing:
            exit(0, false)
            return
        case v := <-middlech:
            select {
            case stoppedby = <-closing:
                exit(v, true)
                return
            case datach <-v:
            }
        }
    }
}()

// the 3rd-parties
for i := 0; i < T; i++ {
    go func(id string){
        r := 1 + rand.Intn(3)
        time.Sleep(time.Duration(r) * time.Second)
        stop("3rd-party#"+id)
    }(strconv.Itoa(i))
}

// the senders
for i := 0; i < S; i++ {
    go func(id string) {
        for {
            value := rand.Intn(Max)
            if value == 0 {
                stop("sender#"+id)
                return
            }

            select {
            case <- closed:
                return
            default:
            }

            select {
            case <- closed:
                return
            case middlech <- value:
            }
        }
    }(strconv.Itoa(i))
}

// the receivers
for range [R]struct{}{} {
    go func() {
        for value := range datach {
            fmt.Printf("%d", value)
        }
    }()
}
```
