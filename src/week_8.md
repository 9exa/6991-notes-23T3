# Concurrency

## A Pool of Threads
Up until now you've been creating single-threaded programs. A *thread* is just a stream of instructions that are run in sequence. 


Boring. In theory, each core in your machine's CPU can run their own instruction stream and there's a good chance your CPU has more than one core. Modern programs often seek to speed up their program by delegating tasks multiple programs to run in parrallel.

When you run your program, a single thread is automatically spawned that runs `main()`. The standard library lets you spawn in your own.

```rust
// thread::spawn takes in a FnOnce() -> (), and will run that function on another thread
use std::{thread, time};

fn call_me() {
    println!("I've been called!!!!")
}

thread::spawn(call_me);

// make sure that it actually runs
thread_sleep(time::Duration::from_secs(1));
```

*Note: A "thread" is an abtract concept seperate from a physical CPU core. Even single-core machines can utilize multiple threads, for example by having the operating system pass CPU recourses between active threads periodically. This is how single-core computers can run multiple programs at once*

*Other Note: According to Rob Pike, creator of GO "Concurrency is about dealing with lots of things at once but parallelism is about doing lots of things at once" (Thanks Wikipedia). So the example in the note above would be a concurrent program but not a parallel one*

*Yet Another Note: A "Process" is an operating system object that represents the running of a program. Each process contains a main thread, some metadata and is the entity that actually asks the operating system for the resources to spawn new threads*

std::thread is not actually aware of the cores on your machine, it just creates a thread and lets your operating system schedulre handle their execution. 

Some languages (Java, Go) actually have their own scheduler built into the language runtime. The advantage of this is that it reduces the number of calls to the OS (which can be quite expensive). In Rust, you'd have to implement such a scheduler yourself (use a library).

## Scheduling threads
When a main thread returns, it's process finishes and the OS terminates the remaining spawned threads. This is good right? We don't want a part of a program to keep running after you've already closed it. But, it means that a thread can be closed before it gets a chance to do anything. 

```rust
fn tell_secret() {
    println!("Listen, Kid");
    println!("I don't have much time");
    println!("The Secret to getting full marks in COMP6991 is...");
    println!("...........");
    println!("[Tom Please finish this meme]");
}

// This program may terminate before the spawned thread finishes
std::thread::spawn(|| tell_secret());

println!("Did you hear the secret?");
```
We need a function that *blocks* the main thread until the spawned thread is done. A simple solution is to just make the main thread wait for a bit with `std::thread::sleep`.

*Oh look a Note. Technically spawned threads may live slightly longer than the main thread as the Process does the command to kill it only after the main thread has already closed. This is why the function inside `thread::spawn` needs a ```rust`static```` lifetime,*

But this obviously isn't very robust. Here are a number of standard library tools that could prove more helpful.

### Handles
`thread::spawn` returns a thread handle. Using it, we can `.join()`` the creating thread with it at some point. That is, make the calling thread wait until a specified thread exits. This concepts exists in many programming languages

```rust
use std::thread;
use std::time::Duration;


let h1 = thread::spawn( || {
    sleep(Duration::from_secs(2.));
    println!("One is Done!");
});

let h2 = thread::spawn( || {
    sleep(Duration::from_secs(1.));
    println!("Two is... blue");
});

println!("They've been spawned")
// wait for the first thread
h1.join();

// The second thread should already be done at this point, so this shouldn't really block
h2.join();
```

Handles are neat because they can be passed to other threads.
```rust
h1 = use std::thread;
use std::time::Duration;


let h1 = thread::spawn( || {
    sleep(Duration::from_secs(2.));
    println!("One is Done!");
});
```

### Scopes
Recall the concept of a *scope* that you've seen in previous week. All reasources between `{` and `}` (declared variables, `use` statements, even struct definitions) are completely cleaned up by the end of the bock. Clearly this behaviour could be useful in a parrallel programming context, where you want to make sure a number of threads are complete, as an alternative to `.join()`ing multiple thread handles.

# Concurrency Gone Wrong (Not Clickbait)
Concurrency is notorious for being one of the most difficult programming techniques to actually get to work.

## Data races
The obvious complication with concurrent programming.

## Sharing information between threads
## ** IT'S TIME TO FINALLY JUSTIFY THE BORROW CHECKER **

### Send and Sync
Not all types can be accessed across threads. These constraints are expressed in the traits

Send: this type can be safely sent accorss threads
Sync: a reference to this type can be safely sent accross threads

As a corollary, a type is Sync iff a reference to it is Send. 

The declaration of Send and Sync look something like

`pub unsafe auto trait Send`
`pub unsafe auto trait Sync`


### Cells

### Mutexes

Unlike Java and C, Mutexes wrap the item it is protecting.

You should still be conservative when deciding to use them as locking and unlocking is quite expensive. 

#### Deadlock

### Arc
`Arc` allows you to share ownership of data amoung multiple variables, like `Rc`. The difference being that `Arc`s are `Send` and can share data across threads. It can do this because the reference counter in `Arc` is [atomic](https://en.wikipedia.org/wiki/Atomic_semantics), giving it performance costs that are unecessary in a single threaded setting.



### Channels

### static variables


## Third Party crates
parkinglot
rayon