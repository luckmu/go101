# memory layout

## Alignment Guarantee

+ 结构体 field：此类型的字段对齐保证（struct）
+ 其它：此类型的一般对齐保证（variable declaration，array element type, etc）

In fact, each type has two alignment guarantees.
+ field alignment guarantees: one is for when it is used as field types of other (struct) types
+ general alignment guarantees: the other is for other cases (when it is used for variable declaration, array element type, etc)

**calls to functions in `unsafe` are always evaluated at compile time**
+ compile time, `unsafe.Alignof(t)` for general alignment guarantee; `unsafe.Alignof(x.t)` for field alignment guarantee
+ run time, `reflect.Typeof(t).Align()` for general alignment guarantee, `reflect.Typeof(t).FieldAlign()` for field alignment guarantee

go specification mentions [`a little on type alignment guarantees`](https://go.dev/ref/spec#Size_and_alignment_guarantees):

> 1. For a variable x of any type: unsafe.Alignof(x) is at least 1.
>
> 2. For a variable x of struct type: unsafe.Alignof(x) is the largest of all the values unsafe.Alignof(x.f) for each field f of x, but at least 1.
>
> 3. For a variable x of array type: unsafe.Alignof(x) is the same as the alignment of a variable of the array's element type.

## Structure Padding

```go
// alignment (max field size)
// top -> bottom: small -> big

type T1 struct {
    a int8  // 1 + 7
    b int64 // 8
    c int16 // 2 + 6
} // size -> 24

type T2 struct {
    a int8  // 1 + 1
    c int16 // 2 + 4
    b int64 // 8
} // size -> 16
```
