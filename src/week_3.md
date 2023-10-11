# References

## Ownership and Moooving

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

| Method |     | Langauages | but... |
| ------ | --- | ---------- | ------ |
| Shallow copy | `=` creates a new value with the exact same data as the RHS | C, C++ (default), Zig... | Completely different parts of your codebase could share the same resource. You have to be extremely maticulous with each active pointer to prevent unexpected (or even undefined) behaviour.|
| Copy constructor | `=` creates a new value from a constructor function with the RHS as an argument. For example you can simply allocate a new block of dynamic memory and copy the contents of an old string into it. | C++ (with overloaded `=`), Rust (with `Copy` trait) | Copying becomes a potentially non-trivial operation, especially if it involves allocating memory. This is why Rust, as a high-performance language makes String impl `Clone` but not `Copy`. |
| Garbage Collector | `=` implicitly assigns a *counted reference* to an object. A garbage collector periodically goes through all unused objects and frees their momory. | Python, Java | The garbage collector becomes a hidden operation that you have no control over. You also lose the ability of manage the dynamic memory directly, which is can be faster and is often desired in performance-critical applications. |
| Moving | `=` 'moves' the object from the RHS to the LHS. That is the variable name on the RHS no longer *owns* to the value being moved, and is usually made invalid. | Rust, c++ (with std::move) | It can be annoyingly verbose to need to explicitly copy everything. Expecially if what you're copying is simple struct that doesn't contain any pointers. |

### Partial moves
Since Rust assigns by move, taking from a part of a compound value will *partially move* the value. This means you can no longer access the whole value, although you can use the still un-moved parts.

```rust
#[derive(Clone, Default)]
struct BigHappyFamily {
    parents: Vec<String>,
    children: Vec<String>,
    pets: Vec<String>,
}

let fam = BigHappyFamily::default();

let dogs = fam.pets;

// Won't work because the pets have been moved out
// let new_fam = fam.clone();

// This is fine, as the parents havent left yet
let adults = fam.parents;

// Move the kids in too
let new_fam = BigHappyFamily {
    parents: adults,
    pets: dogs,
    children: fam.children,
};

```

## Borrows
Well what is if you want to read/modify a value without moving it. You do this all the time reading from a large struct or adding to a collection. To do this, we make the observation that in these cases the value you are using is gaurenteed to exist after the operation, although maybe in a different form. You are just *borrowing* the value.

### Read XOR Mutable
|   |   |   |
| - | - | - |
| Shared borrow / Reference | `&var_name` | Provides read only access to a value. |
| Exclucive borrow / Mutable reference | `&mut var_name` | Provides read and write only access to a value. |

You can have as many shared borrows as you want. However, if you have an Exclusive borrow, there can be no other exclusive or shared borrows to that variable.

**Why these restrictions?**
Read XOR Mutablity is concept largly taken from concurrency to prevent race conditions. See [Week 8] for a deeper discussion on this.

However, even in a single threaded case this can be advantagous. A really common bug occurs when you are reading from a reference whose value changes when you are not expecting to.

```rust
let mut v1 = vec![1, 2, 3];
let mut v2 = vec![4, 5, 6];

fn extend<T>(dest: &mut Vec<T>, src: &Vec<T>) {
    while !src.is_empty() {
        dest.push(src.pop());
    }
}

// Runs fine
extend(&mut v2, &v1);

// Error. If this was valid it could run forever.
// Or Undefined behavior. Whichever is worse
// extend(&mut v1, &v1);

```

This is a relatively simple example. But as your codebase grows it can be increadably difficult to track all the references that can exist at once. For example, imagine the example above, execpt of `&Vec`, `extend`'s arguments were a struct that contained a reference to a struct that contained a reference to a struct... that contained a `&Vec` that may or may not be referencing the same `Vec`. Instead, Rust prevents this from happening at *compile time* with Read XOR Mutable.

### Partial Borrows
Much like partial moves, you can borrow individual fields of compound values. Keep in mind that this would require creating a borrow to said compound borrow of that level of exclusivity (or higher)

