# Go Value Copy Costs

## value size

值的尺寸：直接部分（direct part）在内存中占多少字节，间接部分（indirect part）对尺寸没有贡献。

数组：元素类型尺寸*长度。

结构体：***内存地址对齐***，结构体尺寸必定不小于各字段类型尺寸之和。

## value copy costs

尺寸不大于4个原生字且字段数不超过4个的结构体看作小尺寸值。

对于标准编译器，除大尺寸结构体和数组，其它类型都为小尺寸类型。

***权衡：*** 大尺寸值会使复制代价变高，使用指针替代，指针过多会增加垃圾回收压力。

一般来说，很少使用基类型为***切片类型、映射类型、函数类型、字符串类型和接口类型***的指针，因为复制这些值的代价很小。

Tip: 避免在 `for-range` 时使用双层循环，每个元素将被赋值给第2个循环变量1次。
