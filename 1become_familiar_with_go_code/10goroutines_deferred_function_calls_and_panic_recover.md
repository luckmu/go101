# goroutines deferred function calls and panic recover

## 几个数字进制

```golang
// 0xF       // 16
// 15        // 10
// 0o17, 017 // 8
// 0b1111    // 2
```

一个rune表示一个Unicode码点
单引号中字符序列表示一个Unicode码点值

引出
+ 单字节编码(e.g. standard ASCII), 8bit, 第1位是0, 7位表示字符, 共2^7个字符
+ 多字节编码(e.g. BIG-5), 16bit
+ 统1编码Unicode: utf8(以字节为单位编码Unicode), utf16(以16位无符号整数为单位), utf32(以32为无符号整数为单位)

```golang
fmt.Println('a', '\141', '\x61', '\u0061', '\U00000061')
// \ + 3 个8进制表示1个byte值
// \x + 2 个16进制数字表示1个byte值
// \u + 4 个16进制表示1个rune值
// \U + 8 个16进制表示1个rune值
fmt.Printf("%x\n", '心') // 5fce
fmt.Println(rune('\u5fc3')) // 心
```

`runtime.NumCPU()`: 打印当前当前程序可利用逻辑CPU数目

## goroutine schedule

+ Create
+ Running
  + waiting
  + executing
    + (sleep/syscall/...)
    + blocking
+ Blocking
+ Exit

## MPG
+ `M - machine`: 系统线程
+ `P - process`: 逻辑处理器(并非逻辑CPU)
+ `G - goroutine`: 协程

`runtime.GOMAXPROCS`: 最大逻辑处理器(process)

默认 `runtime.GOMAXPROCS` 等于 `runtime.NumCPU`, 对于某些文件操作频繁的程序, 设置1个大于 `runtime.NumCPU` 的 `GOMAXPROCS` 可能更好(文件IO会blocking)

## 协程和延迟函数实参的估值时刻
+ 1个延迟函数的实参是在此调用对应的延迟调用语句被执行时被估值的, 或者说**是在此延迟函数被推入延迟调用栈时被估值的**, 这个估值的结果在延迟调用执行时使用
+ **匿名函数体内的表达式在此函数执行时候才会估值**, 不管是被普通调用还是延迟/协程调用

```go
// 延迟函数实参入栈时估值
// 结果: 2,1,0
func() {
  for i := 0; i < 3; i++ {
    defer fmt.Println("a:", i)
  }
}()

// 匿名函数在执行时才会估值
// 结果: 3,3,3
func() {
  for i := 0; i < 3; i++ {
    defer func() {
      fmt.Println("b:", i)
    }()
  }
}
```

## 恐慌和恢复 panic & recover

```go
func panic(v interface{})
func recover() interface{}
```
1个`recover`函数的返回值为其所恢复的恐慌在产生时被1个`panic`函数调用所消费的参数
```golang
{
  defer func() {
    v := recover()
    fmt.Println(v)
  }()
  panic("errmsg")
}
// 会捕获这个 panic 并打印 "errmsg"
```
