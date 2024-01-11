# Unsafe Rust

## Memory safety

### Undefined behaviour

### Goals of a programming language
Of course the myriad of popular programming languages have a variety of goals, but for sake of argument, consider these four

- *Memory Safety*: How hard is it to accidentally make the most dangerous class of bugs?
- *Low-level control*: PERFORMANCE! (In both memory usage and time of instructions)
- *Ease of Use/Expressiveness*: How much of a pain in the butt is it to write the behaviour you want?
- *Platform support*: Which machines can run your code>

These goals are largely mutually exclusive. 

For example, it can be argued that c focuses on the 2nd and 4th goals; c provides the control required for the kernel of your OS and is the language of choice for most embedded systems. However, it's increadibly easy to cause a SEGFAULT. 

Python can be considered the polar opposite to that. It tries to be the language you can just sit down and write **what you want to do**. No platform specific compiler, forgiving semantics, simple syntax, you don't worry about the memory used by your abstractions, no main function even. However, a python script can be orders of magnitudes slower than the equivalent program in a lower-level language.


Of course, if a language can be changed slightly to accomplish one of the other goals without sacrificing its main one, the maintainers will attempt to do that. *(Barring institutional inertia...)*

Rust tries to juggle all four of these goals. 
- Memory Safety: Has a very strict memory model and compile time
- Low level Access: Rust strives to use Zero-cost abstractions, the compiled machine code should be the same as if all the layers of abraction weren't even there.
- Expressiveness: Traits, algebraic data types, closures, and other features you've been learning
- Platform support: Compiled to machine code using one of the compiler frameworks popular with C, [llvm]

The big one is Memory Saftey. The idea is that it stops you from even being able to write code that **could potentially** have those bugs. 

## Why does unsafe Rust exist?
There are many programming patterns that you **can** use safely, but the Rust compiler just can't know for sure that you always will. In fact alot of Rust types that you've been using so far need to rely on those techniques, like the humble `Vec`. Rather than have a completely seperate language to do the unsafe stuff, we have sections of Rust code with relaxed restrictions. 




Contrary to what you may think, Unsafe Rust is **superset** of Safe Rust. That is, everything you've been writing so far is code valid in an `unsafe` block.

All the `unsafe` block allows you to do is

## Responsible unsafety

### Soundness checking tools

*Safety comments*

## unsafe impls

## Why not just allways use unsafe rust

# FFI - Rust is not an island
It surprise you to know that Rust is not the only programming language in the world. Another goal of programming languages is being able to use/be used by those other programming languages. The tools a language provides to do that is called it's *Foriegn Function Interface (FFI)*.

As it so happens, the C language is ubiqutous: most other languages are interoperable with it. As a result, if Rust can "talk to" C libraries, it can work with pretty much anything

## Linking to a C library
Assume you have a C library whose source code has the following lines:
```c
typedef *void CURL;
CURL curl();
```

We want a way to call `curl` in Rust. You can do so like this
```Rust
use std::ffi::*;

#[link("libcurl")]
extern "C" fn curl() -> const *c_void {}

fn main() {

    let ptr = unsafe { curl() };

    println!("{:?}", ptr);
}
```

There are several things going on here:
- The `link` attribute tells the compiler to link to a library named 'libcurl'. i.e. this is where it expectes the function to be defined
- `extern "C"` tells the compiler that the `curl` function is defined elsewhere and is formatted as a c function
- The return type of the function is a C void pointer. The Rust standard library has a bunch of types equivalent to the basic C types, in this case `c_void`.
-- The basic rust types that you've been using so far are typically NOT equivalent to their similarly named counterpart in C. For example
--- `char`s in C are just a 1 byte integer, a `char` in Rust is a UTF8 symbol. 
--- `int`s in C can have different sizes depending on your architecture, it may not be equivalent to `i32`
- The compiler has no idea of whats going on in the function and thus cannot ensure it's memory saftey. Hence you may only call `extern "C"` functions in an unsafe block.

Unlike some other languages, Rust tries to link to C code at compile time. An alternative would be to define calling conventions that you coerce all your c functions into before the receiving language invokes them.
For example python (natively) requires you to wrap all your functions in a `PyObject *`, *IN C*, which is then called by the python interperator.
Rusts approach means you have to put more effort into making sure that your binding link correctly because there are more potential forms your c functions can take.

### Exposing code to C
Rust symbols don't are't formatted the same way as they would be in C, usually. Prefacing a struct/enum with `#repr(C)` will tell the compiler to align it in a C-compatible way.

```rust
#[repr(C)]
enum JustNumers {}
```

### I don't like generating 'extern "C" {...}' manually
Luckily there are a bunch of tools generate all these "bindings" automatically
- [bindgen]() - C/C++ library to Rust code
- [cbindgen]() - Rust code