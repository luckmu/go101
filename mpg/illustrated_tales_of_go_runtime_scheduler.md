# illustrated tales of go runtime scheduler

[illustrated-tales-of-go-runtime-scheduler](https://medium.com/@ankur_anand/illustrated-tales-of-go-runtime-scheduler-74809ef6d19b)

Some instances when a goroutine can block
+ Sending and Receiving on Channel
+ Network I/O
+ Blocking System call
+ Timers
+ Mutexes

> Of course, the blocked goroutines shouldn't block the underlying kernel thread.

blocked goroutine during channel operation
+ recvq: store the blocked goroutines trying to read from the channel
+ sendq: store the blocked goroutines trying to send to the channel

system call (blocking system call blocks the underlying kernel thread, so cannot schedule any other goroutine on this thread)
+ resulting in CPU waste, so the G&M pair is moved off from P, attach new M to the P, find G to execute

non-blocking system call
+ blocks goroutines on the integrated runtime poller, and the thread is released to run another goroutine.
+ G is moved off, M&P pair find another G to execute

basiclly all of the goroutines blocked for operation on
+ Channel
+ Mutex
+ Network IO
+ Timers

Functionality
+ It can handle Parallel Execution (Multiple threads).
+ Handles Blocking System call and network I/O.
+ Handles Blocking User level (on channel) calls.
+ Scalable (Distributed Scheduler - Run queue per thread, `LRQ`&`GRQ`&`work-stealing`)
+ Efficient
+ Fair

> from where we should run our next goroutine?

**Poll order has been defined as follows.**
1. local run queue
2. global run queue
3. network poller
4. work stealing


Why M >= P ?
> 完成系统调用后，若销毁M，且没有足够的M执行goroutine，需要创建一个新的M；选择不销毁M，让M spin，虽然有loop的cpu消耗，但比频繁销毁&创建thread优秀
