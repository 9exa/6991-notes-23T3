# Week 1

## Preamble
This is primarily a programming course, not (just) a Rust course. As you go through these notes considere
why the designers of Rust made those decision and the downsides of those decisions

"I will mention code that wakes you up at the 3am every lecture" - Zac


## The Toolchain
A toolchain is a series of programs used to facilitate the building of your code into an actual executable/library. 

| tool | function |
| ---- | -------- |
| rustup | Rust Version manager. Installs Rust on your system. Lets you switch between Nightly, Stable and even previous version |
| rustc | Compiler. Compile *manually specified* source code files into an executable/library. Comparable to gcc |
| cargo | Build System + Package manager. Let's you add external packages (crates) to your rust project. Also contains other commands to aid development (E.g. tests, docs, ...). The recommended way to write Rust programs in this course. |

### Basic cargo commands
Remember, on CSE Machines use 6991 cargo to use the latest toolchain.

| Command | Description |
| ------- | ----------- |
| new *proj_name* | creates a new Rust project in a newly created *proj_name* directory |
| init | initializes a new Rust project in the currect directory |
| build | trys to compile the current project |
| run -- *cargs* | build and run your program with the *cargs* command-line arguments |

Of course, entering 6991 cargo -h will give you these commands in greater detail

## Jumping into it / Some code from the lectures

### Hello World
```rust
// copy and pasted
fn main() {
    println!("Hello World!");
}
```

Of note:
- `main` returns `()`
- `println` is a macro.


You can run this by entering the command `cargo run`.

### Lines
```rust
use bmp::{Pixel, Image};

const IMAGE_SIZE_PX: u32 = 256;

struct Student {
    name: String,
    zid: u32,
    wam: f64,
    is_comp_student: bool,
}

fn main() {
    let mut my_image = Image::new(IMAGE_SIZE_PX, IMAGE_SIZE_PX);

    let red = create_pixel(255, 0, 0);
    let green = create_pixel(0, 255, 0);
    let blue = create_pixel(0, 0, 255);
    let white = create_pixel(255, 255, 255);
    
    for x in 0..IMAGE_SIZE_PX {
        for y in 0..IMAGE_SIZE_PX {
            let pixel = if (x + y) % 15 == 0 {
                red
            } else if (x + y) % 15 == 5 {
                green
            } else if (x + y) % 15 == 10 {
                blue
            } else {
                white
            };

            my_image.set_pixel(
                x,
                y,
                pixel
            );

            // equivalent to
            // if (x + y) % 15 == 0 {
            //     my_image.set_pixel(x, y, red);
            // } else if (x + y) % 15 == 5 {
            //     my_image.set_pixel(x, y, green);
            // } else if (x + y) % 15 == 10 {
            //     my_image.set_pixel(x, y, blue);
            // } else {
            //     my_image.set_pixel(x, y, white);
            // }
        }
    }

    my_image.save("my_image.bmp").expect("Failed to save image!");
}

fn create_pixel(r: i32, g: i32, b: i32) -> Pixel {
    // let pixel = Pixel {
    //     r: r.try_into().expect("i32 was too big"),
    //     g: g.try_into().expect("i32 was too big"),
    //     b: b.try_into().expect("i32 was too big"),
    // };

    // return pixel;

    Pixel {
        r: r.try_into().expect("i32 was too big"),
        g: g.try_into().expect("i32 was too big"),
        b: b.try_into().expect("i32 was too big"),
    }
}
```
Of note:
- To import external files: use {module}::{stuff-from-file}
- `const` values are typed
- image must be declared as `mut`
- You can declare a function after using it
- The `as` operation can result in some unexpected behaviour, so you can use `try_into().unwrap()`
- Function blocks are treated as expressions, so you can have it return a value withouth using the `return` keyword by typing it in the last.

## Rust basics

### Basic types

| Integer types |  |
| --- | ------- |
| u8 | unsigned 8-bit integer |
| u16 | unsigned 16-bit integer |
| u32 | unsigned 32-bit integer |
| u64 | unsigned 64-bit integer |
| u128 | unsigned 128-bit integer |
| i8 | signed 8-bit integer |
| i16 | signed 16-bit integer |
| i32 | signed 32-bit integer |
| i64 | signed 64-bit integer |
| i128 | signed 128-bit integer |
| usize | pointer sized unsigned int. Architecture specific. E.g. 64 bit in x86_64 |
| isize | pointer sized signed int |

\* Tip. You can directly specify the integer type by writing the typename as a suffix to the literal. Eg. `let x = 1u128`.

| Floating point types |  |
| --- | ------- |
| f32 | 32-bit floating point number |
| f64 | 64-bit floating point number |
| f128 | 128-bit floating point number |

\* To distinguish from integer types, you have to include the decimal point OR the suffix when writing a float literal. Eg `let x = 1.` OR `let x = 1f32`.

|  Boolean |  |
| --- | ------- |
| bool | can be `true` or `false` |
 

| Character types |  |
| --- | ------- |
| char | a single UTF-8 character |
| str | a contiguous collection of UTF-8 characters. Can't be created directly. See [String](week_2/#collections-and-iterators). Usually accessed as a &str |


### Variables
Create variables with the `let` keyword. Variables created this way are *immutable* and cannot be changed. To make mutable one use `mut let`.