```rust
#[derive(Clone, Default)]
struct BigHappyFamily {
    parents: Vec<String>,
    children: Vec<String>,
    pets: Vec<String>,
}

let fam = BigHappyFamily::default();


// Creates an exclusive borrow of the 'parents' field of fam, 
// which (implicitly) creates an exclusive borrow of fam
let adults = &mut fam.parents;
adults.push("Tom".to_string());

// Error. fam.parents exclusively borrowed above and used below in the println!
// let static_adults = &fam.parents;

// Error. fam exclusively borrowed above, used below in the println!
// let static_fam = &fam;

// Totally ok, fam.children has not been borrowed. We can use the 
// exclusive borrow of fam to make a shared borrow of its fields
let kids = &fam.children;
let kids_again = &fam.children;

println!("{:?}, {:?}", adults, kids);



// This is ok now, because the compiler can safely drop the exclusive
// borrow to fam.parents as 'adults' will not be used again
// I.e 'adults' lifetime end here
let static_adults = &fam.parents;
```
<!-- An example of the borrow checker throwing an error in a match or for loop
 I remember it happening in my A1/A2 all the time but totally forgot

**This will cause you to suffer**

```rust
struct Data(i32);

struct ArgHolder {
    a: Data,
    b: Data
} -->

```
The borrow checker is notorious for being the most frustrating part of using the language, so don't worry if you struggle with it.

## Lifetimes
Remember, the one thing you can't do with a shared or exclusive borrow is *move* the variable being borrowed. To that end, a reference may only exist for as long as the referenced variable: its *lifetime*. Of course, we can choose to end the *reference* lifetime before that (i.e. when we create another mutable reference to the variable, which will end the old one).

```rust
fn consume(a: String) {}

let s = String::from("Hiya");
let s1 = &s;

// moving s ends its lifetime, which in turn ends s1's lifetime
consume(s);

// Won't compile. s1's lifetime ended when s was moved
println("{s1}");
```



### In functions
In Rust a function that returns a reference needs to know how long the reference will live for. We do
this by declaring generic lifetimes `'lifetime_name`, and then assigning them to each reference used as an argument or return value.

```rust
// Returns a reference that will live for AT MOST as long as the
// first argument. It will have no relation to the second argument.
fn split_any<'long, 'idc>(text: &'long str, delims: &'idc str) -> Option<&'long str> {
    text.iter()
        .enumerate()
        .filter(|(_, x)| { delims.contains(x) })
        .nth(0)
        .and_then(|(i, _)| Some(&text[i..]))
}


// Outside this function s1 and s2 may have completely different lifetimes,
// by assigning the same one here, we're saying "pick the shortest of two".
fn longest<'a>(s1: &'a str, s2: &'a str) -> &'a str {
    [s1, s2].iter()
        .max_by_key(|s| s.len())
        .unwrap()
}
```

### In structs
Structs with references as fields will also require explicit lifetimes
```rust
struct NamedThing<'a, T> {
    name: &'a str,
    thing: T,
}
// The same is true for enums
enum CouldBeAName<'a> {
    Name(&'a str),
    NotName(i32)
}

// When you impl methods of such a struct, you must declare the lifetimes of that impl
impl <'a, T> for NamedString<'a, T> {
    // Notice that since we specifies 'a in the impl, we needn't declare in the method 
    pub fn new(&'a name, T thing) -> Self {
        Self { name, thing }
    }

    // Of course if we want to declare a completely different lifetime, you have to declare it
    pub fn find_name<'b>(&self, text: &'b str) -> Option<&'b str> {
        b.find(a)
    }
}
```

### Lifetime constraints
You can declare that a certain lifetime *must* outlive another lifetime in the where clause of a function/type.


*Observation* You may notice that lifetimes are specified in a similar manner to generics. Why? 


### 'static lifetime
`'static` is a special liftime. A variable with a `'static` lifetime **will not expire until the end of the program**. 

