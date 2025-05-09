# Project

--

Install Rust

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

Start a new project:

```bash
cargo new demo_project
```

--

### Project Anatomy

```project
demo_project
    |
    |--/src
    |   |- main.rs  <-- source entry point
    |
    |--/target
    |   |-...       <-- compiled source code
    |
    |--.gitignore
    |
    |-- Cargo.toml  <-- project entry point
    |-- Cargo.lock  <-- dependency version pin
```

--

### Cargo.toml

```toml
[package]
name = "demo" # package name
version = "0.1.0" # Semantic version
edition = "2024" # Pins the feature set

# See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html

[dependencies]

```

--

### main.rs

```rust
fn main() {
    println!("Hello, world!");
}
```

```bash
cargo run
# or
cargo run --release
```

```bash
rust-presentation on  main [?] is 󰏗 v0.1.0 via 󱘗 v1.85.0 
❯ cargo run
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.00s
     Running `target/debug/demo_project`
Hello, world!
```

---

# Syntax Basics

--

`let` introduces a variable binding:

```rust
let x; // declare "x"
x = 42;
```

```rust
let x = 42; // declare in one line
```

<https://doc.rust-lang.org/rust-by-example/primitives.html>

--

All variables are **immutable** by default

```rust
let x = 42;
x = 69
```

Compile time error:

```bash
error[E0384]: cannot assign twice to immutable variable `x`
 --> src/main.rs:3:5
  |
2 |     let x = 42;
  |         - first assignment to `x`
3 |     x = 69
  |     ^^^^^^ cannot assign twice to immutable variable
  |
help: consider making this binding mutable
  |
2 |     let mut x = 42;
  |         +++
```

--

Enable **mutability** with _`mut`_ keyword:

```rust
let mut x = 42;
x = 69;
```

--

### specify the type with "`:`"

```rust
let x: i32; // signed 32-bit integer
x = 42;
```

```rust
let x: i32 = 42; // declare in one line
```

| |
| -- |
| signed i8 i16 i32 i64 i128 isize |
| unsigned u8 u32 u64 u128 usize |
| float f32 f64 |

--

Compiule time error to use an uninitialized variable:

```rust
let x;
foobar(x);
x = 42;
```

```bash
cargo run

error[E0381]: used binding `x` is possibly-uninitialized
 --> src/main.rs:7:12
  |
6 |     let x;
  |         - binding declared here but left uninitialized
7 |     foobar(x);
  |            ^ `x` used here but it is possibly-uninitialized

For more information about this error, try `rustc --explain E0381`.
```

runtime error in C?

--

This is fine though:

```rust
let x;
x = 42;
foobar(x) // <-- the type of 'x' will be inferred from here
```

types inferred where unambiguous

--

Underscore "`_`" is a special name for an unused variable:

```rust
// this does *nothing* because 42 is a constant;
let _ = 42;
```

```rust
// this calls `try_thing` but throws away its result
let _ = try_thing();
```

```rust
// we may use `_x` eventually, but our code is a work-in-progress
// and we just wanted to get rid of a compiler warning for now
let _x = 42;
```

--

Separate bindings with the same name can be introduces:

```rust
let x = 13
let x = x + 3;
// using `x` after that linen only refers to the second `x`,
// the first `x` no longer exists.
```

This is _shadowing_ a variable binding

--

We also have other basic types:

| | |
| -- | -- |
| bool | true, false |
| char | unicode character |
| &str | utf-8 string literal |

---

# More Syntax

--

### Tuples

(fixed-length collections)

```rust
let pair = ('a', 17);
pair.0; // this is 'a'
pair.1; // this is 17
```

Annotate the type:

```rust
let pair (char, i32) = ('a', 17);
```

_Destructure_ the output:

```rust
let (some_char, some_int) = pair; // variables are named in snake_case
```

Very useful when a function returns:

```rust
let (left, right) = slice.split_at(middle);
```

--

The semi-colon marks the end of a statement:

```rust
let x = 3;
let y = 5;
let z = y + x;
```

Which means statements can span multiple lines:

