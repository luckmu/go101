# Concurrent Synchronization Overview

+ `sync.WaitGroup`: `Add(delta int)`, `Done()`, `Wait()`
+ `sync.Once`
+ `sync.Mutex`: `Lock()`, `Unlock()`
+ `sync.RWMutex`: `Lock()`, `Unlock()`, `RLock()`, `RUnlock()`
+ `sync.Cond`: `Wait()`, `Signal()`, `Broadcast()`，

## `sync.Cond`
用在多个协程等待1个协程的情况，顾名思义 `Wait()` 在等待 goroutine 中调用，如果只是 1:1 的情况，直接使用 chan 阻塞即可，1:N 的情况，用 `Broadcast()`
```go
// 初始化
c := sync.NewCond(&sync.Mutex{})
// 3 个方法: Wait(), Signal(), Broadcast()
// Wait() 的固定使用，顾名思义是等待协程，
// 上游 goroutine 处理完毕后调用或 c.Broadcast() 或 c.Signal()
// 唤醒所有（某个）协程进行数据处理
go func() {
    c.L.Lock()
    defer c.L.Unlock()
    for !condition() {
        c.Wait()
    }
    // ... code ...
}()

// code 部分处理数据，使得 condition 判定为 true，
// 解锁并唤醒 goroutines 进行后续处理
go func() {
    c.L.Lock()
    defer c.Broadcast()
    defer c.L.Unlock()
    // defer c.Signal()，只会唤醒 1 个 Wait() 协程，其余继续阻塞
    // ... code ...
}
```

## sync/atomic

```go
// T 可以是 int32, int64, uint32, uint64 和 uintptr
func AddT(addr *T, delta T)(newv T)
func LoadT(addr *T) (val T)
func StoreT(addr *T, val T)
func SwapT(addr *T, newv T) (oldv T)
func CompareAndSwapT(addr *T, oldv, newv T) (swapped bool)

// and Pointer
func LoadPointer(addr *unsafe.Pointer) (val unsafe.Pointer)
func StorePointer(addr *unsafe.Pointer, val unsafe.Pointer)
func SwapPointer(addr *unsafe.Pointer, newv unsafe.Pointer) (oldv unsafe.Pointer)
func CompareAndSwapPointer(addr *unsafe.Pointer, oldv, newv unsafe.Pointer) (swapped bool)
```

***v1.19+***
```go
// T 可以是 atomic.Int32, atomic.Int64, atomic.Uint32, atomic.Uint64, atomic.Uintptr
func (*T) Add(delta T) (newv T)
func (*T) Load() T
func (*T) Store(val T)
func (*T) Swap(newv T) (oldv T)
func (*T) CompareAndSwap(oldv, newv T) (swapped bool)

// and Pointer
(*Pointer[T]) Load() *T
(*Pointer[T]) Store(val *T)
(*Pointer[T]) Swap(newp *T) (oldp *T)
(*Pointer[T]) CompareAndSwap(oldp, newp *T) (swapped bool)
```
