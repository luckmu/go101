# go garbage collection

## from go blogs

[Go Gc: Prioritizing low latency and simplicity](https://go.dev/blog/go15gc)

+ concurrent, tri-color, mark-sweep collector
  + white, black, grey -> **white to purge; black to maintain; grey to scan;**
  + global variable & stack are not collected by GC, so be grey
  + pick 1 `grey_obj` from `grey_set`: blacken the `grey_obj`; find `white_objs` linked to it, adding them to `grey_set`; 
  + loop until `grey_set` is empty
+ mutator: changing pointers while the collector is running
+ write barrier
  + [lest `black_obj` point to `white_obj`](#problem-of-go15s-concurrent-tri-color-mark-sweep-algorithm)
+ stop-the-world
+ 1 GC knob
  + `GOGC` (`SetGCPercent`)

[Getting go Go: The Journey of Go's Garbage Collector](https://go.dev/blog/ismmkeynote)

+ go is value-oriented, thus? allows interior pointers (`b = &r.blk`)
+ static ahead of time compilation system so the binary contains the entire runtime; no JIT recompilation
+ 2 GC knobs:
  + `SetGCPercent`
  + `SetMaxHeap` (yet not released, but used & evaluated internally)
+ write barriers: no reachable objects get lost during the tri-color operations
+ GC Pacer: 

[Go runtime: 4 years later](https://go.dev/blog/go119runtime)

+ A new knob: `SetMemoryLimit` (since go1.19)

### problem of go1.5's concurrent tri-color, mark, sweep algorithm

a situation: `white_obj` 挂在 `black_obj` 下, 且该 `white_obj` 不在任何 `grey_obj` 下游; 该 `white_obj` 被错误清除

## write barrier

[because of ...](#problem-of-go15s-concurrent-tri-color-mark-sweep-algorithm), so `write barrier`

> write barrier, which is a small function run by the mutator ***whenever a pointer in the heap is modified***. Go’s write barrier colors the now-reachable object grey if it is currently white, ensuring that the garbage collector will eventually scan it for pointers.

即新 `obj` 自动变成 `grey`; 上述, `heap` 启用 `write barrier`, 堆要避免这个问题, so `stw(stop-the-world)`






[tri-color-hybrid-write-barrier-by-AceId](https://zhuanlan.zhihu.com/p/334999060)

+ 垃圾回收开始，所有内存块被标记为白色
+ 将所有开辟在stack和全局内存区上的内存块标记为灰色，并把它们加入一个灰色内存块列表(***全局变量不回收，stack 不由GC回收***)
+ 循环下面两步直到灰色内存块清空
  + 从灰色内存块列表中取出一个内存块，并把它标记为黑色
  + 扫描此内存块上的指针值，通过这些指针找到它们引用着的内存块。如果一个引用着的内存块为白色，则将其标记为灰色并加入灰色内存块列表；否则忽略

write-barrier-golang: 某已经被标记为黑色的内存块在标记过程中被修改而使其新引用着的某仍标记为白色的内存块时，此白色内存块需要被标记为灰色，否则此白色内存块可能被认为是垃圾而回收掉。

正常情况下

并行的三色标记，如果没有STW，存在**两个问题**：
1. 某白色对象被黑色对象引用（白色挂在黑色下）<br/>已扫描过的 black_obj 仍活动，新建关系指向 white_obj, 此时应该 grey(white_obj), 但黑色不再扫描, 该 white_obj 被误清除
2. 灰色对象与某白色对象的可达关系被破坏（灰色**同时**丢失白色）
  + 

> 为什么会产生问题？<br/>
> 黑色是已扫描，灰色是未扫描；e.g. 若某白色同时被黑色、灰色引用，并发操作解除白色和灰色的引用关系，则该白色不会被检查并被直接清除

引出强弱三色不变式
+ 强三色不变式：不存在黑色对象到白色对象的引用关系
+ 弱三色不变式：所有被黑色对象引用的白色对象都处于灰色对象的保护状态

引入 **插入屏障** 和 **删除屏障**
+ 插入屏障：
  + 操作：A对象引用B对象的时候，B对象被标记为灰色。（将B挂在A下游，B必须被标记为灰色）
  + 满足：强三色不变式（不存在黑色引用白色，白色强制变为灰色）
+ 删除屏障：
  + 操作：被删除的对象，如果自身为灰色或白色，被标记为灰色
  + 满足：弱三色不变式（保护灰色对象到白色对象的路径不会断）

**混合写屏障**
  + 操作
    + GC 开始将栈上所有对象扫描并标记为黑色（之后不再进行二次扫描，无需STW）
    + GC 期间，任何在栈上创建的对象，都是黑色
    + 被删除的对象标记为灰色
    + 被添加的对象标记为灰色
  + 满足：变形的弱三色不变式

```plain-text
添加下游对象(当前下游对象slot，新下游对象ptr) {
    标记灰色(当前下游对象slot)
    if 当前 stack 是灰色:
        标记灰色(新下游对象ptr)
    当前下游对象slot=新下游对象ptr
}
```
+ 场景1：对象（白色）被某heap对象删除引用，成为stack对象的下游
  + 添加下游：stack不触发屏障，直接挂下游
  + 删除引用：heap对象，触发屏障，obj标记为灰色
+ 场景2：对象被某stack删除引用，成为另一个栈对象的下游
  + 添加下游：stack不触发屏障，直接挂下游 + 删除引用：stack不触发屏障，直接删除
+ 场景3：对象被某heap对象删除引用，成为另一个heap对象的下游
  + 添加下游：heap添加触发屏障，obj标记为灰色
  + 删除引用：heap删除触发屏障，obj标记为灰色
+ 场景4：从stack删除，成为heap下游
  + stack删除：不触发屏障，删除当前stack obj
  + heap删除&添加：heap操作触发屏障，删除heap当前obj引用下一个obj的引用，并将下一个obj标记为灰色 & heap当前obj添加对stack删除obj的引用（stack仍是黑色）
