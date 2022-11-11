# rip rsp rbp

r-prefix -> 64-bit register

rip(pc register): 下一个指令的地址<br/>
rsp(stack pointer): 始终指向函数调用栈栈顶<br/>
rbp(base pointer): 指向函数栈帧的开始位置

```
A()->B()

> 高地址 <
A 栈帧:
 + A 的局部变量
 + B 的返回值
 + B 的参数
 + B 执行完返回到 A 的地址
B 栈帧:
 + rbp
 + 函数 B 的局部变量
 + rsp (指向整个栈的栈顶, 同时指向 B 栈帧的栈顶)
> 低地址 <
```
