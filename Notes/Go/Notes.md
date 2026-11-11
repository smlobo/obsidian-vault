# Go language quick reference

## Predeclared types and zero values

The zero value is what a variable gets when declared without an explicit
initializer. For arrays and structs, it is the zero value of each element or
field.

| Type | Zero value |
| --- | --- |
| `bool` | `false` |
| `string` | `""` (empty string) |
| `int`, `int8`, `int16`, `int32`, `int64` | `0` |
| `uint`, `uint8`, `uint16`, `uint32`, `uint64`, `uintptr` | `0` |
| `byte` (`uint8` alias), `rune` (`int32` alias) | `0` |
| `float32`, `float64` | `0` |
| `complex64`, `complex128` | `0` (zero complex value) |
| Pointer | `nil` |
| Function | `nil` |
| Interface | `nil` |
| Slice | `nil` |
| Map | `nil` |
| Channel | `nil` |
| Array | Array whose elements are their zero values |
| Struct | Struct whose fields are their zero values |

`any` is an alias for the empty interface, and `comparable` is a predeclared
constraint; neither is a concrete value type with a separate zero value.

## Predeclared constants

| Name | Meaning |
| --- | --- |
| `true`, `false` | Boolean constants |
| `iota` | Integer constant used to generate successive values in a `const` declaration; it starts at `0` in each `const` block and increments for each `ConstSpec` |
| `nil` | Predeclared zero value for pointer, function, interface, slice, map, and channel types |

`nil` has no type of its own. It must be assigned or compared in a context
that provides a compatible type; it is not valid for numeric, boolean, or
string values.

## Keywords

Go has 25 reserved keywords:

```text
break      default     func       interface  select
case       defer       go         map        struct
chan       else        goto       package    switch
const      fallthrough if         range      type
continue   for         import     return     var
```

## Built-in functions

These predeclared functions are available without importing a package:

```text
append  cap      clear   close   complex  copy
delete  imag     len     make    max      min
new     panic    print   println real     recover
```

- `make` creates and initializes slices, maps, and channels; `new` allocates
  zeroed storage and returns a pointer.
- `append` adds elements to a slice and returns the resulting slice. `copy`
  copies slice elements.
- `len` and `cap` report length and capacity. `clear` zeroes map entries or
  slice elements.
- `close` closes a channel; `delete` removes a map entry.
- `complex`, `real`, and `imag` construct or inspect complex numbers.
- `max` and `min` return the largest or smallest value among their arguments.
- `panic` and `recover` control panic handling. `print` and `println` are
  implementation-specific debugging functions; use `fmt` for normal output.

Reference: [The Go Programming Language Specification](https://go.dev/ref/spec).
