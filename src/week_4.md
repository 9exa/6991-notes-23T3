# Multi-file projects
In all but the smallest projects, you'll probably want to incorporate multiple files. Below are the building blocks of a Rust project, which determine how the build system uses those files.

## Crates
Crates are the smallest unit of compilation. That is, when you run `cargo build` or even `rustc main.rs`, you are compiling **that crate**, not a selection of modules. Each crate in the dependency tree is compiled seperately, with their own set of feature flags. They're linked up with the main crate at the end to create one binary. You create a crate every time you run `cargo new`.

Crates typically correspond to one binary file, either an executable or a binary. To clarify, an executable is a program that can be run directly by your device. A library is a collection of code to be usesd by other code. 

Technically, it is possible for a crate to build any number of executables. This can be usefull if you want to test the functionality of your library using a graphical application, for example. You may only build **one** library file per crate, though.

*Fun fact: It used to be that in order to use other local crates, you were required to write `extern [library_name]` in the root of the consuming crate. This is not the case anymore,*


### Directory trees
A typical Rust project muight look something like this

```

```

*Why would a single project have multiple crates?* Often, you may want to reuse code from one of your old projects in a new one. To that end organising code into a bunch of different libraries, each of which can be compiled seperately.s 

## Modules
Modules are the logical namespaces that your function/type definitions reside. Each file corresponds to a module. You import items from other modules with the `use` keyword.
```rust
// import the function/type/modules thingo from a file named stuff.rs or stuff/mod.rs
use stuff::thingo;
// import the multiple items from more_stuff.rs
use more_stuff::{thing1, thing2};

// import everything from an other_stuff.rs file
use other_stuff::*;

// import something from the external crate 'num'
use num::complex::Complex;

// import everything from the parent module (useful for accessing sibling modules)
use super::*;

// import a module using a path relative to the crate root
use crate::yet_more_stuff;

```

### In-file modules


### Access Levels

#### Private
Objects, including Types, functions and other modules are private by default.
This means that you can only access them in the module they are declared.

#### Public
To make an item available to other modules, you need to preface them with the `pub` keyword.

```rust

struct DontMakeMe;

// A struct that can be used by other files
pub struct MakeMe {
    // each field has it's own publicty level
    pub accessible: i32,
    non_accessible:i32,
}
// However, it is typically considered bad practise to mix publicity levels
// in struct fields. This is because we usually conceptualise types as totally opaque (all private)
// or a collection of variables (all public).

// An enum's variants are either all public or all private
pub enum CountMe {
    ImAccessible,
    MeToo,
}

// For functions to be public, all of the argument types and output types must be public
// pub fn call_me(d: DontMakeMe) // {} will no compile as it uses a private type
pub fn call_me(m: MakeMe) {}

// Similarly, public structs/enums must only have public fields.

// **MOST IMPORTANTLY** If you want to make submodules available to other files, you need to make that pub too
pub mod use_me;

```

#### Levels of publicity
You might want to write code available across your crate, while being unisable to consumers of your library. In this case you may use the followiong keywords:

`pub (super)` the item is accessible by the direct parent module
`pub (crate)` the item is from anywhere in the crate



## Testing

```rust
#[derive(Default)]
struct MyType {
    data1: i32,
    data2: String,
}

// Tells the compiler that the following module will only be compiled in a cargo test
#[cfg(test)]
mod tests {
    // because `tests` is a seperate module, 
    // in order to use the symbols in your file, you have to import them
    use super::*;

    // teslls the compiler that this function is, in fact, a test
    #[test]
    fn add_test
}

```

### Unit testing

### Integration testing

### Doctesting

## Documentation