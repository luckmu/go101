# function

**function signature**: 2 type list, input parameter list + output result type list

**function type** literal: `func` keyword + function signature literal

~~can think function type and function signature as the same concept~~

**function prototype**: function name + function type (or signature); a function prototype is a function declaration without the body portion

**function declaration**: function prototype + function body

~~the result of calls to custom function can be discarded, not true for calls to some built-in functions (except `recover` and `copy`)~~

## function calls as expressions

+ single result function: can always be used as a single value
+ multi-result function: can only be used as a multi-value expressions in 2 scenarios
    1. Used in **assignment as sources values**. But can't mix with other source values in the assignment.
    2. **Nested in another function call as arguments**. But can't mix with other arguments.

~~for the standard Go compiler, some [built-in functions break the universality](https://go101.org/article/exceptions.html#nest-function-calls) of just described rule above.~~

## function values

~~declaring a function is just like declaring a variable  (funcname = varname), but the funcvar is immutable~~

Attention: **built-in functions & `init` function cannot be used as values**

When we declare a custom function, we also declared an **immutable** *function value*. The function value is identified by the function name. The function type is the literal by omitting the function name from the function prototype literal.

When one function value is assigned to another, the two functions **share the same underlying parts(s)**. The 2 functions represent the same internal function object. *The effects of invoking 2 functions are the same*.

```go
func Double(n int) in {
    return n+n
}

func Apply(n int, f func(n int) int) int {
    return f(n)
}

func main() {
    fmt.Printf("%T\n", Double) // func(int) int
    // Double = nil // error: Double is immutable

    var f func(n int) int // default value is nil
    f = Double
    g := Apply
    fmt.Printf("%T\n", g) // func(int, func(int) int) int
    f(9)         // 18
    g(6, Double) // 12
    Apply(6, f)  // 12
}
```

*Any function value can be invoked just like a declared function*. It is fatal error to call a nil function to start a **new goroutine** (**unrecoverable**). Calls to nil function will produce recoverable panics, including **deferred** function calls.

e.g.,

+ go nilFunc() -> unrecoverable crash
+ nilFunc() || defer nilFunc() -> recoverable crash

in practice, often **assign anonymous functions to function variables**, so that we can call the anonymous functions multiple times
