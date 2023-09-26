# References

## Read XOR Mutable

Recall from Week 1 that in Rust variables are move-assigned, and thus can only be used
directly in one operation before being invalid.

**But why though?**
Remember the goals of Rusts:
- Be low level and fast
- Be memory safe
With that in mind, imagine you had the following type:

```rust
struct MemoryOwner<T> {
    // size of memory block
    size: usize,
    // pointer to heap memory
    data: mut *T,
}
```
If you directly copied this object every time you assigned it to something, then you could have two objects that contain a pointer to the same memory. If you were to change that memory in one of the objects, then it would silently affect the other. Also, who is supposed to actually free it?

There are several ways to approach memory management and variable assignment
| Method |   | Langauages | but... |
| ------ | - | ---------- | ------ |
| Shallow copy | `=` creates a new value with the exact same data as the RHS | C, C++ (default), Zig... | Completely different parts of your codebase could share the same resource. You have to be extremely maticulous with each active pointer to prevent unexpected (or even undefined) behaviour.
| Copy constructor | - | `=` creates a new value from a constructor function with the RHS as an argument. For example you can simply allocate a new block of dynamic memory and copy the contents of an old string into it. | C++ (with overloaded `=`), Rust (with `Copy` trait) | Copying becomes a potentially non-trivial operation, especially if it involves allocating memory. This is why Rust, as a high-performance language makes String impl `Clone` but not `Copy`. |
| Garbage Collector | `=` implicitly assigns a *counted reference* to an object. A garbage collector periodically goes through all unused objects and frees their momory. | Python, Java | The garbage collector becomes a hidden operation that you have no control over. You also lose the ability of manage the dynamic memory directly, which is can be faster and is often desired in performance-critical applications. |
| Moving | `=` 'moves' the object from the RHS to the LHS. That is the variable name on the RHS no longer refers to the value being moved, and is usually made invalid. | Rust, c++ (with std::move) | It can be annoyingly verbose to need to explicitly copy everything. Expecially if what you're copying is simple struct that doesn't contain any pointers. |

### Exceptions
Other languages may have checked exceptions


### Sum Types (Aka enums continued)

#### Bonus! Unions
These look similar to [Unions] in other languages. In fact, Unions exist in Rust as well. The main differences are:

| Enums | Unions |
|-------|--------|
| Wrapped data cannot be accessed directly. Variant type must be checked first | Can access data directly, but only in an `unsafe` block (more in week 9) |
| To change the type of wrapped data, you must explicitly wrap them in the corresponding variant | Assign to the field of that type |
| Have hidden *discriminant* u8, whose value corresponds to a particular variant. | No discriminator. |
| No restrictions on in internal data type | Internal data types must impl `Copy` and `ManuallyDrop<T>`, or they must be references, or structs/tuples containing only aforementioned union types |
| Can derive `Debug` | Cannot derive `Debug` |


```rust
// Total size is 9 bytes. (8 for largest variant + 1 for discriminator)
#[derive(Debug)]
enum VarEnum {
    Int(i32),
    LongInt(i64),
    Float(f64),
}

let mut var_enum = VarEnum::Int(42);

var_enum = VarEnum::Float(69.);
// Even editing the data of the same variant requires explicitly declaring it
var_enum = VarEnum::Float(69.9);
// Reading to manipulate data requires pattern matching. What a pain
var_enum = match var_enum {
    VarEnum::Float(f) => VarEnum::Float(f + 1.),
    _ => var_enum
};


// Total size is 8 bytes. (8 for largest field)
// #[derive(Debug)] error, Unions cannot derive Debug
union VarUnion {
    int: i32,
    long_int: i64,
    float: f64,
}

// To create a union. Declare exactly one field
let mut var_union = VarUnion { int: 42 };
// You can change the internal data even in safe Rust all you want
var_union.long_int = 9;
var_union.float = 69.0;

// But reading can get a bit dangersome
unsafe {
    let f = var_union.float;
    println!("float: {f}");

    // Who knows what will happen here
    let mut lf = var_union.long_int;
    println!("long: {lf}");

    // Could be useful for reinterperet casting?
    let mut exponent = (lf & (0x11111111111 << 52)) >> 52;
    exponent -= 1;

    lf = lf & !(0x11111111111 << 52);
    lf = lf | (exponent << 52);
    var_union.long_int = lf;

    println!("Half float: {}", var_union.float);
    // ???????????????????????????????????????????
    // https://en.wikipedia.org/wiki/Double-precision_floating-point_format

    
}
```

The moral of the story is: with unions you have to be constantly aware of the underlying type. In other languages this usually involves explicitly storing a discriminant value explicitly. Rust does this implicitly, saving you some boilerplate (an providing type safety).

Of course, you can explicitly declare and even access discriminants as well

```rust
use std::mem;

// Enum with explicit discriminants
#[repr(u8)]
enum WhatAmI {
    Int(i32) = 0,
    Float(f64) = 3,
    // Error, can't assign different variants the same discriminant
    // LongDouble(f128) = 3
}


// Accessing the discriminants
assert_eq!(mem::discriminant(&WhatAmI::Int(4)), mem::discriminant(&WhatAmI::Int(-12)));

```
More info can be found on [the Rust Reference](https://doc.rust-lang.org/reference/items/enumerations.html)