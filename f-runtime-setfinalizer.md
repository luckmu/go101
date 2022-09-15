# f: runtime.SetFinalizer

[explanation of go101](https://gfw.go101.org/article/unofficial-faq.html#finalizers)

**类似析构函数，但不能当作析构函数使用**: 忘记调用 `Close(), Stop()` 的保险（参考 `os.NewFile()` 中 `runtime.SetFinalizer(f.file, (*file).close)`），稳定的使用还是 `defer f.Close()`，而不能只依赖于 `runtime.SetFinalizer()` 来释放资源。

> The finalizer is scheduled to run at some ***arbitrary time*** after the program can no longer reach the object to which obj points. There is ***no guarantee that finalizers will run before a program exits***, so typically they are ***useful only for releasing non-memory resources*** associated with an object during a long-running program. For example, an os.File object could use a finalizer to close the associated operating system file descriptor when a program discards an os.File without calling Close, but it would be a mistake to depend on a finalizer to flush an in-memory I/O buffer such as a bufio.Writer, because the buffer would not be flushed at program exit.  - [SetFinalizer](https://pkg.go.dev/runtime#SetFinalizer)

**Example**:
```go
// 不能依赖这个步骤将 data 保存，程序 exit 也不能保证 finalizer 运行
type atype struct {
    f *afield
}

func NewAtype() *atype {
    t := &atype{
        f: &afield{},
    }
    runtime.SetFinalizer(t.f, (*afield).stop)
    return t
}

type afield struct{}

func (f *afield) stop() {
    // ... releasing some non-memory resources
    runtime.SetFinalizer(f, nil)
}
```
