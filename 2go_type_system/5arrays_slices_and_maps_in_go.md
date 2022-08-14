# arrays, slices and maps

```golang
// 空切片和 nil 切片
_ = []T{} // 表示类型 []T 的1个空切片,
_ = []T(nil) // 和这个表达式是不等价的

// 容器类型: map, array, slice
// 各种类型容器赋值是否共享底层元素?
// 映射元素不能直接取地址: `&aMap[k]` is invalid
// copy(dst, src)
// for-range aContainer; 创建 aContainer 副本代替遍历, 不同 container 副本在遍历时的表现不同
// `memclr` 编译器优化数组, 切片清零
for i := range a {
    a[i] = t0
}
// 也就是说, 在增添元素时 append() 可能多次引发申请内存的操作, 应该具体情况考虑其使用
```

**空切片和`nil`切片。**

**内嵌组合字面量可以被简化, 可简化为`{...}`, 取地址操作符`&`有时也可以省略。**
```golang
type LangCategory struct {
    dynamic bool
    strong  bool
}
var _ = map[LangCategory]map[string]int{
    {true, true}: {
        "Python": 1991,
        "Erlang": 1986,
    },
    {true, false}: {
        "JavaScript": 1995,
    },
    {false, true}: {
        "Go":  2009,
        "Rust": 2010,
    },
    {false, false}: {
        "C": 1972,
    },
}
```

**容器赋值。**
1. 映射: 赋值语句执行完毕, 目标映射值和源映射值***共享底层元素***, 向其中添加、删除元素将体现在另一个映射中。
2. 切片: 1个切片赋值给另1个切片后, 它们将共享底层元素, 长度和容量也相等。但是和映射不同, ***如果其中1个切片改变了长度或容量, 此变化不会体现在另1个切片中***。
3. 数组: 当1个数组被赋值给另1个数组, 所有元素都将从源数组复制到目标数组。赋值完成后, ***两个数组不共享任何元素***。
```golang
// 重点是切片的赋值
s0 := []int{0, 2}
s1 := s0
s1[0] = 2 // s0's cap: 2; s1's cap: 2
// 赋值并修改, s0, s1 底层数组没有变化
hdr0 := (*reflect.SliceHeader)(unsafe.Pointer(&s0))
hdr1 := (*reflect.SliceHeader)(unsafe.Pointer(&s1))
// bp0 := (*[2]int)(unsafe.Pointer(hdr0.Data))
// bp1 := (*[2]int)(unsafe.Pointer(hdr1.Data))
// %x 打印 s0, s1 的地址, 即 hdr0.Data, hdr1.Data
fmt.Printf("%x %x", hdr0.Data, hdr1.Data)
s1 = append(s1, 0) // s0's cap: 2; s1's cap: 4
// (*[4]int)(unsafe.Pointer(hdr1.Data))
hdr0 = (*reflect.SliceHeader)(unsafe.Pointer(&s0))
hdr1 = (*reflect.SliceHeader)(unsafe.Pointer(&s1))
// 此时 s0 和 s1 底层数组 address 已经不同,
// 因为原 cap, len 都为 2, s1 append 后, len 为 3, cap 为 4,
// 重新申请了1块大小为4的数组, 并将前 len 个元素 copy 到新的底层数组中
fmt.Printf("%x %x", hdr0.Data, hdr1.Data)
```

**_slice 的 `make()` 声明方式。**
```golang
make(S, length, capacity)
make(S, length) // <=> make(S, length, length)
```

**`new()` 创建容器值(没有太大价值)。**
```golang
m := *new(map[string]int) // <=> var m map[string]int
fmt.Println(m == nil)     // true
s := *new([]int)          // <=> var s []int
fmt.Println(s == nil)     // true
a := *new([5]bool)        // <=> var a [5]bool
fmt.Println(a == nil)     // true
```

***映射元素不可取地址。***
```golang
// p := &map[key]val, 非法表达
// 合法表达
val := map[key]
p := &val

// 2 个原因
// 1. hashtable 原地 or double 扩容, 映射中元素地址会改变, 
//   如果映射元素可以取地址, Go运行时**必须修改所有已经存储了 `&amap[k]` 的指针值**,
//   增大Go编译器、运行时的实现难度, 影响程序的运行效率
// 2. &amap[k].Modify()
```

