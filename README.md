# Go


## Short variable declarations
Inside a function, the := short assignment statement can be used in place of a var declaration with implicit type.

Outside a function, every statement begins with a keyword (var, func, and so on) and so the := construct is not available.

```go
var i, j int = 1, 2
k := 3
c, python, java := true, false, "no!"
```

## types

. string
. bool
. int8, uint8 (byte), int16, uint16, int32 (rune), uint32, int64, uint64, int, uint, uintptr.
float32, float64.
complex64, complex128

Note, byte is a built-in alias of `uint8`, and `rune` is a built-in alias of `int32`. We can also declare custom type aliases (see below).

### cast

`uint(x)`

## pointers

Go has pointers. A pointer holds the memory address of a value.

The type `*T` is a pointer to a `T` value. Its zero value is `nil`.

`var p *int` The `&` operator generates a pointer to its operand.

```go
i := 42
p = &i
```
The * operator denotes the pointer's underlying value.

```go
fmt.Println(*p) // read i through the pointer p
*p = 21         // set i through the pointer p
```
This is known as "dereferencing" or "indirecting".

Unlike C, Go has no pointer arithmetic.

## structs

```go
type Vertex struct {
    X int
    Y int
}

func main() {
    v := Vertex{1, 2}
    v.X = 4
    fmt.Println(v.X)
    // To access the field X of a struct when we have the struct pointer p we could write (*p).X. However, that notation is cumbersome, so the language permits us instead to write just p.X, without the explicit dereference.
    p := &v
    p.X = 1e9}

```


## array

The expression `var a [10]int` declares a variable a as an array of ten integers. An array's length is part of its type, so arrays cannot be resized. 

### silces

A slice does not store any data, it just describes a section of an underlying array.
Changing the elements of a slice modifies the corresponding elements of its underlying array.
Other slices that share the same underlying array will see those changes.

This is an array literal:

`[3]bool{true, true, false}`
And this creates the same array as above, then builds a slice that references it:

`[]bool{true, true, false}`

#### nil slices

```go
    var s []int
    fmt.Println(s, len(s), cap(s))
    if s == nil {
        fmt.Println("nil!")
    }
```

### make

The make function allocates a zeroed array and returns a slice that refers to that array:

`a := make([]int, 5)  // len(a)=5`
`a len=5 cap=5 [0 0 0 0 0]`
To specify a capacity, pass a third argument to make:

`b := make([]int, 0, 5) // len(b)=0, cap(b)=5`
`b len=0 cap=5 []`

`b = b[:cap(b)] // len(b)=5, cap(b)=5`
`b = b[1:]      // len(b)=4, cap(b)=4`

### append 

`func append(s []T, vs ...T) []T`

The first parameter s of append is a slice of type T, and the rest are T values to append to the slice.

### range

You can skip the index or value by assigning to _.

```go
for i, _ := range pow
for _, value := range pow
```

```go
    for i, value := range pow {
        fmt.Printf("%d: %d\n", i, value)
    }
```

## maps

```go
var m map[string]Vertex
m = make(map[string]Vertex)
```

```go
var m = map[string]Vertex{
    "Bell Labs": Vertex{
        40.68433, -74.39967,
    },
    "Google": Vertex{
        37.42202, -122.08408,
    },
}
```

Delete an element: `delete(m, key)`
Test that a key is present with a two-value assignment: `elem, ok = m[key]`

## functions

Function closures
Go functions may be closures. A closure is a function value that references variables from outside its body. The function may access and assign to the referenced variables; in this sense the function is "bound" to the variables.

For example, the adder function returns a closure. Each closure is bound to its own sum variable.

```go
package main

import "fmt"

func adder() func(int) int {
    sum := 0
    return func(x int) int {
        sum += x
        return sum
    }
}

func main() {
    pos, neg := adder(), adder()
    for i := 0; i < 10; i++ {
        fmt.Println(
            pos(i),
            neg(-2*i),
        )
    }
}
```

### defer

A defer statement defers the execution of a function until the surrounding function returns.

The deferred call's arguments are evaluated immediately, but the function call is not executed until the surrounding function returns.

```go
func main() {
    defer fmt.Println("world")
    fmt.Println("hello")
}
```

### methods

Go does not have classes. However, you can define methods on types.
A method is a function with a special receiver argument.
The receiver appears in its own argument list between the func keyword and the method name.

```go
type Vertex struct {
    X, Y float64
}

func (v Vertex) Abs() float64 {
    return math.Sqrt(v.X*v.X + v.Y*v.Y)
}
```
You can only declare a method with a receiver whose type is defined in the same package as the method. You cannot declare a method with a receiver whose type is defined in another package (which includes the built-in types such as int).

You can declare methods with pointer receivers. For example, the Scale method here is defined on *Vertex.

```go
func (v *Vertex) Scale(f float64) {
    v.X = v.X * f
    v.Y = v.Y * f
}
```
Methods with pointer receivers can modify the value to which the receiver points (as Scale does here). Since methods often need to modify their receiver, pointer receivers are more common than value receivers.

That is, as a convenience, Go interprets the statement v.Scale(5) as (&v).Scale(5) since the Scale method has a pointer receiver. (you can do it with _methods_, but not with _functions_ and viceversa)

## interfaces

An interface type is defined as a set of method signatures. A value of interface type can hold any value that implements those methods. 

A type implements an interface by implementing its methods. There is no explicit declaration of intent, no "implements" keyword.

Implicit interfaces decouple the definition of an interface from its implementation, which could then appear in any package without prearrangement.

### interface values

Under the hood, interface values can be thought of as a tuple of a value and a concrete type:

`(value, type)`
An interface value holds a value of a specific underlying concrete type.

Calling a method on an interface value executes the method of the same name on its underlying type.

### empty interface

```go
func main() {
    var i interface{}
    describe(i)

    i = 42
    describe(i)

    i = "hello"
    describe(i)
}

func describe(i interface{}) {
    fmt.Printf("(%v, %T)\n", i, i)
}
(<nil>, <nil>)
(42, int)
(hello, string)
```

### type assertion

```go
func main() {
    var i interface{} = "hello"

    s := i.(string)
    fmt.Println(s) // hello

    s, ok := i.(string)
    fmt.Println(s, ok) // hello true

    f, ok := i.(float64)
    fmt.Println(f, ok) // 0 false

    f = i.(float64) // panic
    fmt.Println(f)
}
```

you can either use the keyword `type` for selecting variable type:

```go
func main() {
    do("hello")
    do(21)
    do(true)
}

func do(i interface{}) {
    switch v := i.(type) {
    case int:
        fmt.Printf("twice %v is %v\n", v, v*2)
    case string:
        fmt.Printf("%v is is %v bytes long\n", v, len(v))
    default:
        fmt.Printf("%v is type %T\n", v, v)
    }
}
```