```rust
let x = vec![1, 2, 3, 4, 5, 6, 7, 8]
    .iter()
    .map(|x| x + 3) // <-- this is closure syntax
    .fold(0, |x, y| x + y);
```

--

Fixed length (static) arrays:

```rust
let my_array: [char; 3] = ['a', 'b', 'c'];
```

Variable length (heap-allocated) arrays are Vecs:

```rust
let my_vec: Vec<char> = vec!['a', 'b', 'c'];
```

Declare as a range:

```rust
let range = 0..5; // 0,1,2,3,4
```

--

These are lazily loaded, meaning we can happily declare a range to infinity:

```rust
use std::{thread, time::Duration};

fn main() {
    for i in 0.. { // all positive integers
        println!("{}", i);
        thread::sleep(Duration::from_secs(1));        
    }
}
```

--

We also have other collection types:

| | |
| -- | -- |
| String | Variable length string |
| VecDeque | First in first out array |
| HashMap | Like a python dict |
| BTreeMap | Ordered map |

...

---

# Borrowing & Ownership

--

### Ownership Principles

| | |
| -- | -- |
| Single Owner | Each value has a single owner, the variable that binds to it. |
| Scope-Bound | It is dropped when its owner goes out of scope, freeing resources |
| Transfer | Ownership is transferred through assignment, preventing data races. |

--

Ownership transferred

```rust
let a = vec![0, 1, 2];
let b = a;                      // <-- ownership transferred here
println!("{:?}, {:?}", a, b);   // <-- cannot use 'a' any longer
```

<https://github.com/cordx56/rustowl>

but a nice compiler error

```bash
error[E0382]: borrow of moved value: `a`
  --> src/main.rs:10:28
   |
8  |     let a = vec![0, 1, 2];
   |         - move occurs because `a` has type `Vec<i32>`, which does not implement the `Copy` trait
9  |     let b = a; // <-- ownership transferred here
   |             - value moved here
10 |     println!("{:?}, {:?}", a, b); // <-- cannot use 'a' any longer
   |                            ^ value borrowed here after move
```

--

### Borrowing Rules

| | |
| -- | -- |
| References | Borrowing is done through references, allowing access to data without taking ownership. |
| Immutable Borrows | Multiple immutable references (&T) allowed, ensuring read-only access. |
| Mutable Borrows | Or ONE mutable reference (&mut T) at a time, preventing data races. |

--

Many read-only borrows allowed

```rust
let a = vec![0, 1, 2];
let b = &a;                             // <-- borrow here
let c = &a;                             // <-- many times
println!("{:?}, {:?}, {:?}", a, b, c)   // can continue to use 'a'
```

--

Only one borrow if mutable

```rust
let mut a = vec![0, 1, 2];
let b = &mut a;
let c = &a      // <-- cannot borrow 'a' as immutable...
println!("{:?}, {:?}, {:?}", a, b, c)   // error
```

enforced by the compiler:

```bash
error[E0502]: cannot borrow `a` as immutable because it is also borrowed as mutable
  --> src/main.rs:10:9
   |
9  | let b = &mut a;
   |         ------ mutable borrow occurs here
10 | let c = &a      // <-- cannot borrow 'a' as immutable...
   |         ^^ immutable borrow occurs here
11 | println!("{:?}, {:?}, {:?}", a, b, c)   // error
   |                                 - mutable borrow later used here
```

--

### Lifetimes

| | |
| -- | -- |
| Safety | They ensure references are valid as long as they are used. |
| Annotation | In non-trivial scenarios, they clarify the scope of references. |
| Static Analysis | The compiler relies on function signatures for correctness. |

```rust
fn find_longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```

Can be complex to understand for new users

--

### Benefits

| | |
| -- | -- |
| Memory Safety | Eliminates common bugs (e.g. dangling pointers, data races) without a garbage collector. |
| Concurrency | These rules naturally lead to safer concurrent code execution. |

--

### Drawbacks

| | |
| -- | -- |
| Complexity | Lifetime signatures can propogate through a codebase. |
| Weirdness | Adds to the strangeness budget of learning a new language. |

