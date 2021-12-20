# type unsafe pointers

```go
type Pointer *ArbitraryType
```

+ **a pointer can be explicitly converted to an unsafe pointer, and vice versa**.
+ **an uintptr value can be explicitly converted to an unsafe pointer, and vice versa**. but a nil unsafe.Pointer shouldn't be converted to uintptr and back with arithmetic

facts

1. unsafe pointers are pointers and uintptr values are integer
2. unused memory blocks may be collected at any time
3. the addresses of some values might change at runtime
4. the life range of a value at runtime may be not as large as it looks in code
5. *unsafe.Pointer is a general safe pointer type
