# reflection

```go
// package reflect
func TypeOf(i interface{}) Type {
    eface := *(*emptyInterface)(unsafe.Pointer(&i))
    return toType(eface.typ)
}
```

method & field

```go
// 非接口类型，作用于导出方法；接口类型无此限制
reflect.Type.NumMethod
reflect.Type.MethodByName

// 作用于结构体所有字段（包括非导出字段）
reflect.Type.NumField
reflect.Type.FieldByName
```
tag

```go
type StructTag string
func (tag StructTag) Get(key string) string
func (tag StructTag) Lookup(key string) (value string, ok bool)
```

reflect.Value

被一个 reflect.Value 值代表的值常称为此 reflect.Value 值的底层值 (underlying value)

获取 指针所引用的值的 reflect.Value 值

1. 调用此指针值的 reflect.Value 的 Elem 方法
2. 将代表此指针值的 reflect.Value 值传递给一个 reflect.Indirect 函数调用

reflect.Value.Elem 也可以用来获取一个代表一个接口值的动态值的 reflect.Value 值
