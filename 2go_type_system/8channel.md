# channel

```golang
// buffered & unbuffered channel? cap 0
// 
// 通道 recv, send 的几种情况?
// Recvq <- (Buffer or Directly) <- Sendq
//   Recvq: recv goroutines queue
//   Sendq: send goroutines queue
// 1. goroutine R 从非零且未关闭的通道接收数据
//   1.1. 缓冲队列不为空: Recv空, Buffer pop, Sendq push to Buffer
//   1.2. 缓冲队列为空: Sendq push(recv)
//   1.3. 缓冲队列, 发送数据协程队列都为空: R block
// 2. goroutine S 向非零且未关闭的通道发送数据
//   2.1. 接收数据协程队列不为空: Buffer空, Recvq pop R, S send to R
//   2.2. 接收数据协程队列为空, 缓冲队列没满: S send to Buffer
//   2.3. 接收数据协程队列为空, 缓冲队列已满: S block
// 3. 协程成功获取到1个非零且未关闭的通道的锁并且准备关闭这个通道
//   3.1. 接收数据协程队列不为空: Buffer空, Recvq 中协程被依次弹出, 接收1个零值, 恢复运行状态
//   3.2. 发送数据协程队列不为空: 
v = <-ch
v, sentBeforeClosed = <-ch
```

***通道含底层部分, 赋值后共享底层数据, 比较这两个通道结果为`true`***

***通道元素值的传递都是复制过程(只复制直接部分)***

***`for-range`用在通道(不断从channel接收数据, 直到通道关闭且Buffer为空), 只允许出现1个循环变量***
```golang
// 以下两个表达式等价
for v := range aChannel {
    // using v
}
for {
    v, ok := <-aChannel
    if !ok {
        break
    }
    // using v
}
```

***`select-case`用法***
```golang
select {} // select 和 {} 之间无任何表达式, 这个表达式会永久阻塞
// select 选择分支顺序随机
// 所有 case 操作阻塞的情况, 若 default 存在, 执行 default; 否则将当前协程推入所有
//   阻塞操作相关通道的发送数据协程队列或接收数据协程队列
```

***`select-case` 流程控制实现机理***
1. 将所有`case`中涉及*通道表达式*和*发送值表达式*从上到下, 从左到右顺序估值
2. 所有**分支随机排序**; `default`分支**总是排最后**, case 中可能有相同的 `channel`
3. 根据通道 address 进行排序(**前N个通道不重复, 此为通道锁顺序**, 逆序即为逆通道锁顺序)
4. 通道锁顺序获取相关通道的锁
5. 按第2步检查分支顺序: </br>5.1. `case`分支, 向已关闭通道发送数据, 逆序解锁并在当前协程产生恐慌</br>5.2. `case`分支, 且非阻塞, 逆序解锁并执行相应`case`分支代码块</br>5.3. `default`分支, 逆序解锁并执行该`default`分支代码
6. 将当前协程推入每个`case`相应的`发送数据协程队列`或`接收数据协程队列`(当前协程可能被多次推入同1`channel`的这两个队列)
7. 使前协程阻塞并逆序解锁所有`channel`
8. ..., 当前协程阻塞, 等待其它协程通过通道操作唤醒当前协程, ...
9. 当前协程被唤醒(通道关闭或数据发送/接收操作), 某`case`操作与之对应, 当前协程从相应`case`相关通道的`发送/接收数据协程队列`中弹出
10. 按3步中通道锁顺序获取锁
11. 将当前协程从各`case`对应通道的`发送/接收数据协程队列`中移除</br>11.1. 如果当前协程是被通道关闭操作唤醒, 跳到第5步</br>11.2. 数据发送/接收操作唤醒, 相应`case`在第9步已经知道, 逆序解锁并执行此`case`代码块
12. 完毕

categories:

1. **nil channel**
2. **non-nil closed channel**
3. **non-nil not-closed channel**

|操作|零值nil通道|非零值已关闭通道|非零值未关闭通道|
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
```golang
func main() {
    ch := make(chan struct{})
    go func() {
        <-ch
    }()
    ch<-struct{}{}
}
```
