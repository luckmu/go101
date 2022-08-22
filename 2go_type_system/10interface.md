# interface

+ Part 1 
  + 接口? 方法集? 方法描述(method specification)?
+ Part 2
  + 基本接口类型 & 非基本接口类型
  + `nil`接口 & 包裹`nil`值的接口
  + 空接口 & 非空接口的内部结构? 动态类型? 动态值?
+ Part 3
  + 接口实现, 接口值包裹
+ Part 4: ***多态 & 反射***
  + ***多态***: 形参为接口类型 & 实参为任意实现了该接口类型的类型(最终落实到非接口类型)
  + ***反射***: 断言(`a.(T)`) & `type-switch`

[1. 接口类型](#interface-type)

[2. 基本接口类型](#basic-interface-types)

[3. 实现和值包裹](#implementation--value-boxing)

[4. 多态和反射](#polymorphism--reflection)

## ***interface type***
接口类型(interface type)内嵌若干接口元素(interface elements)

接口元素:
+ 方法元素(method element): 方法描述(method specification); 接口类型中的方法描述不能用空标识符`_`命名
+ 类型元素(type element): 类型名称, 类型字面量形式, 近似类型, 类型并集

将1个接口类型内嵌到另1个接口类型中, 等价于将前者元素(递归, 平铺)展开放入后者
```golang
type ReadWriteCloser = interface {
    Read(buf []byte) (n int, err error)
    Write(buf []byte) (n int, err error)
    error
    interface{ Close() error }
}

// is equal to
interface {
    Read(buf []byte) (n int, err error)
    Write(buf []byte) (n int, err error)
    Error() string
    Close() error
}
```

## ***basic interface types***

基本接口类型指可以用作值类型的接口类型, 一个非基本接口类型只能用作(**自定义泛型中使用的**)约束接口类型(即类型约束)
> Basic interface types are the interface types which may be used as value types. A non-basic interface type is also called a constraint-only interface type.

```golang
// e.g.
// A basic interface.
type aBasicInterface interface {
    Read(buf []byte)  (n int, err error)
    Write(buf []byte) (n int, err error)
    Error() string
    Close() error
}

// A non-basic interface.
type Unsigned interface {
    uint | uint8 | uint16 | uint32 | uint64 | uintptr
}
```


### *`nil`接口值 & 包裹`nil`值的接口*
```golang
a := (interface{})(nil) // equal to nil
b := (interface{})((*int)(nil)) // not equal to nil

_ = a == nil // true
_ = b == nil // false
```
### *`dynamic type` & `dynamic value`*
```go
// blank interface
type _interface struct {
    dynamicType *_type
    dynamicValue unsafe.Pointer
}
// non-blank interface
type _interface struct {
    dynamicTypeInfo *struct {
        dynamicTYpe *_type
        methods []*_function
    }
    dynamicValue unsafe.Pointer
}
```

## ***implementation & value boxing***

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

non-interface value is boxed into an interface value, Go 运行时分析两个值类型的实现关系，并将关系存储到这个接口值内。每一对这样的类型，实现关系信息最多构建一次，会被缓存在内存的一个全局映射中。所以全局映射中的条目永不减少。事实上，一个非零接口值在内部只是使用一个指针字段来引用着此全局映射中的一个实现关系信息条目。

对于一个非接口和接口类型对，实现关系包括两部分内容：

+  动态类型（此非接口类型）的信息
+ 一个方法表（切片类型），存储了所有此接口类型指定的并且为此非接口类型（动态类型）声明的方法

## ***polymorphism & reflection***

[1. polymorphism](https://gfw.go101.org/article/interface.html#polymorphism)

调用一个接口值的方法实际上将调用此接口值的动态值对应的方法，一个接口值通过包裹不同的动态类型的动态值来表现出不同的行为，称为多态。

[2. reflection](https://gfw.go101.org/article/interface.html#reflection)

一个接口值中存储的动态类型信息可以被用来检视此接口值的动态值和操作此动态值所引用的值，这称为反射。

Go 内置反射机制：断言（type assertion）和 type-switch 流程控制

+ compile-time check (src type must implement dst type, dst = src)
  + inter = non-inter
    + non-inter implement inter
  + inter1 = inter2
    + inter2 implement inter1
+ runtime assert
  + inter.(non-inter)
    + non-inter must implement inter
  + inter1 = inter2
    + inter2 may not implement inter1, dynamic value of inter2 may implement inter1

i.(T)

+ T is non-interface type, dynamic value of i is the T, success otherwise failed. result is **a copy of dynamic value**.
+ T is interface type, dynamic value exists and the value implements T, success otherwise failed. result is **copy of T which boxing the dynamic type of i**.

type-switch

```go
switch aSimpleStatement; v := x.(type) {
case TypeA:
    ...
case TypeB:
    ...
case nil:
    ...
default:
    ...
}
```
