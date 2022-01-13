# Memory Leaking Scenarios

## Kind-of Memory Leaking Caused by Substrings

```go
s := "google"

si := s[:2]
fmt.Printf("si: %s\n", si)

// si & sii have the same underlying part (bytes slice)
// si hasn't been collected, then underlying part of sii won't be collected 
// though sii may just use part of si
sii := si
fmt.Printf("sii: %s\n", sii)

// use following 2 good ways
// `stirngs.Repeat` (implemented by `strings.Builder`)
siii := strings.Repeat(si, 1)
fmt.Printf("siii: %s\n", siii)

// `strings.Builder`
builder := strings.Builder{}
builder.WriteString(si)
siiii := builder.String()
fmt.Printf("siiii: %s\n", siiii)
```

## Kind-of Memory Leaking Caused by Subslices

```go
// just like scenario in `string` (above)
// avoid sharing the same underlying part (array)
func copys(s []int) {
    si := make([]int, 30)
    copy(si, s[:30])
}
```

## Kind-of Memory Leaking Caused by Not Resetting Pointers in Lost Slice Elements

```go
func h() []*int {
    s := []*int{new(int), new(int), new(int), new(int), new(int)}
    return s[1:3:3]
}

func hi() []*int {
    s := []*int{new(int), new(int), new(int), new(int), new(int)}

    // reset pointer values
    s[0], s[len(s)-1] = nil, nil
    return s[1:3:3]
}
```


## Real Memory Leaking Caused By Hanging Goroutines

There're two reasons why go runtime won't kill hanging goroutines.
+ sometimes it's hard for go runtime to judge whether or not a blocking goroutine will be blocked forever.
+ sometime we deliberately make a goroutine hanging. e.g., sometimes we may let the main goroutine of a go program hang to avoid the program exiting.

## Real Memory Leaking Caused by Not Stopping `time.Ticker` Values Which Are Not Used Anymore

`time.Timer` will be garbage collect after some time, but should stop a `time.Ticker` when it is not used anymore.

## Real Memory Leaking Caused by Using Finalizer Improperly

```go
type T struct {
    v [1<<20]int
    t *T
}

var finalizer = func(t *T) {
    fmt.Println("finalizer called")
}

var x, y T

// x escapes to heap
runtime.SetFinalizer(&x, finalizer)
// this causes x and y are not collectable
x.t, y.t = &y, &x // y also escapes to heap
```

## Kind-of Resource Leaking by Deferring Function calls

see this [article](https://go101.org/article/defer-more.html#kind-of-resource-leaking)
