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

##### Aside

- I have worked `continue` and `break` out of guessing game.
```rs
fn main() -> std::io::Result<()> {
    /* This will brick Windows, which is a feature */
    let mut devrnd = std::fs::File::open("/dev/urandom").expect("Oop!");
    let mut buffer = [0u8, 1];
    std::io::Read::read_exact(&mut devrnd, &mut buffer).expect("CHONK");
    let secret = buffer[0];
    let mut win = false;

    while !win {
        /* Blows up if I declare this outside */
        let mut io = String::new();

        /* Continue less a print is Hmmge */
        std::io::stdin().read_line(&mut io)?;
        win = match io.trim().parse::<u8>() {
            Ok(n) => {
                if n < secret {
                    println!("<");
                    n == secret
                } else {
                    println!(">");
                    n == secret 
                }
            }
            Err(_) => false,
        }
    }
    println!("girlypop");
    Ok(())
}
```
- I will work to remove the options next.

### 02 Match

- Matches `match` at least `enum`, hopefully more?
- Yes, uber slay.
- I don't seem to be able to match on expressions, so this is just switch.

### 03 If Let

- But does it have `let` `in`.
- Basically, `if let` is close enough.

## ch07 Package Management

- Rust falsely believes I will need libraries and multiple files to implement an OS.
- Vocab test:
    - Packages are a cargo way to distribute crates.
    - Crates trees of modules that can be linked.
    - Modules set paths, which I think are namespace things.
    - Paths are ways of naming items (so yes).

### 01 Crates

- Are the atomic unit.
- Libraries lack main, and are the entry level crate categorization. 
- Packages are are a DAG of $n\geq1$ crates.
    - Maximum one library crate.
- `cargo` new was (sighs) a package (cargo) making a package.

### 02 Modules

- Root looks like `lib.rs`/`main.rs` is a social convention but may be the compiler default.
- Root should enumerates modules via `mod <name>` which map to `<name>.rs` or `<name>/mod.rs` - probably for multifile modules?
- `mod` in nonroot is submodule by construction, and follows the expected pathing.
- The `outer::middle::inner` syntax is a file path.
- Default to private with `pub` to publish, which can be applied `mod` level.
- `use` introduces a symbolic link.

### 03 Paths

- Can use absolute introduced by `crate` or relatives introduced by `self` and `super`.
- I thought C.header and Java.interface had been well regarded and am surprised to see a fused API/implementation, which seems to be the direction here.
    - I'm wondering if this is due to factoring occuring in the compiler backend.
- There's a link to API Guidelines I should read someday.
    - Especially if I make `vcd2pl`
- This chapter appears unusually preachy about completely Rust agnostic topics, which seems odd for the stated audience (knows at least one language, by which they mean C++).
    - Okay having written that, there was probably some C++ beef this is directly responding to.
- On loopback, it looks like `lib.rs` is the intended header via `mod` statements.

#### Structs

- Public `struct`s have private fields by default.
- Public `enums` have public variants by default.

### 04 Use

- `use` keyword does exactly what you'd expect.
- Oh there is `use` `as` I had kinda assumed there wasn't.
- Can use `pub use` to diverge the API and file structure.
- `std` is available without config but no other packages are, otherwise manipulate `.toml`
- Oh there's nested `use`, I really wanted that`
```rs
use std::{cmp::Ordering, io};
use std::io::{self, Write};
```
- The capitalization regime is getting omega suss.
- There's Glob!
```rs
use std::collections::*;
```
- I will not use this for evil.

### 05 Separation

- Ah there it is.

> In other words, `mod` is not an “include” operation that you may have seen in other programming languages.

- I question to appropriateness of burning the 7th chapter over some petty grievance.
- My take is that if Rust did in fact solve package management, everyone would know that and it wouldn't be the subject of lengthy prose.
    - And they definitely didn't solve package management *per examples used in this section*.
- The chapter correctly notes:

> The main downside to the style that uses files named mod.rs is that your project can end up with many files named mod.rs, which can get confusing when you have them open in your editor at the same time.

- Even just skimming this guide I have 9 `src/main.rs` files in my notes.
    - Either something is a good practice or it isn't.

#### Closing thoughts

- Overall a tedious and embarrassing chapter.
- I was annoyed with `cargo` when I *had* been giving it the benefit of the doubt.
- Fusing compilation to a filesystem will complicate self-hosting.
    - I do wonder if `rustc` can compile an io stream.

## ch08 Collections

- These are the heap collections (e.g. not tuple/array).
    - vector
    - string
    - hash map
