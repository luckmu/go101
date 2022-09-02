# go no copy

[1. noCopy & `go vet`](#struct-nocopy)

[2. `type copyChecker uintptr`](#checker-in-cond)

## struct noCopy

```go
type noCopy struct {}
func (*noCopy) Lock() {}
func (*noCopy) Unlock() {}

type noCopier struct {
    noCopy
}

func main() {
    var d noCopier
    d2 := d
    _ = d2
}
```

实现了 `Lock()`，`Unlock` 的结构体不能被拷贝, `Mutex` 内部结构：

```go
type Mutex struct {
    state int32
    sema  uint32
}
```

`Mutex` 被拷贝后就不是同1个锁，在存在数据竞争的代码段使用这样的锁，依然可能造成冲突

使用 `noCopy` 来避免拷贝，`go vet ...` 来检查

## checker in cond

```go
// type copyChecker uintptr 存储1个地址值
// 通过 CompareAndSwapUintptr，第1次 check() 将 c 地址赋值给 c（即 c = uintptr(unsafe.Pointer(&c))），
//   此时 uintptr(*c) != uintptr(unsafe.Pointer(c)) 为 true
// 若 copy，c 地址改变，check() 中: uintptr(*c) != uintptr(unsafe.Pointer(c)) 为 false
// 即 if (true) && (false) && (false) {}

// Check Original: if (false) && (true) && (false) {}
// Check Copy: if (true) && (true) && (true) { panic() }
type copyChecker uintptr

func (c *copyChecker) check() {
    if uintptr(*c) != uintptr(unsafe.Pointer(c)) && 
        !atomic.CompareAndSwapUintptr((*uintptr)(c), 0, uintptr(unsafe.Pointer(c))) &&
        uintptr(*c) != uintptr(unsafe.Pointer(c)) {
            panic("struct is copied")
        }
}
```

### double `uintptr(*c) != uintptr(unsafe.Pointer(c))`：

多于 1 个 goroutines，同时进行 init check, 并发 atomic op，只有 1 个操作成功：

+ `atomic pass g`: true, false, false
+ `atomic fail g`: true, true, false(被某 g 成功改值为 `uintptr(unsafe.Pointer(c))`，此时应该是 false)

可见如果没有第 2 个 `uintptr(*c) != uintptr(unsafe.Pointer(c))`，所有 atomic 操作失败的 g 都会 panic