<https://steveklabnik.com/writing/the-language-strangeness-budget>

---

# Functions

--

_`fn`_ declares a function:

```rust
// void function
fn greet() {
    println!("Hi there!");
}
```

the compiler's got your back if you accidentally write python code

```bash
error: expected one of `!` or `::`, found `greet`
 --> src/main.rs:1:5
  |
1 | def greet():
  | --- ^^^^^ expected one of `!` or `::`
  | |
  | help: write `fn` instead of `def` to declare a function
```

--

return a value from a function:

```rust
fn roll_a_dice() -> i32 {
    return 4;
}
```

but actually we can just omit the semi-colon to return the last value:

```rust
fn roll_a_dice() -> i32 {
    4
}
```

---

# Blocks

--

Braces declare a block:

```rust
// this prints "in" then "out"
let x = "out";
{
    // this is a difference 'x'
    let x = "in";
    println!("{}", x);
}
println!("{}", x);
```

--

Blocks are also expressions:

```rust
// this:
let x = 42;

// is equivalent to this:
let x = { 42 };
```

--

Inside a block, there can be multiple statements:

```rust
let x = {
    let y = 1;
    let z = 2;
    y + z // this is the *tail* that will be returned
};
```

Which behaves like an immediately invoked function expression (IIFE)

--

`if` Conditionals are also expressions:

```rust
let score = {
    if feeling_lucky {
        6
    } else {
        1
    }
}
```

A `match` is also an expression:

```rust
let score = match feeling_lucky {
    true => 6,
    false => 1,
}
```

--

So is a loop:

```rust
let mut counter = 0;
let result = loop {
    counter += 1;
    if counter == 10 {
        break counter * 2;
    }
};
```

--

### Accessors

Dots typically access fields of a value:

```rust
let a = (10, 20);
a.0 // this is 10
```

or methods:

```rust
let statement = "rust is fun";
statement.len() // this is 11
```

--

### Namespaces

Double-colon **::** is similar but operates on namespaces

```rust
let least std::cmp::min(3,8); // this is 3
```

`use` brings into scope from other namespaces:

```rust
use std::cmp::min;

let least = min(7,1); // this is 1
```

A wildcard (\*) lets you import every symbol from a namespace:

```rust
use std::cmp::*;
```

--

Types are namespaces too:

```rust
let x = "james".len(); // this is 5
let x = str::len("james"); // this is also 5
```

Many types are in scope by default:

```rust
// `Vec` is a regular struct, not a primitive type
let v = Vec::new();

// this is exactly the same code but with the *full* path to `Vec`
let v = std::vec::Vec::new();
```

This works because Rust inserts this in the beginning of every module:

```rust
use std::prelude::v1::*;
```

---

# Structs

--

Declare a Struct:

```rust
// structs are always named in PascalCase
struct Coord {
    x: f64,
    y: f64,
}
```

Initialize instances using struct literals:

```rust
// the order does not matter, only the names
let a = Coord { x: 1.0, y: 3.1 };
let b = Coord { y: 5.5, 66.0 };
```

There is a shortcut for initializing the rest of the fields from another instance:

```rust
let c = Coord {
    x: 14.0,
    ..b // <-- get the remaining fields from struct b
};
```

--

By default a struct has minimal methods

```rust
#[derive(Debug, Copy, Clone, PartialEq, Eq, PartialOrd, Ord)]
struct Coord {
    x: f64,
    y: f64,
}
```

But opt in to behaviour with `derive`

--

Structs can have no fields:

```rust
struct UnitStruct;
```

or anonymous fields:

```rust
struct TupleStruct(int, bool);
```

--

Declare methods on a struct with an `impl` block

```rust
struct Coord {
    x: f64,
    y: f64.
}

impl Coord {
    /// Calculates the Euclidean distance between two coordinates
    fn distance_to(&self, other: &Coord) -> f64 {
        let dx = self.x - other.x;
        let dy = self.y - other.y;
        (dx.powi(2) + dy.powi(2)).sqrt()
    }
}
```

--

## Struct Memory Layout

