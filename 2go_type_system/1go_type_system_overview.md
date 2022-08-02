# go type system overview

## Basic types (predeclared type)
+ 内置字符串类型: `string`
+ 内置布尔类型: `bool`
+ 内置数值类型:
  + `int8`, `uint8(byte)`, `int16`, `uint16`, `int32(rune)`, `uint32`, `int64`, `uint64`, `int`, `uint`, `uintptr`
  + `float32`, `float64`
  + `complex64`, `complex128`

## Composite types
+ 指针类型 - 类C指针
+ 结构体类型 - 类C结构体
+ 函数类型 - 函数类型在Go中是1种1等公民类别
+ 容器类型:
  + 数组类型 - 定长容器类型
  + 切片类型 - 动态长度和容量容器类型
  + 映射类型(`map`) - 也称字典类型, 在标准编译器中是 hash 表实现的
+ 通道类型 - 通道用来同步并发的协程
+ 接口类型 - 接口在**反射和多态**中发挥重要角色

2种类型声明形式:
+ 类型定义(type definition): `type boolean bool`
+ 类型别名(type alias): `type boolean = bool`
```golang
type NewTypeName SourceType
type (
    NewTypeName1 SourceType1
)
```