```rust
/// immutable variables can't be changed, but they CAN be reassigned with a brand new values
let a = 4;
println!("The value of a: {a}");
// a = 4.5; Would not compile
let a = 4.5;
println!("The NEW value of a: {a}");


/// However, mut variable cannot be reassigned [until we exit the scope, ofc].
let mut b = 42i32;
println!("The value of b: {b}");
// let b = 4.2; Would not compile
// b = 4.2; Also wouldn't compile, cause it's a different type
b = 4;
println!("The NEW value of b: {b}");
```
In this way, you can view variables as LABELS to data as opposed to the data itself.


All variables are strongly typed.
```rust
// If you don't type the var implicitly, the compile will do it's best to guess for you.
let explicitly_typed: &str = "I'm a &str, right?";
let implicitly_typed = "I know I am.";
let mut future_imp_typed;
future_imp_typed = "me too";

// But sometimes it can't
// let v = Vec::new(); can't know the items of the Vec
let v: Vec<i32> = Vec::new();
```

In Rust, the `=` operator is 'assign-by-move'. Basically it consumes the value on the right-hand-side. To access
variables *without* moving them, use references to them with the `&` and `&mut ` operators
```rust
let a = 32;
let b = a / 2;
// a has been moved
println!("{}", a);
```
```rust

fn double(x: &mut i32) {
    *x *= 2;
}

let mut a = 32;

// we can use & ti read a and &mut a to change it with another function
println!("old a: {}", &a);
double(&mut a);
println!("new a: {}", a);

```

### Functions
To define a function in Rust, use the `fn` keyword at the file scope. Arguments go in the (), like most other languages.

```rust
fn add5(a: i32) -> i32 {
    // The body of a function is an expression. So if you're not exiting
    // the function early, you can forgoe the return keyword and semicolon
    a + 5
}
println!("2 + 5 = {}", add5(2));

```
Closures (which are basically functions with extra state) are first-class values and can be assigned to variable

```rust
let add5 = |a: i32| { a + 5 };

println!("2 + 5 = {}", add5(2));
```


### Tuples
Collection of values. Can have mixed types. Access by using `.0, .1, .2, ...` or use pattern matching. Careful, they have a maximum length of 16!
```rust
let pair = (4, 2);
println!("With indexes .0: {}, .1: {}", pair.0, pair.1);

// Unpack to new variables with pattern matching
let pair = (4, 2);
let (a, b) = pair;

println!("With patterns a: {}, b: {}", a, b);
```


### Structs 
A statically defined collection of data.

```rust
// Classic Struct with named fields
struct MyStruct {
    a: u32,
    b: i32,
    c: String,    
}

// You define them without named fields. Aka a tuple struct. 
// (Notice the ;)
struct OtherStruct(u32, i32, String);

// Structs with no feilds are valid, 
// and do not consume memory
struct ZeroSized;

// Creating a the Struct. Each named field has to specified, but can be in any order
MyStruct { a: 1, b: -42, c: "Ahhhhhh".to_string() };

// In the case of a tuple structs, fields have to be in order
OtherStruct(1, -42, "Ahhhhh again".to_string());

```


### Enums
A value that can take multiple forms. This is similar to a 'variant' in other programming languages
```rust
enum MyEnum {
    Form1,
    Norm2,
    // Enums can hold fields, like a struct
    WithValues(u32, u32),
    WithNamed {a: u32, b: u32},
}

// Zero sized enums are also valid 
enum Empty {};
```

### Optional Values
Rust does not have a NULL value. To declare that a variable may not hold a value, you can declare
it as an Option<T>. Internally Option is just a generic enum.
```rust
enum Option<T> {
    // The value exists
    Some(T),
    // The value does not exist
    None,
}
```
There are various methods to access the value in an option
```rust
let opt = Some(45);

/// Resolves to the wrapped T if opt is Some. panics if opt is None
opt.unrwap();
/// Resolves to the wrapped T if opt is Some. panics with the provided error message if None
opt.expect("Error Message");
/// Resolves to the wrapped T if opt is Some. Resolves to provided default value if None
opt.unwrap_or(def);
/// Resolves to the wrapped T if opt is Some. Resolves to T::default() if None
opt.unwrap_or_default();

/// Assigns the inner value to the variable x in the scope of {} if opt is Some. Does not run {} if opt is None 
if let Some(x) = opt {}
```

### Result
Sometimes a function may not be successful. However, instead of returning `None`, we may want to pass some data that describes the error. This is where Results are useful.

```rust
// Internal Definition of a Result
enum Result<T, E> {
    // Function was successful and produced a T value
    Ok(T),
    // Function failed and produced an error of type E
    Err(E),
}
```
An easy thing you can do is implement the error type as an enum

```rust
#[derive(Debug)]
enum AnError {
    IsNegative,
    NotInteger,
}

fn add_natural_numbers(p: f32, q: f32) -> Result<u32, AnError> {
    if (p.fract(), q.fract()) != (0.0, 0.0) {
        return Err(AnError::NotInteger);
    } 
    else if p.is_sign_negative() || q.is_sign_negative() {
        return Err(AnError::IsNegative);
    }
    Ok((p + q) as u32)
}

println!("Valid input:   {:?}", add_natural_numbers(1.,3.));
println!("Invalid input: {:?}", add_natural_numbers(1.,-3.));
```

Like Option, Result also has various methods (like the `.unwrap` family) to help extract valid data.

Have a think. Why would you want to use Results or Options as opposed to creating your own enum to handle multiple cases for your functions?