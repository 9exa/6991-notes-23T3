# Week 2
# Collections and iterators


## My best friend: The Standard Library documentation
All of the collections mentioned here are implemented in Rusts' Standard Library, so it's [documentation](https://doc.rust-lang.org/std/index.html)
will provide a more in depth description of their methods.

### Array
- Contiguous collection of a type,
- Size is declared statically, cannot be changed
- Elements accessed by index O(1)
- Allocates memory on the stack 😎

### Vec
- Dynamically sized contiguous collection of a type,
- Elements added and removed at the back. First-in-Last-out style O(1)
- Inserting/Removing elements in other indexes is O(n)
- Elements accessed by index O(1)

### Slice
- Pointer to beginning and size of a contiguous collection of a type.
- Basically a reference to an Array, but can be generated from other collections as well

```rust
// Oh look an array
let a = [1, 2, 8, 9, 2, -1, 4, -5, -6];

// making the same Vec inline
let a2 = vec![1, 2, 8, 9, 2, -1, 4, -5, -6];
let a2 = Vec::from([1, 2, 8, 9, 2, -1, 4, -5, -6]);
let mut a2 = Vec::from(a);

// Lets get a value
// careful this will panic if the index does not exist in a2
let x = a2[4];
// much safer, but it's an Option
let x = a2.get(4);

// adding value to the back
a2.push(17);
// adding value somewhere else. :< slow
a2.insert(4, 71);

// removing value from the back
println!("{}", a2.pop());
// removing value somewhere else. :< slow
println!("{}", a2.remove(5));

// taking all but the first 3 elements as a slice
let a_slice = &a2[3..];

// iterating over the slice/array/Vec
a_slice.into_iter()
    .for_each(|x| print!("{x}, "));
println!("");

// To change variables within the collection,
// we need mutable references
let mut a_m_slice = &mut a2[3..];
a_m_slice.into_iter()
    .for_each(|x| *x = 69);

println!("A somewhat nice Vec: {:?}", a2);

```

### String
- Dynamically sized contiguous collection of chars,
- Plethora of methods to make manipulating words and sentences easier.
- Can be `Deref`ed to a `&str`

```rust
// Equivalent ways of creating an empty String
let s = String::new();
let s = "".to_string();
let s = String::from("");

// Concatenating. A method native to String. Notice the rhs is a &str
let s = s + "AAAAAAHHHHHHH";

// An &str is often called a 'string slice'. This is why
let substr: &str = &s[3..9];
println!("{}", substr);


```

### VecDeque
- Double ended queue of a type
- Elements can be added and removed at both the front and back. O(1)
- Elements accessed by index O(1)
- Inserting/Removing other elements is O(1)
- Not gaurenteed to be contiguous in memory

### LinkedList
- Series of nodes containing a type and a pointer to the next node
- Elements Added and Removed in the frontt or back O(1)
- Cursor Api allows removal elements elswhere in O(1) time, however i is currently experimental.
- In general, usual usage of LinkedLists conflict with Rust's borrowing system, so it might be better to use [VecDeque](#VecDeque)  

## Maps
### HashMap
- Collection of key-value pairs.
- Internally managed using a hashtable
-- Keys must implement `Hash` and `Eq`
-- Each key is unique
- Values are accessed via `[key]` as reference to the value associated with key. Panics if key has not been assigned. Expected O(1)
- Values are accessed via `.get(key)` as an Option. Expected O(1)
- Values are added/changed via `.insert(key, value)`. Expected O(1)
- Values removed via `.remove(key, values)`. O(n)
- No gaurentees with the order of items when iterating

### BTreeMap
- Collection of key-value pairs.
- Internally managed using [B-Tree](https://en.wikipedia.org/wiki/B-tree). See std documentation for a discussion as to why
-- Keys must implement `Eq`
-- Each key is unique
- Values are accessed via `.get(key)` as an Option. Expected O(log(n))
- Values are added/changed via `.insert(key, value)`. Expected O(log(n))
- Values removed via `.remove(key, values)`. O(log(n))
- Items iterated in ascending order w.r.t. their keys 

```rust
use std::collections::{HashMap, BTreeMap};

/// Equivalent methods of creating a hashmap
let hmap = {
    let mut h = HashMap::new();
    h.insert("zero", 0);
    h.insert("one", 1);
    h.insert("two", 2);
    h
};
let hmap = HashMap::from([("zero", -0), ("one", 1), ("two", 2)]);
/// Actually, I like trees better
let mut bmap: BTreeMap<_, _> = hmap.into_iter().collect();

// Replacing a value returns an Option with old one
println!("Kicked out {:?}", bmap.insert("zero", 0));


// The Entry Api can help with the 'insert default' tasks
for k in ["two", "three"] {
    bmap.entry(k).and_modify(|x| *x += 1).or_insert(1);
}

println!("{:?}", bmap);

```


## Sets
### HashSet
- Collection of unique values.
- Implemented with a hashtable.
- No gaurentees with the order of items when iterating

### BtreeSet
- Collection of unique values.
- Implemented with a B-Tree.
- Values iterated in ascending order

## Other
### BinaryHeap
- A priority queue implemented with a max-heap binary heap
- Values must implement `Ord`
- Values accessed from descending order

etc...

### Iterators
Often, we put data into the collections described above in order to apply an operation to/using each element in them. To do this, most 'collection' types have an associated [Iterator](https://doc.rust-lang.org/std/iter/trait.Iterator.html#method.reduce). You may have encountered them in other languages, as iterators are heavily used in the [Functional](https://en.wikipedia.org/wiki/Functional_programming) style of programming. 

```rust
/// In Rust a for loop silently calls the into_iter() method of the right most argument. 
/// So you've actually been using iterators from day 1
let v: [i32; 9] = [1, 2, 8, 9, 2, -1, 4, -5, -6];
let u = v.clone();

for x in v {
    print!("{x}, ");
}
print!("\nIs equivalent to\n");
u.into_iter()
    .for_each(|x| print!("{x}, "));

```
The method `.into_iter()` moves elements as it iterates, along with the entire collection. The methods `.iter()` and `iter_mut()` iterate over references to each element, preserving the collection.

```rust
// iter provides read-only references
let mut v: [i32; 9] = [1, 2, 8, 9, 2, -1, 4, -5, -6];
for x in v.iter() {
    println!("Found {x}.");
}
println!("{v:?}");

// iter_mut provides mutable references
for x in v.iter_mut() {
    println!("Found {x}, changing it.");
    *x = -*x;
}
println!("{v:?}");
```


As an example lets say we want to find the mean of the numbers in an array, but to stop once we encounter a negative number.
#### Imperative Method
```rust
let v: [i32; 9] = [1, 2, 8, 9, 2, -1, 4, -5, -6];
let (mut total, mut n) = (0, 0);

for num in v {
    if num < 0 {
        break;
    }

    total += num;
    n += 1;
}

let mean = if n == 0 { 0 } 
    else { total / n };

assert_eq!(mean, 4);

```

##### Advantages
- More explicit
- No hidden behaviours
- (Slightly) easier for compiler to optimise
- More natural to insert if statements to gaurd for specific cases

##### Disadvantages
- Intermediate variables can clutter code
- Minutia often obscures overall idea behind operation

#### Functional Method
```rust
let v: [i32; 9] = [1, 2, 8, 9, 2, -1, 4, -5, -6];

let (total, n) = v.iter()
    .take_while(|num| !num.is_negative())
    .fold((0, 0), |(total, n), x| {
        (total + x, n + 1)
    });
let mean = total / n;

assert_eq!(mean, 4);

```

##### Advantages
- Declarative
- Nerds love it. [Something about formal correctness or robustness or something]{https://www.handbook.unsw.edu.au/undergraduate/courses/2024/COMP4161?year=2024}

##### Disavantages
- Declarative
-- Can have hidden behaviours. E.g. Allocating dynamic memory
- Requires all team members to know the meaning of functional methods used
- Might need to import more specific operations from other libraries. See [Itertools]

#### Mixed Method
```rust
let v: [i32; 9] = [1, 2, 8, 9, 2, -1, 4, -5, -6];

let (mut total, mut n) = (0, 0);
for x in v.iter().take_while(|num| !num.is_negative()) {
    total += x;
    n += 1;
}
let mean = total / n;

assert_eq!(mean, 4);

```
##### Advantages
- Pick and choose the advantages and disadvantages of both methods

##### Disavantages
- Can lead to a somewhat inconsitent codebase

### Bonus! Pattern Matching
Rust unpacks expressions to allow you to create variables convinently
```rust
// Only run block if a value is Some
let optional = Some(14);
if let Some(x) = optional {
    // Some code
};

// run code when a condition is not true, then exit the fuction
let Some(x) = optional else {
    // Edge case gaurding
    return ();
};

// Declare multiple variables in a single line
let (x, y, mut z) = (1, 2, 3);

// You can unpack structs too
#[derive(Clone)]
struct Point {
    x: i32,
    y: i32,
}
let p = Point {x: 1, y: 42};

let Point { x: w, y: h} = p.clone();
println!("w: {w} h:{h}");
let Point {x: w2, ..} = p.clone();
println!("I only care about w: {w2}");

// Querying multiple conditions in one match statement
let s = "Ahhhhhhhhh";
match (s.starts_with('a'), s.ends_with('h')) {
    (true, true) => println!("Both Conditions"),
    (_, true) => println!("Second only"),
    (_, _) => println!("Other stuff"),
    (true, true) => println!("Never happens because i'm covered by (_, _) :<"),
};

// You can also match on ranges
match 14 {
    0..=10 => println!("smol number"),
    11..=50=> println!("medium number"),
    51..=100=> println!("big number"),
    _ => ()
};

// or more complicated conditions with ifs
let row = match p {
    Point {y: y, ..} if y % 2 == 0 => "even", 
    _ => "odd"
};

```
