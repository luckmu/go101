# memory blocks

```go
// 内存块将被开辟在哪？
//   堆（heap）/栈（stack）
// go 中，堆/栈是怎样的概念？
//   标准编译器，at runtime 每个协程维护1个栈，初始大小2KB，
//   因此，开辟在某协程栈上的内存块只能在此协程内部使用，其它协程无法访问，
//   按需增长收缩：
//     1. initial: minimum required size 2KB
//     2. stack overflow: 申请1个2倍大小的栈，copy content
//     3. GC time：stack 使用少于 1/4 申请内存，申请 1/2 大小内存，copy content
//   go runtime 维护协程栈最大尺寸限制，v1.19 64位系统1GB，32位系统250MB，可用 runtime/debug SetMaxStack 修改这个限制
//   栈是虚拟概念，每个程序只有1个堆，开辟在堆上的内存块可以被多个协程并发访问
// 编译器策略，内存块会被多个goroutines访问或无法断定只被1个goroutine访问时，开辟在堆
// 栈的优点：
// + 栈上分配内存比堆快得多
// + 栈上内存块不需要GC
// + 栈上内存块对CPU缓存更友好
```

[1. 逃逸分析，heap & stack](#escape-analysis)

[2. 内存块的回收时刻](#内存块在什么条件被回收)

[3. concurrent tri-color sweep algorithm](#并发三色标记清除concurrent-tri-color-sweep-algorithm判断内存块是否仍被使用)

[4. GC 具体回收时刻](#有关gc回收时间)

## escape analysis
首先理解Go中的堆栈，内存在逻辑上分为两种结构，stack 和 heap，所有 goroutines 都会申请1块内存作为栈帧（goroutine1, goroutine2, ... : stack1, stack2, ...；初始大小为2KB，[扩缩容策略](https://docs.google.com/document/d/1YDlGIdVTPnmUiTAavlZxBI1d9pwGQgZT7IKFKlIXohQ/edit#heading=h.ywtmh4qacfis)），其它变量申请在堆。

stack & heap usage
+ stack : goroutines, after goroutine exits, stack is reclamed.
+ heap : other variables

局部声明变量某些值部被开辟在堆（heap）上，称**逃逸**至堆上

编译器检查：`go build -gcflags -m`: `moved to heap: v`

每个堆上的类型`T`变量，都有与之对应的`*T`类型隐式指针被创建，形成1个从***栈->堆***的引用关系，编译器将所有**对这个局部变量的使用替换为对这个隐式指针的解引用**

Go 官方的说法 [stack_or_heap](https://go.dev/doc/faq#stack_or_heap) ，在函数返回后如果无法判断其是否继续使用，分配在 heap；
~~局部变量（local variables）非常大，存储在堆上更有意义（？避免在 stack 多次声明）；~~

***变量取地址*** ->成为分配在 ***堆上的候选者（candidate）***，基本的逃逸分析（escape analysis）会识别该情况（***变量的生命周期不比函数返回更长时，可分配在 stack 上***）

~~包级变量（package-level variable）视为堆上的对象，被全局内存区上的隐式指针引用。这个指针引用包级变量的直接部分，直接部分又引用着其它值（部）。~~

[Understanding Allocations: the Stack and the Heap - GopherCon SG 2019](https://www.youtube.com/watch?v=ZMZpH4yT7M0)

stack for each goroutine, heap (basically) for everything else.
+ 指针，引用作参数：sharing down typically stays on the stack(passing pointers, reference).
+ 指针（包含指针的结构），引用作返回值：sharing up typically escapes to the heap(returning pointers, reference, things that have pointers).

~~When are values constructed on the heap?~~
+ ~~When a value could possibly be refereced after the function that constructed the value returns.~~
+ ~~When the compiler determines a value is too large to fit on the stack.~~
+ ~~When the compiler doesn't know the size of a value at compile time.~~

```go
type Reader interface {
    Read(p []byte) (n int, err error)
}
// instead of
// 这种方法会发生内存逃逸
// b = make([]byte, n) escapes to heap, 因为 slice is a reference
// 多次使用会产生很多垃圾
type Reader interface {
    Read(n int) (b []byte, err error)
}
```

一点，协程栈在协程退出时被**整体回收**，栈内存池不由垃圾回收器回收。

堆上的内存块，在被标记为不再使用后，在某个时刻被垃圾回收器回收。

## 内存块在什么条件被回收？
heap:
+ ~~为包级变量直接部分开辟的内存块永远不会被回收。~~
+ 堆上的内存块，不被 reference 后，可以被安全的垃圾回收（在以后的某个时刻被回收）。

stack:
+ each 协程的栈将在协程退出时被整体回收，栈上开辟的内存块（memory block）不必单独回收，栈内存池不由垃圾回收器回收


## 并发三色标记清除（concurrent tri-color sweep algorithm），判断内存块是否仍被使用？
1个 GC 分为2个阶段：**标记（mark phase） + 清除（sweep phase）**

1. 标记（mark phase）：
+ 每1轮，所有堆内存标记为**白色**
+ 栈和全局内存区域标记为灰色，加入**灰色内存块列表**
+ 循环以下步骤直到**灰色内存块列表**为空
  + 灰色内存块列表中取出1个内存块，标记为**黑色**
  + 扫描此内存块上的指针，找到引用的内存块。如果是**白色**，标记为**灰色**并加入**灰色内存块列表**；否则忽略。
2. 清除（sweep phase）：在此阶段，仍被标记为**白色**的内存块被认为不再使用而被回收掉。

~~此垃圾回收算法不会移动内存块来整理内存碎片（compactiing）。~~

## 有关GC回收时间
+ [About `GOGC` and `GOMEMLIMIT`](https://pkg.go.dev/runtime#hdr-Environment_Variables)
+ [A Guide to the Go Garbage Colletor](https://go.dev/doc/gc-guide)
+ [GC Pacer Redesign](https://github.com/golang/proposal/blob/master/design/44167-gc-pacer-redesign.md)

search "write barrier golang"

concurrent, when obj is black, user goroutine modify it to reference a white obj, should color the referenced obj to be grey, or it can be collected

>  The GOGC variable sets the initial garbage collection `target percentage`. A collection is triggered when the ratio of freshly allocated data to live data remaining after the previous collection reaches this percentage. The **default is GOGC=100**. **Setting GOGC=off disables the garbage collector entirely**.

`runtime/debug.SetGCPercent` smaller values lead to more frequent garbage collections. A negative percentage disables automatic garbage collection.

`runtime.GC` to manually start a gc.