```rust
// This is how we can declare references as global variables
const GREETING_STR: &'static str = "Hoya";
static FAREWELL_STR: &'static str = "Sayonara suckers!";

enum SmallTalk {
    Greeting,
    Farewell
}

fn get_smalltalk(s: SmallTalk) -> &'static str {
    match s {
        SmallTalk::Greeting => GREETING_STR,
        SmallTalk::Farewell => &FAREWELL_STR
    }
}

println!("{}", get_smalltalk(SmallTalk::Greeting));
```

```rust
// A few (very weird) functions will let you leak memory so that it lasts forever
fn make_forever<T: Default>() -> &'static mut T {
    let boxed = Box::new(T::default());
    Box::leak(boxed)
}

let v : &mut i32 = make_forever();
assert_eq!(v, &0);

```


### Elision
In certain cases, Rust will allow you to not explicity declare a lifetime.

You're already familiar with the first case, when a function takes in references, but does not output one.

Another case is when a function returns the same reference every time.

Rust can do this because, in both cases, the compiler can easily 'figure out' the lifetime the function returns.

*Why can't the compiler just do that all the time?*

### Exceptions
Other languages may have:
##### Unchecked Exceptions
Where a function can `throw` an error and, when it does so run time and there is no `catch` arm, the program will just crash. The idea is that the caller of a failable function may not directly handle it's error case, but another function up the call tree will. This way you can have one parent function handle a variety of errors that it's children may propagate in a uniform way.

Rust also has this... in the form of [`panic!()`](https://doc.rust-lang.org/std/macro.panic.html). (i.e. just crash the program)

##### Checked Exceptions
Where a function can `throw` an error and, if the developer has not created a `catch` arm somewhere up the call stack to handle it, will *cause the code to not compile*. 
This way the you can rest assured that an error won't happen that you are not prepared for.

#### Results continued
But the preferred way to handle errors in Rust is with Results. The `enum` `Result<T, E>` will 

```rust
enum AnError { E };

fn failable() -> Result<(), AnError> {
    if false {
        // Function succeeds, return ()
        Ok(())
    }
    else {
        // Function fails
        Err(AnError::E)
    }
    // Notice that, thanks to syntax sugar, you needn't write
    // Result::Ok or Result::Err, just like Options
}

```

Any time you call a failable function, you must handle it locally. If you do want to propogate it upward you have to do it manually, and all functions up the call stack also handle it.

```rust
enum AnError { E };

fn failable() -> Result<(), AnError> {
    Err(AnError::E)
}

// caller must also be a Result with the same Err variant
fn caller() -> Result<i32, AnError> {
    // The ? operator will return the value (exit the function) if it's an Err
    let x = failable()?;

    Ok(32)
}

```

It's not uncommon for libraries to define specialised versions of Result, so you don't have to write the output and error types all the time. E.g. `std::io::Result` does this
```rust
enum AnError { E };
struct AnOutput;

type AnResult = Result<AnOutput, AnError>;

```

### Sum Types (Aka enums continued)
A Sum Type is basically a type that could take the form of one of multiple inner types (Variants) and a *discriminant* to determine which inner type it is. Some may call them Tagged Unions. This is in contrast to an Enumerable Type, a type that could take one of multiple *labels* (which is how `enum`s work in most other languages). 

```rust
// Sum Types are considered disjoint unions. This means that even if two different variants 
// have the same value, they are not considered equal.
enum Number {
    Type1(i32),
    Type2(i32)
}

let t1 = Number::Type1(4);
let t2 = Number::Type2(4);

assert_neq!(t1, t2);

// Unless, of course, you impl a PartialEq that makes them equal
```

Under the hood, the discriminator is probably a `u8`, making an Rust enum with valueless inner types exactly like regular enums.

You can even define zero-variant enums. They have no valid value, which is surprisingly equivalent to the [Never](https://doc.rust-lang.org/std/primitive.never.html) type.

```rust
enum NoValues {};

let v : NoValues = panic!();

```


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
    other_float: f64,
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


    // Also, Unions are not disjoint, so different fields can be equal
    assert_eq!(var_union.float, var_union.other_float);
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