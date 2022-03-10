# concurrent atomic operations

```go
// for unsigned integers
u - c
  = u + (-c)
  = u + ^uintn(c-1)

atomic.AddUint64(&u, ^uint64(c-1))
```

atomic

```go
type T struct{v int}
var pT *T
addr := (*unsafe.Pointer)(unsafe.Pointer(&pT))

atomic.StorePointer(addr *unsafe.Pointer, val unsafe.Pointer)
atomic.LoadPointer(addr *unsafe.Pointer) (val unsafe.Pointer)
atomic.SwapPointer(addr *unsafe.Pointer, new unsafe.Pointer) (old unsafel.Pointer)
atomic.CompareAndSwapPointer(addr *unsafe.Pointer, old unsafe.Pointer, new unsafe.Pointer) (swapped bool)
```

atomic.Value

```go
v.Store()
v.Load().(T)
old := v.Swap()
swapped := v.CompareAndSwap(ta, tb)
```
