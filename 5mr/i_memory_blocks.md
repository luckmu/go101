# memory blocks

## Memory blocks

a memory block is a **continous memory segment** to host value parts at runtime.

one memory block may host multiple value parts:
+ memory block is allocated for *a struct value* (several fields)
+ memory block is allocated for *a array value* (many elements)
+ the underlying element sequences of two slices may be hosted on the same memory block

## Where will memory blocks be allocated on?

### Stack

At run time, each goroutine will maintain a stack, which is a memory segment (it acts as a memory pool for some memory blocks to be allocated from/on).

The **initial stack size of each goroutine is small (2KiB bytes)**. The stack will grow and shrink as needed in goroutine running.

There is a global limit of stack size each goroutine may reach. *If a goroutine exceeds the limit while growing its stack, the program crashes*. 

As of go1.17, default maximum stack size is 1 GB on 64-bit systems, and 250 MB on 32-bit systems. *We can call `SetMaxStack` in `runtime/debug` to change the size*. By the current standart go compiler implementatino, the actual allowed maximum stack size is the largest power of 2 which is not larger than the `MaxStack` setting. So the default setting, the *actual allowed maximum stack size is 512 MiB on 64-bit systems, and 128 MiB on 32-bit systems*.

Memory blocks can be allocated on stacks. *Memory blocks allocated on the stack of a goroutine can only be used (referenced) in the goroutine internally. They are goroutine localized resources. They are not safe to be referenced crossing goroutines*. A goroutines can access or modify the *value parts* hosted on a memory block allocated on the stack of the goroutine without using any data synchronization techniques.

### Heap

**Heap is a singleton in each program. It is a virtual concept. If a memory block is not allocated on any goroutine stack, the we say the memory block is allocated on heap**. Value parts hosted on memory blocks allocated on heap can be used by multiple goroutines. In other words, they can be used concurrently. Their uses should be synchronized when needed.

In fact, stacks are not essential for go programs, *go compiler/runtime can allocate all memory block on heap. Supporting stack is just to make go program run more effecient*:
+ allocating memory blocks on stacks is much faster than on heap
+ memory blocks allocated on a stack don't need to be garbage collected
+ stack memory blocks are more CPU cache friendly than heap ones

If some value parts of a local variable declared in a function is allocated on heap, we can say that value parts (and the variable) escape to heap. Run `go build -gcflags -m` to check which local values (value parts) will escape to heap at run time.

facts:
+ a field of a struct value escapes to heap -> whole struct will escape to heap
+ an element of array escapes to heap -> whole array value will escape to heap
+ an element of a slice value escapes to heap -> all elements of the slice will escape to heap
+ a value v is referenced by a value which escapes to heap, v will escape to heap

When the size of a goroutine stack changes, a new memory segment will be allocated for the stack. So *the memory blocks allocated on the stack will very likely be moved, or their addresses will change*. Consequently, the pointers, which must be also allocated on the stack, referencing these memory blocks also need to be modified accordingly.

```go
func f(i int) byte {
    type T int
    var a [1<<20]byte
    return a[i]
}

func main() {
    var x int
    println(&x)
    f(100)
    println(&x) // different from above
}
```

## When can a memory block be collected?

Memory blocks allocated for **direct parts of package-level variables will never be colleted**.

The stack of a goroutine will be colleted as a whole when the goroutine exits. So there is no need to collect the memory blocks allocated on stacks, individually, one by one. Stacks are not collected by the garbage collector.

For a memory block allocated on heap, it can be safely collected **only if it is no longer referenced (either directly or indirectly) by all the value parts allocated on goroutine stacks and the global memory zone**. We call such memory blocks as unused memory blocks. Unused memory blocks on heap will be collected by the garbage collector.

## How are unused memory blocks detected?

concurrent, tri-color, mark-sweep collector

described in [go blog](https://go.dev/blog/go15gc#the-embellishment)

```go
// 1. at the start of a GC cycle, all objs are white
whiteobjs = ColorWhite(allobjs)
// 2. color roots grey (globals, stack things)
greyobjs = ColorGrey(rootobjs)
// 3. loop until there're no grey objs
for len(greyobjs) != 0 {
    // choose one grey obj & blacken the obj
    // means remove from greyobjs and add into blackobjs
    obj = ChooseOne(greyobjs)
    blackobj = ColorBlack(obj)
    // scan ptr to other objs, turn these grey
    refobjs = FindRefObjs(blackobj)
    AddGreyObjs(greyobjs, refobjs)
}
// 4. garbage collect all white objs
collect(whiteobjs)
```

search "write barrier golang"

concurrent, when obj is black, user goroutine modify it to reference a white obj, should color the referenced obj to be grey, or it can be collected


## When will an unused memory block be collected?

>  The GOGC variable sets the initial garbage collection `target percentage`. A collection is triggered when the ratio of freshly allocated data to live data remaining after the previous collection reaches this percentage. The **default is GOGC=100**. **Setting GOGC=off disables the garbage collector entirely**.

`runtime/debug.SetGCPercent` smaller values lead to more frequent garbage collections. A negative percentage disables automatic garbage collection.

`runtime.GC` to manually start a gc.
