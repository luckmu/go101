# channel

*并发编程*

`chan` T : `chan<-` T & `<-chan` T

*unbuffered channel* & *buffered channel*: *容量是否为 0*

operations:

+ send: `ch <- v`
+ recv: `<- ch`
+ close: `close(ch)`
+ capacity: `cap(ch)`
+ length: `len(ch)`

categories:

1. **nil channel**
2. **non-nil closed channel**
3. **non-nil not-closed channel**

|op|nil channel|non-nil closed channel|non-nil not-closed channel|
|:-:|:-:|:-:|:-:|
|close|panic|panic|closed **(C)**|
|send|forever block|panic|block or send **(B)**|
|recv|forever block|never block **(D)**|block or recv **(A)**|

rules:

+ *close nil/closed channel 将 panic*
+ *send to closed channel 将 panic*
+ *send to/ recv from nil channel 将 block forever*

**[channel operation explanations zh ver.](https://gfw.go101.org/article/channel.html#channel-operation-explanations) explains 4 operation scenarios A, B, C, D listed in above table**

3 internal structures: *recv <- (buffer or directly) <- send*

1. **sendq** (*sending goroutine queue*, generally FIFO, linked list)
2. **qcount & dataqsize** (*value buffer queue*, absolutely FIFO, circular linked list, dataqsize is capacity, qcount is number of buffered items)
3. **recvq** (*receiving goroutine queue*, generally FIFO, linked list)

shallow copy, 2 or 1 times (buffer or directly)

[channel range zh ver.](https://gfw.go101.org/article/channel.html#range)

*one value range*

```go
// following 2 expressions perform the same
for v := range ch {
    // ...
    // close(ch) will break the loop
}

for {
    v, ok := <-ch
    if !ok {
        break
    }
    // ...
}
```

[channel select zh ver.](https://gfw.go101.org/article/channel.html#select)

[channel select implementation zh ver.](https://gfw.go101.org/article/channel.html#select-implementation)

1. top -> bottom, left -> right **evaluation** (*v := <- ch, not ch <- v*)
2. get **case branch order**, random sort case branches, default-branch is the last
3. get **channel lock order** (sort by chan addresses, heap sort), *first N different chans* (to avoid getting lock of the same chan)
4. get locks
5. **check branches** by **case branch order**
    + case branch & send to closed channel, unlock by reverse **channel lock order**, and panic current goroutine, jump to 12
    + case branch & **non-blocking op**, unlock by reverse **channel lock order**, execute code block, jump to 12
    + default, unlock by reverse **channel lock order**, execute default code, jump to 12
6. **push current goroutine (and case info) into all *sending/receiving goroutine queues* which are corresponded to the case-ops.** (current goroutine may be pushed into the 2 queues of a same channel several times, because multiple case-op can be correspond to the same goroutine)
7. block current goroutine, then unlock all channels by reverse **channel lock order**
8. ... current goroutine is blocking, waiting for other goroutines to wake it up ...
9. current goroutine is waked up by another goroutine (close/send/recv op). send/recv op then there msut be one corresponding case-op to deal with the data. and current goroutine will be popped out by the recving/sending goroutine queue.
10. get locks by **channel lock order**
11. dequeue current goroutine (popped out or other methods) from all receiving/sending goroutine queues which are corresponded to the case-ops
12. finish

[select.go](https://github.com/golang/go/blob/master/src/runtime/select.go)

unbuffered channel eg:

recvq <- (buffer) <- sendq

`sendq.first` and `recvq.first` cannot be the same

cannot directly block the main goroutine (as the first recv, send goroutine, or it will block the main goroutine)
