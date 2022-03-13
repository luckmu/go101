# mpg

[scalable_go_scheduler_design_doc](https://docs.google.com/document/d/1TTj4T2JO42uD5ID9e89oa0sLKhJYD0Y_kqxDv3I3XMw/edit)

[work-stealing scheduler](http://supertech.csail.mit.edu/papers/steal.pdf)

[go_questions/sched](https://github.com/golang-design/Go-Questions/tree/main/content/sched)

+ M: OS thread.
+ P: Resource that is required to execute Go code.
+ G: Goroutine, it has an associated P. When M is idle or in syscall, it does need P. 虚拟processor，维护一个runnable的g队列，m需要获取p才能运行g

有 exactly GOMAXPROCS 个 P，所有的 P 都被组织在1个数组中，这是 work-stealing 的1个需要。

```c++
struct P {
    Lock;
    G *gfree; // freelist, moved from sched
    G *ghead; // runnable, moved from sched
    G *gtail;
    MCache *mcache; // moved from M
    FixAlloc *stackalloc; // moved from M
    uint64 ncgocall;
    GCStats gcstats;
    // etc
    ...
}

P *allp; // [GOMAXPROCS]
P *idlep; // lock-free list
```

M准备好执行Go代码时，必须从list中pop出1个P。当M结束Go代码执行时，push P到list中。所以M必须1个关联的P才能执行Go代码。

## Scheduling
当G被created或become runnable时，被push到当前P的runnable goroutines list中。当P结束执行G，尝试从自身runnable goroutine list中pop1个G，若list为空，选择1个random victim（another P），尝试从它的runnable goroutine list中偷取一半的goroutines来执行。

## Syscalls/M Parking and Unparking
当M创建1个new G，必须确认有其他的M（another M）来执行这个G（如果没有，肯定所有M都是busy）。同样地，当1个M进入syscall，必须确认有另外的M来执行Go代码。

Spinning is 2-level:
+ 类型1：idle的M，已经关联P，M自旋等待新的G
+ 类型2：idle的M，没有可用的P，M自旋等待可用的P

That means:
1. p <= m? because M will spin for available P
2. M&P >= G? because M with associated P will spin for new G

At most `GOMAXPROCS` spinning M, because at most `GOMAXPROCS` M (which is equal to number of vCPUs).

> Idle M's of type(1) do not block while there are idle M's of type(2)

Because type(1) has gotten P, and idle M of type(2) exists, so there're no more P.

> When a new G is spawned, or M enters syscall, or M transitions from idle to busy, it ensures that there is at least 1 spinning M (or all P’s are busy). 

+ new G is spawned, need P & G to execute
+ M enters syscall, release P?
+ M transitions from idle to busy, runnig go code, hold P?

> This ensures that there are no runnable G’s that can be otherwise running; and avoids excessive M blocking/unblocking at the same time.

+ run out all OS threads, cannot run another G
+ when M will block? like IO, ..., if M blocks, it will release P

## LockOSThread
> 1.Locked G becomes non-runnable (Gwaiting). M instantly returns P to idle list, wakes up another M and blocks.

> 2.Locked G becomes runnable (and reaches head of the runq). Current M hands off own P and locked G to the M associated with the locked G, and unblocks it. Current M becomes idle.

## Idle G
> There is a global queue of (or a single?) idle G. An M that looks for work checks the queue after several unsuccessful steal attempts.
