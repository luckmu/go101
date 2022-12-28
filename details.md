# details

## 两个类型最终享有相同的底层类型，可以多次、间接地互相转换类型。
```go
type Ti int64
type Ta *int64
type Tb *Ti
```

## 单项接收通道不能关闭
```go
var ch = make(<-chan struct{})
close(ch)
// invalid operation: cannot close receive-only channel ch
```

## `reflect.DeepEqual`

## os.IsNotExist() && os.ErrNotExist
using `errors.Is(err, os.ErrNotExist)`

## flag
+ `-flag`: only for bool type
+ `-flag=x`: any type
+ `-flag x`: non-bool types
