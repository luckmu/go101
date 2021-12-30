# defer more

1. 很多有返回值的内置函数（built-in functions）是不能延迟调用的

    ```go
    // 一个可能的场景 defer append
    // 使用匿名函数实现
    defer func() {
        sli := append(sli, eles...)
    }()
    ```
2. **延迟调用函数值的估值时刻**

    一个被延迟调用的函数值是在其调用被推入延迟调用堆栈之前被估值的。
    ```go
    var f func() // f == nil
    defer f() // panic
    f = func() {} // 不会阻止恐慌产生
    ```
3. 延迟方法调用的属主实参的估值时刻

    方法的属主实参也是在其调用被推入延迟调用堆栈之前被估值的。
    ```go
    type T int
    func (t T) M(n int) T {
        print(n)
        return t
    }

    func main() {
        var t T
        // t.M(1) 是 M(2) 的属主实参
        // 在 M(2) 调用被延迟推入调用堆栈之前被估值
        defer t.M(1).M(2)
        t.M(3).M(4)
        // 1342
    }
    ```
4. 延迟调用使得代码更简洁和鲁棒

    ```go
    fp, err := os.Open(filepath)
    if err != nil {
        return err
    }
    // 只用在这里调用
    // 之后所有文件操作都不用在失败后调用 fp.Close()
    defer fp.Close()
    ```
    ```go
    var m sync.Mutex
    // obviously, f1 is better
    func f1() {
        m.Lock()
        defer m.Unlock()
        doSomething()
    }
    func f2() {
        m.Lock()
        doSomething()
        m.Unlock()
    }
    ```
5. **延迟调用导致暂时性内存泄漏**

    ```go
    for _, file := range files {
        fp, err := os.Open(file.Path)
        if err != nil {
            return err
        }
        defer fp.Close()
    }
    // 缩小 defer 的声明周期
    for _, file := range files {
        if err := func() error {
            fp, err := os.Open(file.Path)
            if err != nil {
                return err
            }
            defer fp.Close() // 循环内就释放 fp
        }(); err != nil {
            return err
        }
    }
    ```
