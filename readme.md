# Install

- Went to [https://www.rust-lang.org/](https://www.rust-lang.org/)
- Navigated to ["Install"](https://www.rust-lang.org/tools/install)
- On Ubuntu, followed the direction to use the following command.
```{.bash}
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```
- I should note I previously had to:
```{.bash}
sudo apt install curl
```
- Install took a moment, then I restarted my terminal and verified installation via:
```{.bash}
cargo -V
```

# Starting

- Went to [https://doc.rust-lang.org/book/](https://doc.rust-lang.org/book/)
- I understand available offline via
```{.bash}
rustup doc --book
```
- That *did not* work for me and I didn't both debugging.

## ch01: Install

### 01

- The book also recommended the `curl` method for install.
- It also notes I'll need `gcc` or `clang`.
    - I had `gcc` but I also invoked:
    - I thought `clang` worked better for Rust?
```{.bash}
`sudo apt install clang`
```
- I was prompted to invoke:
```{.bash}
rustc --version
```
- I also checked for updates, which seemed unlikely to be necessary:
```{.bash}
rustup update
```
- Anticipating I may need to work online, I quick grabbed all dependencies:
```{.bash}
cargo new get-dependencies
cd get-dependencies
cargo add rand@0.8.5 trpl@0.2.0
```

### 02

- Rust recommends `~/projects`, but I'll be trying to use `~/rust/code/projects`
- I am working with the following directories at this time:
```{.bash}
$ tree rust/
rust/
├── code
│   └── projects
├── get-dependencies
│   ├── Cargo.lock
│   ├── Cargo.toml
│   └── src
│       └── main.rs
└── note
    └── 00.md

6 directories, 4 files
```

#### Hello, world!

- I made an additional `hello_world` within `projects`.
- I'm using `nvim` as my editor.
```{.bash}
$ nvim main.rs
$ cat main.rs
fn main() {
    println!("Hello, world!");
}
$ rustc main.rs ; ./main
Hello, world!
```

#### Limit testing

- I also tested the following:
```{.rust filename="main.rs"}
println!("Hello, world!");
```
- I got lengthy error messages that will probably be more interesting latter.

### 03

- I am advised to check I have a `cargo` version, which I've done already.
```{.bash}
cargo --version
```
- In `projects` directory I invoke:
```{.bash}
cargo new hello_cargo ; cd hello_cargo
```
- I see the following:
```{.bash}
$ tree
.
├── Cargo.toml
└── src
    └── main.rs

2 directories, 2 files
$ cat Cargo.toml 
[package]
name = "hello_cargo"
version = "0.1.0"
edition = "2024"

[dependencies]
```
- I am advised to immediately build, but I edit `main.rs` with different greeting.
- I run amicably but note that `tree` is already up to 18 (!!!) files.
    - I am opinionated about this!
- On the toy example, each of `build`, `run`, `check` run in about the same time and `build --release` is ~3x slower.

## ch02: Quick IO demo

- Just slammed a quick
```{.bash}
cargo new guessing_game
```
- We apparently need `std:io` for input but not printing, interesting.
- I made some minor edits then attempted a compile.
```{.rs filename="main.rs"}
use std::io;

fn main() {
    println!("Guess the number!\nPlease input your guess.");
    
    let mut guess = String::new();

    io::stdin().read_line(&mut guess).expect("Rip.");
    println!("You guessed: {}", guess);
}
```
- It appears:
    - Using `mut` is mandatory for function signatures.
    - `\n` is legal in println
    - If you dump enough text (I piped the book) into `main` it ties up your terminal for a good while.
    - I expected `from std use io` to work and it didn't.
    - I was unable to print (using `{}`) either `println!` or `String::new`.

### `/dev/unradom`

- As a security researcher, I was unwilling *not* to use `/dev/urandom`
    - This additionally enforces a Linux dependency and ameliorates a Rust package dependency.
- I found it *quite* involved to read from a file, and ultimately just "ctrl+f"'ed "dev/random" in this blog: [https://blog.orhun.dev/zero-deps-random-in-rust/](https://blog.orhun.dev/zero-deps-random-in-rust/)
- The debugging quite quite involved, but I think this is what I found.

#### Findings

- To read from a file, I seem to need both:
```{.rs}
use std::io::Read;
use std::io::File;
```
- I want to `?` terminate lines rather than using expects but seem to have to change the function signature of `main()` which I'd rather not do yet.
- To read a `char`, I instantiated a mutable `u8` buf of size 1, I think.
- Then I could `read_exact` which I'm guessing is safe? - but incurs an `expect`.
```{.rs}
let mut devrnd = File::open("/dev/urandom").expect("Oop!");
let mut secret = [0u8; 1];
devrnd.read_exact(&mut secret).expect("CHONK");
```
- I think I'm not supposed to know about buffers this earlier and I can't print them.
- I tried to type `secret` as `[u8]` and got told to borrow by `cargo`, which I don't know how to do.

#### Types

- It looks like `parse` is `atoi`, though may be more general.
- Array indexing appears as expected.
- It looks like most library functions return options, which forces a lot of error handling.
    - This is a feature for development and a bug for scripting.

### Closing Thoughts

- Loops are unconditional with `loop` and `break` is advised.
    - I'm opinionated.
- `continue` is also recommended.
- There's some namespace things going on I'll need to hear more about latter.
- My code:
```{.rs filename="main.rs"}
use std::cmp;
use std::fs::File;
use std::io;
use std::io::Read;

fn main() {
    println!("Guess the number!\nPlease input your guess.");

    /* This will brick Windows, which is a feature */
    let mut devrnd = File::open("/dev/urandom").expect("Oop!");
    let mut secret = [0u8, 1];
    devrnd.read_exact(&mut secret).expect("CHONK");
    println!("Secret is {} - sh!", secret[0]);

    loop {
        let mut guess = String::new();
        io::stdin().read_line(&mut guess).expect("Rip");
        let guess: u8 = match guess.trim().parse() {
            Ok(num) => num,
            Err(_) => continue,
        };
        println!("You guessed: {}", guess);

        match guess.cmp(&secret[0]) {
            cmp::Ordering::Less => println!("<"),
            cmp::Ordering::Greater => println!(">"),
            cmp::Ordering::Equal => {
                println!("girlypop!");
                break;
            }
        }
    }
}
```

- Pros:
    - `match`
    - `parse` looks cool, given there's types.
- Cons:
    - Real irregularities around printing, return types, and pointers.
    - Don't like the way they use loops.
- Okay:
    - File/IO is probably closer to being how awful it should be.

## ch03: Language Features

### 00

- Futureproofing keywords is cool.
- We recall that I hate this:
```{.py}
print("odd" if x % 2 else "even")
```
- And this:
```{.py}
f = lambda f->int x::int: x**x
```

### 01: Mutability

- Starting with mutability is extremely safety-first.
- `const`'s are compile-time constant.
    - I think this is typical? .js ruined me.
    - They also require type specification.
- Don't love shadowing personally, I think that should be against the law.
    - Especially the interaction with code blocks.
```{.rs}
let x = 1;
let x = x + 1; /* Not sure about this one */
```
- This drew warnings but was legal.
```{.rs}
let mut x = "5";
let mut x = 5;
let mut x = x + 6;
```

### 02: Types

#### Scalars

- `i32` default in 2025 is something else.
- I couldn't get wrapping on my guessing game, I'll need to find a way to do this:

> When you’re compiling in release mode with the --release flag, Rust does not include checks for integer overflow that cause panics.

- Has 128 bit types.
- `f64` default.
- This does not add up with `i32`

> The default type is `f64` because on modern CPUs, it’s roughly the same speed as `f32` but is capable of more precision. All floating-point types are signed.

- `%` works on floats.
- Can't combine floats and ints in the same operations.
- Chars probably explain the `i32` detail.

> Rust’s char type is four bytes in size and represents a Unicode Scalar Value

#### Collections

- Tuples allow mixed types, declared by type.
    - They're structs.
    - They both pattern match and use OO `.` with 0-indexed int literals.
    - `()` is legal and called `unit` - no comma!
- Arrays are arrays.
    - Rust has run-time buffer overflow blockers.
    - That prefix on the type specifies an initial value and dodge typing I think. 
```{.rs}
let mut secret = [0u8, 1];
```

### 03: Functions

- Parameters are post-typed Go-style
- Statements are side-effecting (`let`)
- Expressions return - unclear if they side-effect.
    - Expressions can be blocks that evaluate to the last statement like R (if not semicolon terminated.
    - Therefore, they may contain both statements and expressions.
- Functions return *without* a `return` keyword, I think, by omitting `;`, like blocks.
    - Return type is specifed with `->`...
    - So functions are named block expressions.

### 04: Comments

- Looks like C and C++ comments are legal.

### 05: Control Flow

#### Cond

- If is trivially `else if` with no `elif` on `{}` blocks.
    - They are legal in statements!
```{.rs}
let x = if true {1} else {0};
```
    - They must pass a type check.

#### Loop

- Named `loop`s are `goto` but worse.
    - `break` exits
    - `continue` restarts
    - Can return a value (????) via `break` (???????)
- This should work but doesn't:
```rs
fn main() {
    let mut i = 0;

    let r = loop {
        i += 1;

        if i == 10 {
            i * 2
        }
    };

    println!("{r}");
}
```
- With loops in hand I improved `guessing_game`
```rs
fn main() {

    /* This will brick Windows, which is a feature */
    let mut devrnd = std::fs::File::open("/dev/urandom").expect("Oop!");
    let mut buffer = [0u8, 1];
    std::io::Read::read_exact(&mut devrnd, &mut buffer).expect("CHONK");
    let secret = buffer[0];
    let mut guess = !secret;

    while guess != secret {

        /* Blows up if I declare this outside */
        let mut io = String::new();
        
        /* Continue less a print is Hmmge */
        std::io::stdin().read_line(&mut io).expect("Rip");
        guess = match io.trim().parse() {
            Ok(num) => num,
            Err(_) => continue,
        };

        if guess < secret {
            println!("<");
        } else {
            /* Cooler to print then girlypop than ifx2 */
            println!(">");
        }
    }
    println!("girlypop");
}
```
- I think I intend to avoid shadowing and `loop`.
    - Definitely want to be cautious about scoping+shadowing.
- Has `for each` but I don't really know about collections yet so we'll get to that latter.
- I believe this is Rust claim `loop` use un-Rustic and unsafe:
> The safety and conciseness of `for` loops make them the most commonly used loop construct in Rust. 
- I agree!
- There's ranges as well.

## ch04 Ownership

### 01: Ownership

- Ownership - the first Rust topic.
    - They avoid call heap pointers keys.
    - They argue from safety moreso than performance.
- Scoping is tied to ownership maybe.
    - Oh it is literally only scope.
    - De facto exit routines, I need to find the real vocab word.
- Single equals assignment performs a move, which is an atomic shallow copy + invalidation.
    - This means the compiler knows about variable names.
- This is suss.
```rs
let s = String::from("yolo")
let t = s;
println!("{s}"); // Rip
let x = 1;
let y = 1;
println!("{x}"); // Legal
```
- Seems like memory-safe stack allocation management is strictly harder, e.g. `4096_t` completion achieves the Rust LOs.

### 02 References

- References appear to be pointers-but-not-name.
    - These things appear to be fairly blaise objects under the hood.
    - Has to be some infrastructure around reference counting.
- Ownership is a metaphor and not a technology.
    - This is just performant-ish garbage collection.
- I can't figure out how Rust is ever faster than C.
- Ownership enforces access control, so actually we have objects with security policies.
- Ownership is fused pointer/mutex I guess.

### 03 Slices

- Slices are index ranges attached to an object that freeze the object.
- APIs allow mixed slice/object inputs via slice declaration, called "deref coercisions".

## ch05 Structs

### 01 Structs

- Fixed width named fields.
- You can declare with field values I guess.
```r
struct Pnt {
    x: u64,
    y: u64,
}

fn main() {
    let p = Pnt {
        x: 0,
        y: 1,
    };
    println!("Hello, world! ({0},{1})", p.x, p.y);
}
```
- Fields aren't ordered.
- Mutablility is struct level.
- $\exists$ "field init shorthard" which is... suspicious.
    - See also: Struct Update Syntax which is a destructive update/rename.
    - Ah, but only in the case that there's heap copies.
- $\exists$ tuple structs.
    - Names matter for typing and deconstructing.
    - Otherwise same as tuples.
```rs
struct Pnt(u64, u64);
```
- $\exists$ unit structs which basically have just a name.
    - That has to be great for runtime performance.
- There will be a future a future way to put things in structs owned by someone else via lifetimes.

### 02 Example

- You can autogenerate printers with the `#[derive(Debug)]` preprocessor (???) directive. and `:?` specifier in format blocks.
- `dbg!` macro seems great, I think I'll use that a lot.

### 03 Methods

- NOOOOOOOO!
- They're pipes?
- Wait is that good.
- They're good.
- Credit Jimmy and/or [this](https://rust-lang.github.io/rfcs/0445-extension-trait-conventions.html)
    - The `cargo` message here was pretty vague and documentation seems rough, but I like the language feature quite a bit.
```rs
trait PrintHam {
    fn printham(&self);
}

impl PrintHam for u64 {
    fn printham(&self) {
        let mut sum:u64 = 0;
        for i in 0..63 {
            sum += self >> i & 1;
        }
        println!("{sum}");
    }
}

fn main() {
    let n:u64 = 1025;
    n.printham();
}
```
- I am now method pilled (please language designers use pipes).
    - Ideally with type specification.
- I think I like this:

> The fact that Rust makes borrowing implicit for method receivers is a big part of making ownership ergonomic in practice.

- Looks like I could write `hamdist` but I don't see the benefit of doing that right now.
    - I need to write more things with return types at some point.
- Constructors are associated functions.
    - Gotta decide if those are Calvin.constructors, actual.constructors, or ~constructors.

## ch06 Enums

### 00

- Enums (woo) and **Pattern Matching** (BIG WOO !!!)
- Options (huge woo).
- I actually hated `match` in guessing game but bet it will be good here.

### 01 Enum

- I have literally never seen this written out long form before in my life.

> IP addresses: version four and version six.

- Oh I think an `enum` being just `4` might brick them.
    - It did.
```rs
#[derive(Debug)]
enum IP {
    v4,
    v6,
}

fn main() {
    let v = IP::v4;
    dbg!(v);
}
```
- This is not Rust related by representing IP addresses as strings should be illegal.
    - Yes, I do know everyone does it.
    - If I have some freetime I'm going PR a change here I think to `u128`.
- Ah, custom `struct`s inside an `enum` is Good, Actually.
- Oh enums are also unions, that actually makes sense.
- Options are an enum, that's good.

### 02 Match

- Matches `match` at least `enum`, hopefully more?
- Yes, uber slay.