| Feature | Rust | C |
| -- | -- | -- |
| Memory Layout | Undefined by default | Defined and predictable |
| Compiler Optimizations | Can reorder to reduce padding and optimize memory usage | No field reordering, manually optimize field order to reduce padding |
| Field Reordering | Allowed - minimizes memory size & padding | Not allowed |
| Stable ABI | Not guaranteed between compiler versions | Guaranteed, including a predictable memory layout |

<https://doc.rust-lang.org/reference/types/struct.html>

--

#### Unless you really care

Default representation, alignment lowered to 2:

```rust
#[repr(packed(2))]
struct PackedStruct {
    first: i16,
    second: i8,
    third: i32,
}
```

C representation, alignment reaised to 8:

```rust
#[repr(C, align(8))]
struct AlignedStruct {
    first: i16,
    second: i8,
    third: i32,
}
```

<https://doc.rust-lang.org/reference/type-layout.html#representations/>

---

# Enums

## (with super powers)

--

Enums declared like this:

```rust
// enums also named in PascalCase
enum Direction {
    Up,
    Down,
    Left,
    Right,
}
```

They can be initialized like this:

```rust
let w = Direction::Up;
let a = Direction::Left;
```

--

Enums are union types

They consume only as much memory as the largest variant

--

And they can hold data, as each variant is also a struct:

```rust
enum Message {
    Quit,                       // <-- Union Struct
    Move { x: i32, y: i32 },    // <-- Named Struct
    Write(String),              // <-- Tuple Struct
    NestedMessage(Message),     // <-- Recursion Possible
}
```

--

### Just like a struct

Opt in to behaviour with `derive`

```rust
#[derive(Debug, Clone, Copy)]
enum Cat {
    Alive { hungry: bool },
    Dead,
}
```

Define methods with `impl` blocks

--

Two important enums exist

They are just normal enums, but their variants are in scope by default

They use generics to make them very flexible

| Type | Happy | Unhappy | Use |
| --- | --- | --- | --- |
| `Option<T>` | Some(T) | None | To represent Null |
| `Result<T, E>` | Ok(T) | Err(E) | To represent an error |

--

### Option

| | |
| -- | -- |
| Some(T) | None |

Sometimes things don't exist yet.

```rust
let result = database.get("some_key");

match result {
    Some(entry) => println!("found {entry}"),
    None => println!("not found"),
}
```

Rust requires you to be explicit about this.

--

### Result

| | |
| -- | -- |
| Ok(T) | Err(E) |

Sometimes things can go wrong.

```rust
match Mqtt::connect("mqtt://localhost::1883") {
    Err(error) => println!("oops somethign went wrong: {:?}", error),
    Ok(client) => {
        // use the client in this scope
    }
}
```

Again Rust helps you be explicit about when things are fallible.

--

### helper patterns

if let

```rust
if let Ok(client) = Mqtt::connect("mqtt://localhost::1883") {
    client.subscribe("topic");
};
```

let else:

```rust
let Ok(client) = Mqtt::connect("mqtt://localhost::1883") else {
    panic!(); // must diverge here by panicking or returning early from a function
};
client.subscribe("topic");
// use client for the rest of the scope.
```

--

This pattern is so common there is a special syntax to shorten it further using "`?`"

```rust
/// Attempt to connect to the broker and subscribe to a topic.
fn connect(url: &str, topic: &str) -> Result<Client, MqttError> {
    let mut client = Mqtt::connect(url)?;
    client.subscribe(topic)?;
    Ok(client);
}
```

or shorten to:

```rust
fn connect(url: &str, topic: &str) -> Result<Client, MqttError> {
    Ok(Mqtt::connect(url)?.subscribe(topic)?)
}
```

Not accounting for all patterns is a compile time error

```bash
error[E0004]: non-exhaustive patterns: `Suit::Hearts`, `Suit::Diamonds`, `Suit::Clubs` and 1 more not covered
 --> src/main.rs:9:11
  |
9 |     match suit {}
  |           ^^^^ patterns `Suit::Hearts`, `Suit::Diamonds`, `Suit::Clubs` and 1 more not covered
  |
```