**映射元素不能部分修改(元素类型为数组or结构体); 所以一般情况元素值用指针形式即可。**
```golang
type T struct{age int}
mt := map[string]T{
    "John": {20},
}
// mt["John"].age = 10, "invalid expression"
// 用以下临时变量的形式
t := mt["John"]
t.age = 10
mt["John"] = t
// 数组同理, at := map[int][1]int{1:{1}}
// 元素值用指针形式
mpt := map[string]*T{"John":{20}}
mpt["John"].age = 30 // "valid expression"
```

**从数组或切片派生切片(取子切片)**
```golang
baseContainer[low : high] // <=> baseContainer[low : high : cap(baseContainer)]
baseContainer[low : high : max]
// 派生切片长度为 `high-low`, 容量为 `max-low`
```

**切片转化为数组指针**
```golang
sli := make([]int, 3, 5)
arr := (*[3]int)(sli) // 数组长度小于等于切片长度
_ = arr
```

**`copy` 复制切片元素**
```golang
dst := []int{1, 2, 3, 4}
src := []int{5, 6, 7}
copy(dst, src)
// copy 有效长度为 dst 和 src 中 len 的较小值
```

**遍历容器元素**
```golang
for key, element := range aContainer {
    // use key and element...
}
```
***`for-range` 的 3 个重要事实***
1. 被遍历容器是 `aContainer` 的1个副本 **(只有`aContainer`的直接部分被复制了)**。此副本是匿名值, 所以是无法修改的。</br> 如果`aContainer`是数组, 对元素的修改不会体现在循环变量中, 数组的副本不共享任何元素。</br> 如果`aContainer`是切片(或映射), 对元素的修改会体现在循环变量中, 切片(或映射)共享元素。
2. 每个循环步, `aContainer` 副本中的1个键值对将被赋值(复制)给循环变量, 对循环变量直接部分的修改不会体现在`aContainer`对应元素中。***(`for-range`是遍历映射`map`的唯一途径, 最好不要使用大尺寸的映射键值和元素类型, 避免较大的复制负担。)***
3. 所有被遍历的键值对将被赋值给**同一对**循环变量实例。

```golang
// container 是数组, 元素修改不体现在循环变量中, 即修改后面的元素也不会影响遍历的结果
a := [...]int{0, 1}
for k, v := range a {
    a[k] = 2
    fmt.Println(v, a[k])
}
// 0 2
// 1 2
// 怎么理解? 
//   _slice 和 _array 在 for-range 时会生成`相应容器的副本`来遍历(仅 copy 直接部分)
//   *如果一定要遍历数组(复制指针, 切片的代价很小):
//    1. 下标方式 `for i := 0; i < len(a); i++`
//    2. 指针方式 `for i, v := range &a`e
//    3. 切片方式 `for i, v := range a[:]`
```

**数组(切片)清零, 内部memclr调用**
```golang
// 在官方标准编译器中,
// 以下单循环变量for-range将会优化为1个内部的memclr调用
for i := range a {
    a[i] = t0
}
// a为类型T的数组或切片;
// t0位类型T的零值
```

**修改切片的长度或容量**
```golang
s := make([]int, 2, 6)
reflect.ValueOf(&s).Elem().SetLen(3)
reflect.ValueOf(&s).Elem().SetCap(5)
```

**切片克隆, 删除1段(1个)元素**
```golang
// 切片克隆
// 这样当 s == nil 时, sClone 不会是 (nil)(*[]T)
var sClone []T
if s != nil {
    sClone = make([]T, len(s))
    copy(sClone, s)
}

// 删除1段元素
s = append(s[:from], s[to:]...)
// 切片元素可能引用其它值, 重置因为删除元素多出来槽位上的元素值, 避免暂时性的内存泄漏
temp := s[len(s):len(s)+to-from]
// memclr()
for i := range temp {
    temp[i] = t0
}

// 条件性删除切片元素
func DeleteElems(s []T, keep(T) bool, clear bool) []T {
    result := s[:0] // 不需要重新开辟内存
    for _, v := range s {
        if keep(v) {
            result = append(result, v)
        }
    }
    // 返回值不是入参中的 s, 而是1个新切片(底层数组相同)
    // clear 表示原切片是否需要做 memclr()
    if clear {
        temp := s[len(result):]
        for i := range temp {
            temp[i] = t0
        }
    }
    return result
}
```
