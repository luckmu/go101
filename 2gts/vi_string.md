# string

```go
type _string struct {
    elements *byte // underlying bytes
    len int // number of bytes
}
```

len(s) is a constant expression, whereas len(s[:]) is not; lead to different evaluation timing

## string, []byte, []rune

2 string related conversions (string -> []byte, string -> []rune):

1. A string value can be explicitly converted to a byte slice, and vice versa. A byte slice's underlying type is `[]byte` (a.k.a., `[]uint8`)
2. A string value can be explicitly converted to a rune slice, and vice versa. A rune slice's underlying type is `[]rune` (a.k.a., `[]int32`)

## string, []byte: deep copy

string <-> []byte is a **deep copy**, because eles in []slice is mutable but eles in string is immutable.

string <-> []byte, compiler optimizations

+ string -> []byte, follows the `range` in `for-range` loop (e.g., `for i, b := range []byte(str)`)
+ []byte -> string, which is used as **map key in map element retrieval** indexing syntax (e.g., `m[string([]byte{'k'})]`)
+ []byte -> string, which is used in **comparison** (e.g., `if string([]byte{'x'}) != string([]byte{'y'})`)
+ []byte -> string, which is used in string concatenation, and **at least one** of concatenation values is a **non-blank string constant** (e.g., `s := (" " + string([]byte{1023: 'x'} + string([]byte{1023: 'y'})))[1:]`)

## []byte, []rune

Go does not support []byte <-> []rune:

1. []byte <-> string <-> []rune, convenient but *not efficient* (2 deep copies are needed)
2. unicode/utf8
    + utf8.EncodeRune(), rune -> []byte
    + utf8.DecodeRune(), []byte -> rune
3. bytes.Runes(), []byte -> []rune

## for-range a string

+ iterate the Unicode code points (as `rune` values), instead of bytes (**iterate runes**)

    ```go
    for i, rn := range s {}
    ```

+ 2 ways **iterate bytes**

    ```go
    var b byte
    for i := 0; i < len(s) {
        b = s[i]
    }
    // doesn't need a deep copy
    // more effective
    for i, b := range []byte(s) {}
    ```

len(s), for-range string-type, utf8.RuneCountInString, len([]rune(s)) -> all O(n)

## string concatenation

+ Sprintf/Sprint/Sprintln in the fmt package
+ strings.Join()
+ **bytes.Buffer** (e.g., var buf bytes.Buffer)
+ **strings.Builder** (e.g., var builder strings.Builder)
+ `+` operator, standard go compiler makes optimizations for string concatenations by the `+` operator, generally, using `+` is convenient and efficient if number of concatenated strings is known at compile time.

## sugar: strings as []byte

+ **StringType = append(ByteSlice, StringType...)**
    e.g., _ = append([]byte{}, "sugar"...)
+ **copy(ByteSlice, StringType)**
    e.g., copy([]byte{}, "sugar")

## string comparison (`==` & `!=`)

+ **lengths are not equal (then O(1))**, the 2 strings must be not equal (no need to compare bytes)
+ **underlying byte sequence pointers are equal**, comparison result is the same as comparing the lengths of the 2 strings **O(1)**; **otherwise** compare byte sequence **O(n)**

2 equal strings, the time complexity of comparing them depends on **wether or not their underlying byte sequence pointers are equal**. If the two are **equal**, then the time complexity is **O(1)**, **otherwise** the time complexity is **O(n)**

try to avoid comparing 2 long strings, if they don't share the same underlying byte sequence
