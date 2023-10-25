# Polymorphism
It's a concept that you often see in object oriented languages. Roughly, polymorphism means 'single interface, multiple implementations', and in rust it's basically what it sounds like: defining the same function/enum/struct on multiple types.

# Traits
Before we can do that, we need a way of sharing the same functionality among multiple types. We do this by first defining a *trait*, which contains a number of function definitions
```rust
trait ATrait {
    // Types that impl this trait must implement this function
    fn foo1(self) -> Self;
    fn foo2(&self) -> i32;

    // Types that impl this trait will have this function automatically, but
    // can have an overidding implementation, if you want
    fn overiddable(&self) -> &'static str {
        "Nothing"
    }

    // Types that impl this trait and Add<i32, Output=i32> will
    // have this function
    where Self: Add<i32, Output=i32>
    fn add_four(self) -> i32 {
        self + 4
    }
}
```
Then, you `impl` this trait for your types

```rust
struct AType {
    adata: i32,
}

impl ATrait for AType {
    fn foo1(self) -> Self {
        Self { adata: self.adata + 4}
    }

    fn foo2(&self) -> i32 {
        self.adata
    }

    fn overiddable(&self) -> &'static str {
        "Actually, I like this String"
    }
}
```

## Deriving
Many traits can be `impl`ed easily by using the `derive` procedural macro, especially traits in the standard library.
```rust

#[derive(Debug)]
struct WouldBeAnnoyingToDebugManually;

println!("{:?}", WouldBeAnnoyingToDebugManually {});
```

Unfortunately, actually defining a procedural macro is beyond the scope of this course.