- I wonder how they dodge the Java dictionary problem.

### 01

- Very refreshing to see an explicit vector!
    - I have been fighting for my LIFE defending the naming of `argv`.
- They're typed.
- $\exists$ `vec!`
- Use "update" terminology to describe insert/append/extend/push.
    - `v.push(e)`
    - I'd reserve "update" for in place, wonder if that aged out.
- Indices return slices and `.get` returns slice options.
    - I'm inclined to like `get`.
- Updates nuke slices liberally for safety (seems fine).
- Supports `for` `in`
- Mathematics is unsurprisingly not vectorized as far as I can tell.
```rs
let mut v = vec![100, 32, 57];
for i in &mut v {
    *i += 50;
}
```
- vs.
```py
l = [100, 32, 57]
for i in range(l):
    l[i] += 50
```
- vs.
```C
int32_t buf[3] = {100, 32, 57}, i = 3;
while (i--) {
    buf[i] +- 50
}
```
- Seems fine. Ref (` & `) and deref (` * `) felt bad, but the alternatives are worse.
```py
a = np.array([100,32,57]) + 50
```
- Using deference with a 7 chapter foreward reference in chapter 8 is really something though!
- Polymorphic vectors via `enum`, sure why not.
    - Sure beats `void * ` but I think we all know `void * ` is baseline for how bad something can get.
- Vector drops drop elements.

### 02 Strings

- "Oh boy!" - Nobody.
- Taxonomy:
    - `str` (string slice) is only core language string.
    - Literals are ELF cached slices.
        - I say: slices *of what*.
    - `String` is a `std` collection type is:
        - Growable (????)
        - Mutable
        - Owned
- `x.to_string()` and `String::from(x)` are redundant... by design?
- `+` is overloaded for string. Great!
    - It's not great.
- $\exists$ `format!` macro (sprintf?)
- `.push_str` is, of course, an append/concat not a push.
    - Love!
    - It's also a deep copy!
- `.push` is push.
- `+` also takes (1) irregular types and (2) irregularly impacts ownership.
    - Implemented as a method of course.
    - All the downsides of OO with none of the benefits!
```rs
let mut s1 = "hello ".to_string();
s1 += "world";
println!("{s1}");
```
- String indexing is banned.
    - Okay.
- The thing is that this is what I want:

> if &"hi"[0] were valid code that returned the byte value, it would return 104, not h.

- Also `'h' == 104`.
- Wait they bricked string index for *performance*.
    - Are the performance people and unicode people the same people?
- Numerical slices are permitted in code and permitted to crash at runtime.
- Rust recommends requesting `.chars()` or `.bytes` in iteration.
- I think I just need to find (or write) an ASCII API.

### 03 Hash Maps

- "Get in losers; we're doing key-value storage." - Ada Lovelace
- Curious how Big Rust avoids Ye Olde Java Dictionary Probleme
    - Probably by implementing correctly the first time (lol).
