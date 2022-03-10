# method

T, *T **constraints**

+ T must be a **defined type**
+ T and methods are in the **same package**
+ T **cannot be a pointer type**
+ T **cannot be an interface type**

> T & *T **receiver type** of methods;
T **receiver base type** of methods


[method as function](https://gfw.go101.org/article/method.html#method-as-function)

in a way, *method* is just *function*, compiler will declare coressponding functions for user 
defined methods

[implicit pointer methods](https://gfw.go101.org/article/method.html#implicit-pointer-methods)

+ for value receiver, compiler will declare 2 implicit functions and 1 implicit method
+ for pointer receiver, compiler will declare 1 implicit function and 1 implicit method

```go
// user defined value receiver method
func (t T) Method(...) ... {}
// compiler implicitly declared 2 functions
func T.Method(t T, ...) ... {}
func (*T).Method(t *T, ...) ... {}
// t.Method(...)
//   = T.Method(t, ...)
//   = (*T).Method(&t, ...)
// compiler implicitly declared 1 method
func (t T) Method(...) ... {return T.Method(t)}

// user defined pointer receiver method
func (t *T) Method(...) {}
// compiler implicitly declared 1 function
func (*T).Method(t *T, ...) {}
// (*T).Method(&t, ...)
//   = (&t).Method()
//   = t.Method() // syntactic sugar
// compiler implicitly declared 1 method
func (t *T) Method(...) {(*T).Method(t, ...)}
```

[method set](https://gfw.go101.org/article/method.html#method-set)

method prototype: `Method(...) ...`

method decl. = `func` + receiver decl. + **method prot.** + function body

method set: *set of **method prototypes***

[call](https://gfw.go101.org/article/method.html#call)

*method is special function, called member function*

类型的零值也有和方法同名的成员函数，注意 pointer receiver 的成员函数解引用时会 panic

value receiver is a *shallow copy*; while pointer receiver is a *deep copy*

[method value normalization](https://gfw.go101.org/article/method.html#method-value-normnalization)

+ v T
  + v.m -> (&v).m (pointer receiver, *implicit function*)
  + v.m -> v.m (value receiver)
+ p *T
  + p.m -> (*p).m (value receiver, *syntactic sugar*)
  + p.m -> p.m (pointer receiver)
