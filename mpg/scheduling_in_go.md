# scheduling in go

[scheduling_in_go_part_2](https://www.ardanlabs.com/blog/2018/08/scheduling-in-go-part2.html)

`N := runtime.NumCPU();` 分配 N 个 P, initial G (the path of execution for a Go program)

Go Scheduler 有2种运行队列：LRQ (Local Run Queue) & GRQ (Global Run Queue)

每个P有1个LRQ，用于执行在该P上下文期间分配给它的Goroutines，Goroutines轮流在分配给该P上的M进行上下文切换

OS在core进行上下文切换，Goroutines在M进行上下文切换

![](./94_figure2.png)

Goroutines 3 high-level states:
+ Waiting: Goroutines is stopped and waiting for something in order go continue. This could be for reasons like ***waiting for the OS (system calls) for synchronization calls (atomic & mutex operations)***. These types of latencies are a root cause for bad performance.
+ Runnable: Means goroutines wants time on a M so it can execute its assigned instructions. If you have lots of goroutines that want time, then goroutines have to wait longer to get time. Also the individual amount of time any given goroutines gets is shortened as more goroutines compete for time. This type of latencies can also be a cause of bad performance.
+ Executing: Means the goroutine is has been placed on an M and is executing its instructions.

4 classes of events allow scheduler to make scheduling decisions.
+ the use of keyword `go`
+ garbage collection
+ system calls
+ synchronization and orchestration

+ asynchronous system calls (network poller)
  + only G is moved to network poller
  + previous M&P pair will get another G to execute
+ synchronouse system calls file-base I/O
  + M&G pair is moved off
  + a new M2 is brought in to serve the P, and another G can be selected from the LRQ and context-switched on new M2
  + blocking system call made by goroutine-1 finished. goroutine-1 can move back into LRQ and be served by the P again. M1 is then places on the side for future use if this scenario needs to happen again.

参考 golang 中 schedt

```go
// about m
mdile        mintptr // idle m's waiting for work
nmidle       int32   // number of idle m's waiting for work
nmidlelocked int32   // number of locked m's waiting for work
mcount       int32   // number of m's that have been created
maxmcount    int32   // maximum number of m's allowed (or die)

// about p
pidle  puintptr // idle p's
npilde uint32   // number of idle p's

// about g
// global runnable queue
runqhead guintptr
runqtail guintptr
runqsize int32
```
