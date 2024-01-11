# Macros

## Metaprogramming

The exact definition of *metaprogramming* is actually quite contested. One offered by [wikipedia is]() "a program that treats code as data".

For our purposes, metaprogramming means "writing code that writes code", which is pretty broad

Under this definition, flavours of metaprogramming include:
### External langauge API
- Java SQL allows allows you to write SQL queries by method chaining Java objects. The objects are then translated to raw strings to be queried by the database at runtime. This helps protect against SQL injections, keeps more of your codebase in one language and is generally more Java-y. 



### Language Frameworks
Some programmings languages allow you to download packages that change the semantics of the languge itself. This can be useful for extending the language, or reducing boilerplate

- The Lombok project for Java will automatically add getter/setter functions for fields in you classes.
- Qt is a GUI framework where users may write gui application in slightly extended version of c++. To create your own widgets, you declare classes that inherit from a Qt object. To make them react to events (like when a button is pressed), you declare a special `signal:` section. This is similar to Rusts attribute proc-macros

### Text Replacement
Some languages provide a sublanguage of sorts that the compiler will use to insert code items into your source code at compile time.
These are known as *macros*. Rust has these, so they'll be the primary topic of discussion in the following section.

But also, many build tools (including cargo's `build.rs`) allow you to generate intermediate intermediate files based on your source files at build time. It can be useful if your text replacement needs are dependant on data competley outside your source files.

## Macros. A discussion
### In C
To give some context as to why Rusts handles macros the way it does, lets look at a typical Macro in C
```c
#define MIN(A, B) A < B ? A: B;
```


*What could possibly go wrong?*

As you might have guessed alot of things can easily go wrong. 

- Order of Operations:

- Expanded expressions get evaluated:

- Evaluated expressions can change state

All of it stems from the fact that C's preprocessor is <u>stupid</u> (by design) and <u>only</u> performs text replacement. It has absolutely information on the nature of your program.




# Function (traits)

Functions in Rust are treated as first class objects. This means that you can pass them around as a variable, use them as function arguments and even impl traits on them. Well, it's slightly more complicated than that. In Rust, functions are passed as objects that impl one of these traits:
| --- | --- |
| FnOnce | Can be called at least once. May be invalid afterwards |
| FnMut | May mutate state, so it could have different behaviour with repeaated called. Callable many times |
| Fn | Will not mutate state and callable many times. Basically a pure function |
| --- | --- |

The above table is in **increasing** order of strictness. All Fn impl FnMut who all impl FnOnce.

## As function arguments

## Closures
Like in C, defining a function with the `fn` keyword as you have so far merely defines a function pointer to the memory in in which the function instructions are stored. These pointers impl an appropriate function trait.

### FUNCTIONAL PROGRAMMING!!! WOOO
Let's look at some situations where you might want to consider using closures
```rust
struct VeryExpensive(i32);
impl VeryExpensive {
    fn new() -> Self {
        let mut x = 0;
        for i in 1..5000000 {
            x += i;
        } 
        Self(x)
    }
}

// unwrap_or method evaluates to a provided default value if the
// Option is None. However, this method requires the default value to 
// to be constructed EVERY TIME, irregardless if it ends up being used
let x = None.unwrap_or(VeryExpensive::new());

// This is, undesirable if contructing the value is computationally costly/mutates external state
// In thiis case, providing a closure that constructs on demand may be preferable
let x = None.unwrap_or_else(|| { VeryExensive::new() });

```

Multithreading. Covered in [Week 8](week_8)