## Traits on foriegn types
Traits in Rust are similar to Interfaces in lanuages like Java and C#. The difference is that you can define traits for types that you did not define.
```rust
impl ATrait for f32 {
    fn foo1(self) -> Self {
        self - 4.0
    }
    fn foo2(&self) -> i32 {
        self as i32
    }
}
```
This feature is used extensivly in crates like [`num_traits`](https://docs.rs/num-traits).


## Traits for inheritance
Unlike classes in object-oriented languages, traits do not directly support inheritance. You can simulate it, however, by restricting a trait to types that `impl` a 'parent' trait.

```rust
trait BTrait
where Self: ATrait {
    fn bar1(&self) -> String
}
```

Of course you still would have to write an `impl` block to implement the parent trait on your types. Certain 'class' heirarchies allow you to chain `derive` each member of the hierachies in one line (for example `#[derive(PartialEq, Eq)]`)

# Generics
You can create stuff that works on multiple types by passing the **types themselves** as *generic* parameters.

```rust
/// You can use multiple generic types at once
struct GenTuple<T, S> {
    t: T,
    s: S,
}

/// You can create enums specific to each type
enum GenVariant<T> {
    Zero,
    One(T),
    Two(T, T),
    WithNonT(T, i32)
}


fn gen_function<T>(thing: T) -> T {
    // Can't really do much, so just return the argument
    T
}

// traits themselves can also be generic
trait gen_trait<T> {
    fn rereturn(self) -> Self {
        // again we can't really do much because we don't know anything about T yet
        self
    }
}
```

Look complicated? Well, remember that you've been using the generic types that you've been using already.

- `Option<T>`
- `Result<T>`
- `Vec<T>`
- `Add<T, Output=Self>` (Used every time you use `+` operator)

## Trait bounds
In order to actually perform operations on your data, you need to restrict you generics to types that implement a set of traits. You do this with the `where` clause under the item declaration.

```rust
/// This function requires that x be a type that can be added to a T to produce another T
/// and that x be copyable. All the basic numeric types satify this bound
fn double<T>(x: &T) -> T
where T: Add<T, Output=T> + Copy
{
    *x + *x
}

/// If you're bounds for each type is short you can declare them inline,
/// forgoing a where clause altogether
fn triple<T: Mul<T, Output=T> + From<i32>>(x) {
    *x * 3.into();
}

triple(3);

/// Traits can also limit themselves to types that implement other traits
trait DoubleAble
where Self: Add<T, Output<T>> + Copy {
    fn double(&self) -> T;
}

```


***Have A think.*** When you make trait constraits, you're balancing between tradeoffs. The more traits the generic types implement, there more you can do with them in the functions, but the less concrete types this could apply to. Similarly, if you have a function that supports many types, you need to retain support for all those use cases in future version of your library.


Great! You've defined some traits and generic functions that use them as parameters, so how do we call those functions? There are two broad catagories: *static* and *dynamic* dispatch.

## Static dispatch


To actually perform static dispatch, the compiler creates a version of the function for each of the types it's used on, and simply replaces the generic function call with that specific implementation. This is process is called *Monomorphisation*.

You may be thinking: *wouldn't this result in alot of dead binary code?* Thankfully, the compiler will only write versions of generic functions you actually use, not necessarily all the versions that are defined. This is an aspect of Rusts *Zero-Cost-Abstractions*: if you don't use a feature, it shouldn't cost you anything.


## Dynamic dispatch
Say you want to create a collection of a trait, or you want to write a function that returns a type with a trait but don't neccessarily care what the actual type is. (The OOP analogy would be returning an instance of a Base class, with many Derived implementations). Unfortunately, not all types with that trait may have the same layout/size, and size really matters to the compiler.

We can get around this with a `Box` of a trait object.

```rust
let printables: Vec<Box<dyn Display>> = vec![14, 15.6, "I'm with the numbers"];

for p in printables {
    println!("{p}");
}

```

#### Trait Objects
Creating a trait object, it's a simple as writing `dyn MyTrait`. A trait object contains the data of the underlying concrete type and a pointer to a *virtual function table* (v-table) of all functions on that type involved in implementing the trait. Their layout may look something like:

```rust
struct TraitObject<T: Trait> {
    vtable: const *()
    // could hold any data, like (void *) in c, making this struct not Sized
    data: const *,
}
```


It's important to note that `dyn MyTrait` *^is itself** actually a concrete type. But remember, it is not `Sized. So being able to use them, even in boxes requires certain restrictions. For example the following

```rust
trait MyTrait {
    fn return_self(self) -> Self;
}
```
Will not compile. This is because not all types that impl `MyTrait` is not sized, namely `dyn MyTrait`.
This can be resolved by restriction `MyTrait` to types that impl `Sized` like so:

```rust
trait MyTrait: Sized 
{
    fn return_self(self) -> Self;
}
``` 

However this now means that `dyn MyTrait` cannot be defined, as it can **only** be unsized. Cosequently, you will not be able to dynammically dispatch `MyTrait`.

More specificly, in order for Rust to implement `dyn MyTrait`, `MyTrait` must be *object safe*. From the [Rust Reference](https://doc.rust-lang.org/beta/reference/items/traits.html#object-safety), this means that a trait must:

- Must not return `Self`
- Not have any associated functions
- The `self`-like parameter's type must be a reference to `Self`

That is, all functions in `MyTrait` must take the form:
```rust
fn foo1(&self, ...)
fn foo2(&mut self, ...)
fn foo3(self: &Self, ...)
fn foo4(self: Box<Self>)
fn foo5(self: Arc<Self>)
// etc...
```
Technically, it **is** possible to have such functions to be part of a trait, but you will have to corden them off.

```rust
trait MyTrait {
    where Self: Sized
    fn return_self(self) -> Self;

    where Self: Default
    fn zero() -> i32 { 0 }
}
```
As not all types will impl those functions, neither will `dyn MyTrait`, so you cannot use them in dynamic dispatch.

## Static vs Dynamic Dispatch

Static dispatch:
- Doesn't imvolve any inderection and allows for compiler optimasation like [inlining](https://en.wikipedia.org/wiki/Inline_expansion), so it is generally faster
- Retains Type information
But
- Is less flexible
- monomorphisation can bloat the binary

Dynamic dispatch
- Is flexible, as they can hold various concrete types
But
- Require inderection and usually heap allocation, so it is generally less performant
- Discards Type information outside of it `impl`ing the dispacted trait. (Type Erasure)


## Iterators. Again
### MyVecIterator
```rust
struct MyVecIterator<T> {
    curr_ind: usize,
    v: Vec<T>,
}

impl<T> Iterator<Item=T> for MyVecIterator<T> {
    fn next(&mut self) -> Option<Self::Item> {
        if (self.curr_ind < self.v.len()) {
            None
        }
        else {
            self.curr_ind += 1;
            Some(v[self.curr_ind - 1])
        }
    }
}

```


### FibbonacciVec
```rust
struct FibIter {
    first: u64,
    second: u64,
}

impl FibIter {
    fn new() -> Self {
        Self {
            first: 0,
            second: 1,
        }
    }
}

impl Iterator<Item=u64> for FibIter {
    fn next(&mut self) -> Option<Self::Item> {
        let third = self.first + self.second;
        self.first = self.second;
        self.second = third;

        Some(third)
    }
}
```