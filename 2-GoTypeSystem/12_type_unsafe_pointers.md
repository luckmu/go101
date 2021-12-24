# type unsafe pointers

+ *uintptr* 和 *unsafe.Pointer* 互相转换; *unsafe.Pointer* 和*其他类型指针*互相转换
+ `unsafe.Add(ptr Pointer, len IntegerType) Pointer` 等效于运算 `uintptr`
+ `reflect.StringHeader` & `reflect.SliceHeader`
+ uintptr(unsafe.Pointer(&v))，若不注意 v 的生命周期，很容易导致其被 gc 后使用这个无效的整型地址

```go
type Pointer *ArbitraryType
```

+ **a pointer can be explicitly converted to an unsafe pointer, and vice versa**.
+ **an uintptr value can be explicitly converted to an unsafe pointer, and vice versa**. but a nil unsafe.Pointer shouldn't be converted to uintptr and back with arithmetic

facts

1. 非类型安全指针值是指针但uintptr是整数
    + 每一个非零安全或不安全值都引用着另一个值，但是uintptr被看作一个整数，尽管它存储的是一个地址的数字表示。
2. 不再使用的内存块的回收时间点是不确定的
3. 一个值的地址在程序运行中可能改变
    + 当协程的栈大小改变时，开辟在此栈上的内存块需要移动，从而相应值的地址会改变
4. 一个值的生命范围可能并没有代码中看上去的大
    ```go
    // 例子
    n := new(int)
    p := uintptr(unsafe.Pointer(n))
    // 不能保证内存块没有释放掉
    *(*int)(unsafe.Pointer(p)) = 3
    ```
5. *unsafe.Pointer 是一个类型安全指针类型

如何正确使用非类型安全指针

+ pattern1. 将类型 *T1 的一个值转换为非类型安全指针值，然后将非类型安全指针值转换为类型 *T2
    ```
    // 例1.
    // 内存中每个位bit都不变
    var f float64 = 123.456
    var u uint64 = *(*uint64)(unsafe.Pointer(&f))
    // u: 4638387860618067575
    // uint64(f): 123
    // 例2.
    type mystring string
    ms := []mystring{"hello", "jack"}
    // ss := ([]string)ms // 编译错误
    ss := *(*[]string)(unsafe.Pointer(&ms))
    ss[1] = "rose"
    // ms: hello rose
    // ms = ([]mystring)ss // 编译错误
    ms = *(*[]mystring)(unsafe.Pointer(&ss))
    
    // 字节切片 to 字符串
    *(*string)(unsafe.Pointer(&ByteSlice))
    *(*[]byte)(unsafe.Pointer(&String)) // 危险，字符串尺寸比字节切片尺寸小
    // unsafe.Sizeof
    ```
+ pattern2. 将一个非类型安全指针值转换为一个uintptr值，然后使用此uintptr值
    ```go
    // 此模式不是很有用，一般将转换结果uintptr输出到日志来调试，但有很多其他安全且简洁途径也可实现此目的
    fmt.Printf("%x\n", uintptr(unsafe.Pointer(&v)))
    fmt.Printf("%p\n", &v)
    ```

+ pattern3. 将一个非类型安全指针转换为一个uintptr值，然后此uintptr值参与各种算术运算，再将算术运算的结果uintptr值转回非类型安全指针
    ```go
    type T struct {
        x bool
        y []int
    }
    const offset = unsafe.Offsetof(T{}.y)
    const elesz = unsafe.Sizeof(T{}.y[0])
    var t T = T{y: []int{1,2,3}}
    ty2 := (*int)(unsafe.Pointer(uintptr(unsafe.Pointer(&t))+offset+2*elesz))
    fmt.Printf("%v\n", ty2) // 3
    ```
+ pattern4. 将非类型安全指针值转换为uintptr值并传递给syscall.Syscall函数调用
    ```go
    // 此处保证传入地址在函数中不改变，编译器行为
    syscall.Syscall(SYS_READ, uintptr, uintptr, uintptr)
    ```
+ pattern5. 将 reflect.Value.Pointer 或者 reflect.Value.UnsafeAddr 方法的 uintptr 返回值立即转换为非类型安全指针

+ pattern6. 将一个 **reflect.SliceHeader** 或者 **reflect.StringHeader** 值的 Data 替换为非类型安全指针，以及其逆转换
    ```go
    func main() {
        // ...
        var hdr reflect.StringHeader
        hdr.Data = uintptr(unsafe.Pointer(new([5]byte)))
        // ... 新开辟数组已经没有引用，可以被回收
        hdr.Len = 5
        s := *(*string)(unsafe.Pointer(&hdr)) // 不安全的！
    }

    func ByteSlice2Str(bs []byte) (str string) {
        // 在参数和返回值处保证对象的引用
        slicehdr := (*reflect.SliceHeader)(unsafe.Pointer(&bs))
        strhdr := (*reflect.StringHeader)(unsafe.Pointer(&str))
        strhdr.Data = slicehdr.Data
        strhdr.Len = slicehdr.Len
        return
    }
    ```
