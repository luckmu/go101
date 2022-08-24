# reflection

### 空接口与非空接口的 header
```go
// emptyInterface is the header for an interface{} value.
type emptyInterface struct {
    typ *rtype
    word unsafe.Pointer
}
// nonEmptyInterface is the header for an interface value with methods
type nonEmptyInterface struct {
	// see ../runtime/iface.go:/Itab
	itab *struct {
		ityp *rtype // static interface type
		typ  *rtype // dynamic concrete type
		hash uint32 // copy of typ.hash
		_    [4]byte
		fun  [100000]unsafe.Pointer // method table
	}
	word unsafe.Pointer
}
```

## `reflect.TypeOf`
```go
// package reflect
func TypeOf(i any) Type {
    eface := *(*emptyInterface)(unsafe.Pointer(&i))
    return toType(eface.typ)
}
// reflect.Type
t := reflect.TypeOf(T)
t.Kind()
t.Elem() // 1. 获取容器的元素类型, Array, Chan, Map, Pointer, or Slice
         // 2. 指针类型的基类型
```

### method & field(tags)
```go
// 非接口类型，**作用于导出方法**；接口类型无此限制
// reflect.Type.NumMethod
// reflect.Type.Method
// reflect.Type.MethodByName
for i := 0; i < t.NumMethod(); i++ {
    t.Method(i) // 非导出方法不会列出(小写开头)
}

// 作用于结构体所有字段（包括非导出字段）
// reflect.Type.NumField
// reflect.Type.Field
// reflect.Type.FieldByName
for i := 0; i < t.NumField(); i++ {
    t.Field(i)
}
```

```go
type StructTag string
func (tag StructTag) Get(key string) string
func (tag StructTag) Lookup(key string) (value string, ok bool)
```

## `reflect.ValueOf`
reflect.Value

被一个 reflect.Value 值代表的值常称为此 reflect.Value 值的底层值 (underlying value)

***一个结构体的非导出字段不能通过反射修改***

获取 指针所引用的值的 reflect.Value 值

1. 调用此指针值的 reflect.Value 的 Elem 方法
2. 将代表此指针值的 reflect.Value 值传递给一个 reflect.Indirect 函数调用

```go
v := reflect.ValueOf(&T)
vi := reflect.Indirect(v)
ve := v.Elem()

ve.CanSet()
ve.Set...()
```

reflect.Value.Elem 也可以用来获取一个代表一个接口值的动态值的 reflect.Value 值

### reflect.Value 和 原类型的转换
```go
v := reflect.ValueOf(new(int))
vp := v.Interface().(*int) // 0
```

### 其它 `reflect` 的使用

`reflect.MakeFunc`
```go
reflect.MakeFunc(typ Type, fn func(args []Value) (results []Value)) Value {
    // ... implementation
}
// results := fn(args)
// arguments, results 都是 reflect.Value 类型切片
// 认为 arguments 有 typ 相同个数(number)和类型(type), typ 是1个 function
// 所以其使用 like this:
var aFunc func([]int) []int
evf := reflect.ValueOf(&aFunc).Elem()
vmf := reflect.MakeFunc(vf.Elem().Type(), fn)
evf.Set(vmf)
aFunc([]int{1,2,3})
```

`reflect.Value.MapRange`
```go
aMap := map[string]int{}
v := reflect.ValueOf(aMap)
// v.SetMapIndex(key reflect.Value, elem reflect.Value)
for i := v.MapRange(); i.Next; {
    _, _ = i.Key(), i.Value()
}
```
