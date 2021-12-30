# panic and recover use cases

1. 避免恐慌导致程序崩溃

    ```go
    defer func() {
        if e := recover(); e != nil {
            log.Printf("recover from crash: %v\n", e)
        }
    }()
    panic("unknown err")
    ```
2. 自动重启因为恐慌而退出的协程

    ```go
    func NeverExit(name string, f func()) {
        defer func() {
            if e := recover(); e != nil {
                log.Printf("goroutine %s crashed, restart\n", name)
                go NeverExit(name, f)
            }
        }()
        f()
    }
    ```
3. 使用 panic/recover 模拟长程跳转
4. 使用 panic/recover 用来减少错误检查代码（并不推荐）

    ```go
    func doSomething() (err error) {
        defer func() {
            err = recover()
        }()
        
        doStep1()
        doStep2()
        doStep3()
        doStep4()
        doStep5()

        return
    }

    func doStepN() {
        // ...
        if err != nil {
            panic(err)
        }
        // ...
        if done {
            panic(nil)
        }
    }
    ```
