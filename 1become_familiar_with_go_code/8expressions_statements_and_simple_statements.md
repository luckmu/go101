# expressions, statements and simple statements

6 kinds of simple statements:
1. 变量短语句声明
2. 纯赋值语句, 包括 `x op= y` 这种形式
3. 有返回结果的函数或方法调用, 以及通道的接受数据操作
4. 通道的发送数据操作
5. 空语句
6. 自增和自减语句

一些表达式和语句的例子
```golang
// 一些非简单语句:
import "time"
var a = 123
const B = "Go"
type Choice bool
func f() int {
    for a < 10 {
        break
    }
    // 显式代码块
    {
        // ...
    }
    return 567
}

// 一些简单语句例子:
c := make(chan bool)
a = 789
a += 5
a = f()
a++
a--
c <- true
z := <-c

// 一些表达式的例子:
123
true
B
B + "language"
a - 789
a > 0 // 一个类型不确定的布尔值
f     // 一个类型为 func () 的表达式

// 既可视为简单语句, 也可视为表达式
f()
<-c
```