- Oh these will be ludicrously obnoxious to use with String keys huh.
- They avoided Javaproblem by just not implementing it all the way haha.
- All values have to be of the same type, but `enums` exist so that's an imaginary rule.
- Wow I wonder why people don't use these.
```rs
/* When your language doesn't need pipes */
let score = scores.get(&team_name).copied().unwrap_or(0);
```
- I need to look at the Polars source.
- For is over pairs:
```rs
for (k,v) in &d {
    println!("{k}:{v}");
}
```
- They seem to be claiming their HashMap achieves security goals in production via a citation to Wikipedia.
    - There's a paper they just *don't* link. [SipHash](https://web.archive.org/web/20170327151630/https://131002.net/siphash/siphash.pdf)
- It seems like precious little key-value storage is being done after all, though the claims in this chapter suggest it may be "for real". 
    - Need to learn more.

## ch09

- Error handling, which presumably is what makes (alongside memsafety) Rust OS-viable.
- Not having exceptions is, probably, the most correct thing Rust has done, and I think probably a bigger contribution than borrow.

### 01 Panic

- Some errors cause automatically, or you can invoke with `panic!`
- They print by default (whoa!) and cleanup?
    - Oh they allow hard abort via `.toml`?
    - Odd place to put that I think,
> Here, we’re attempting to access the 100th element of our vector (which is at index 99 because indexing starts at zero),
- Being cool enough to 0-index and not cool enough to base-2 magic number is rough.
- Sounds like `cargo run` does extremely heavy debug instrumentation by default, and I can probably turn that off for now.
    - Probably one of the drivers of compile times.

### 02 Results

- These are nullables built using the option system.
- This is going to be huge after years of e.g. `man fread`
```C
fp = fopen("/bin/sh", "rb");
if (!fp) {
    perror("fopen");
    return EXIT_FAILURE;
}
```
- Looks like its mostly used in handle/panic trees.
- `unwrap_or_else` looks cool.
    - I kinda hate `match` in a dangling curly brace language actual.
    - I thought I was getting this.

```ml
fun factorial 0 = 1
  | factorial n = n * factorial (n - 1)
```
- Oh I think I'm basically always using `unwrap` except in an OS.
    - So Rust is good because it enforces `if (!fp)` via the type system.
- Rust is recommending `except` vs `unwrap` but I literally never want an error message giving me anything other than a code line number (and file name!)
    - I don't even want that second line probably.
- Just kidding `?` looks cooler than `.unwrap`.
- This chapter is very obviously why the language is good I'm not sure why I heard so much nonsense about borrowing which actually appear completely mundane.
- You can slam `?` in the middle of a not-pipe let's go.
- `main` is in fact (somehow) restricted, but the manner in which it is restricted is a carefully guarded secret that is not disclosed in documentation (for some reason).

> So far, all the `main` functions we’ve used return `()`. The `main` function is special because it’s the entry point and exit point of an executable program, and there are restrictions on what its return type can be for the program to behave as expected.

- This is the "good" `main`:
```rs
fn main() -> Result<(), Box<dyn Error>> {
    Ok(())
}
```

- The secret legal main return types are terminators (okay that's a tautology but at least a programmable one).

### 03 Panic?

- Panic in prototypes and in unreachable states.
- Easy.
- Also security cases.
- I guess what makes this a Rust chapter is Rust not having exceptions?
    - There a no non-C design decisions discussed in 03.
- wait WHAT
> Recall the guessing game in Chapter 2 in which our code asked the user to guess a number between 1 and 100
- I do *not* recall that and I *do* recall altering the code to a `u8` to *impose* a 100 limit, so why was a `u8` not recommended.
    - Separately, I would've used an `i8` so I could test diffs vs. `0`.
- Okay now they're using `i32`s still what is going on.
    - If bits cost money you must `i8`.
    - If they don't you must `x64`
- The Base 10 privileging is getting weird.
- This is such a good language feature to use so poorly.
    - I don't understand factoring bounds checking into a struct and a not a func.
    - Funcs return results, structs don't.
    - OO poisoning, I think.

## ch10 Generic/Trait/Lifetime

- It has been too long since last I faced a doom-driven hero of the dragon blood.

### 01 Generic

- Okay we're implementing polymorphic vecmax.
- I wonder what the best language framework is for automated factoring, actually.
- Okay so instead of a writing a function on a type you can write a function on a trait.
    - Traits are... sets of types?
    - Odd to describe then as attributes of types (to me).
    - OO poisoning?
- Can have generic structs, cool.
    - Wondering if you can implement C-math somehow.
    - Not that you should!
- Works on methods, which means I need to see if $\exists$ `sizeof`.

### 02 Traits

- Here's a claim:

> Note: Traits are similar to a feature often called interfaces in other languages, although with some differences.

- Oh I learned these already to method a builtin.
- P2P Social Media in Rust would go crazy.
    - Short post
    - Long post
    - Img post
    - Slideshow
    - Video post
    - Audio post
    - Emoji post
    - Enum posts of some type
    - Good user
    - Chrome user
    - iOS user
    - Cartesian product
- Oh this is also non-evil but limited inheritance, neat.
- Okay so I have no idea why `quarto` isn't written in Rust now, and I didn't before, but now I *really* don't know.
    - It may be in Haskell?
        - Oh it was in typescript LOL
    - `mdbook` needs `reveal`.
- `where` is good
```rs
fn some_function<T, U>(t: &T, u: &U) -> i32
where
    T: Display + Clone,
    U: Clone + Debug,
{
```
- That is very standard ml of new jersey
- Oh and this means the return type of `main` is `impl terminate` (sp?)
- Blankets are good, this is my least favorite thing:
```py
class Box:
    
    def __init__(self, x):
        self.x = x

    def __str__(self):
        return "I'm a box"

    def __repr__(self):
        return self.__str__()
```

### 03 Lifetimes

- Rust falsely claims this will be a concept I'm not familiar with.
    - And then motivates with dangling pointer!
    - Pass a pointer into your stack frame! This is trivial!
- I think I need to see how close MISRA C and MISRA C++ are to Rust *once compiled*.
    - Oh apparently there's no actually existing MISRA C lol.
