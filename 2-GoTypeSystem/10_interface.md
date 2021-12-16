# interface

interface: set of *method prototypes*


[implementation](https://gfw.go101.org/article/interface.html#implementation)

arbitary type T (may or may not be interface type), and interface type I, method set of T is a **super set** of I, T implements I

any type implements any blank interface type (as any method set is super set of a blank method set)

[boxing](https://gfw.go101.org/article/interface.html#boxing)

T implements interface I, T can be implicitly converted to I

+ *T 是非接口类型*，T 值的一个 copy 会被包裹在结果（或目标）I 值中。时间复杂度为 O(n), n 为 T 值的尺寸。
+ *T 是接口类型*，当前 T 值包裹的（非接口）值将被复制一份到结果（或目标）I 值中。官方编译器优化，时间复杂度为 O(1)。

non-interface value is boxed in the interface value:

+ non-interface value is **dynamic value** of the interface value
+ non-interface type is **dynamic type** of the interface value

**blank interface & nil interface**

+ nil
    ```go
    // non-nil - (*T, nil)
    var i1 interface{} = (*T)(nil)
    // nil - (nil, nil)
    var i2 interface{}
    // i1 == i2, false
    ```
+ blank
    ```
    type Speaker interface {
        Speak() string
    }
    var blk interface{} // nil & blank interface
    var spk Speaker // nil & not-blank interface
    ```
    