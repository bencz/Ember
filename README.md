# Ember Programming Language

**An educational, 100% object-oriented programming language with native compilation.**

Ember combines Python/Ruby-inspired syntax with static typing, type inference, and LLVM-powered native code generation. It compiles through the **Anvil** ( based on [ELENA](https://github.com/ELENA-LANG/elena-lang) VM e-code ) intermediate representation — a high-level, object-aware IR that bridges the semantic gap between OO languages and low-level machine code.

```ruby
# Hello World
IO.print("Hello, World!")
```

```ruby
# Recursive Fibonacci
def fib(n: Int) -> Int:
    if n <= 1:
        return n
    return fib(n - 1) + fib(n - 2)

for i in 0..20:
    IO.println("fib(${i}) = ${fib(i)}")
```

## Language Highlights

- **Python/Ruby hybrid syntax** — indentation-based, clean and readable
- **100% object-oriented** — everything is an object, including primitives
- **Static typing with inference** — explicit annotations optional where types can be inferred
- **Native compilation** — compiles to machine code via LLVM (not interpreted)
- **Anvil IR** — custom high-level intermediate representation with 80 opcodes
- **Async/Await** — write asynchronous code with `async def` and `await`, backed by a thread pool executor
- **Generics** — user-definable generic classes (`class Box<T>:`) with type-erased specialization
- **GC-managed memory** — custom generational GC (young/mature/permanent regions)
- **Resource management** — `using` blocks, `dispose()`, `finalize()` for deterministic cleanup
- **FFI** — call native C libraries directly via `NativeLibrary` + `@native` annotations
- **Namespaces** — organize code with `namespace` declarations
- **Serialization** — `props: [serializable: json]` for auto-generated `to_json()`/`from_json()` + runtime reflection
- **Rich stdlib** — String, Array, Hash, Int, Double, Float, Math, Set, List, Dict, Tuple, DateTime, TcpSocket, HttpServer, Reflect
- **Iterator protocol** — any class with `has_next()`/`next()` works in `for..in` loops

## Prerequisites

| Dependency | Version | Notes |
|-----------|---------|-------|
| CMake | 3.20+ | Build system |
| LLVM | 21 | Backend code generation |
| C++ compiler | C++17 | Clang or GCC |
| C compiler | C11 | For the runtime |

On macOS with Homebrew:
```bash
brew install cmake llvm pkg-config
```

## Build

```bash
mkdir -p build && cd build
cmake .. -DLLVM_DIR=/opt/homebrew/opt/llvm/lib/cmake/llvm
make -j8
```

The compiler binary is produced at `build/emberc`.

## Usage

```
emberc [options] <source.em>

Options:
  -o <file>       Output executable name (default: a.out)
  -O0             No optimization (debug)
  -O1             Basic optimization
  -O2             Full optimization (default)
  --emit-tokens   Dump lexer tokens and exit
  --emit-ast      Dump parsed AST and exit
  --emit-ecodes   Dump Anvil IR (ecodes) and exit
  --emit-llvm     Dump LLVM IR and exit
  --target <plat> Target platform: mac, linux (default: auto-detect)
  --verbose       Verbose compilation output
  --help, -h      Show help

Linker flags:
  -l<lib>         Link with library (e.g. -lSDL3)
  -L<dir>         Add library search path
  --rpath <dir>   Add runtime library search path
```

Linker flags are merged with any flags auto-derived from `NativeLibrary.load()` calls.

Example:
```bash
./build/emberc test_samples/01_hello_world.em -o hello
./hello
# Hello, World!
```

## Language Features

### Variables and Types

```ruby
let x: Int = 42        # Explicit type
let y = 3.14           # Inferred as Double (64-bit)
let z = 1.5f           # Inferred as Float (32-bit)
let name = "Ember"     # Inferred as String
let flag = true        # Inferred as Bool
```

### Functions

```ruby
def greet(name: String) -> String:
    return "Hello, ${name}!"

IO.println(greet("World"))
```

### Control Flow

```ruby
if x > 10:
    IO.println("big")
elif x > 5:
    IO.println("medium")
else:
    IO.println("small")

while i < 10:
    IO.println(i)
    i = i + 1

for item in [1, 2, 3]:
    IO.println(item)

for i in 0..10:
    IO.println(i)
```

### String Interpolation

```ruby
let name = "World"
IO.println("Hello, ${name}!")
IO.println("2 + 3 = ${2 + 3}")
```

### Generics

```ruby
class Box<T>:
    def initialize(@value):
        let x = 0
    def get():
        return @value

let int_box = Box.new(42)
let str_box = Box.new("hello")
IO.println(int_box.get())     # 42
IO.println(str_box.get())     # hello
```

### Collections

```ruby
let numbers = [1, 2, 3, 4, 5]
IO.println(numbers[0])
IO.println(numbers.length())
IO.println(numbers.sort())       # [1, 2, 3, 4, 5]
IO.println(numbers.reverse())    # [5, 4, 3, 2, 1]
IO.println(numbers.sum())        # 15

# Typed collections
let lst = List.new()
lst.push(10)
lst.push(20)

let s = Set.new()
s.add("apple")
s.add("banana")

let pair = Tuple.new("key", 42)
IO.println(pair.first())   # key
```

### Classes and Inheritance

```ruby
class Animal:
    def initialize(name: String):
        let @name = name

    def speak() -> String:
        return "..."

class Dog(Animal):
    def speak() -> String:
        return "Woof!"

let rex = Dog.new("Rex")
IO.println(rex.speak())
```

### Closures

```ruby
let square = |x| x * x
IO.println(square.call(5))

items.each(do |item|:
    IO.println(item)
)
```

### Exception Handling

```ruby
try:
    let result = divide(10, 0)
catch e: DivisionByZeroError:
    IO.println("Cannot divide by zero!")
finally:
    IO.println("Done")
```

### Concurrency

```ruby
let ch = Channel.new(0)

Thread.new(do:
    ch.send(42)
)

let value = ch.receive()
IO.println("Got: ${value}")
```

### Async/Await

```ruby
async def compute(x):
    return x * 2

let f = compute(21)
IO.println("Result: ${f.value()}")    # Result: 42

# Await inside async functions
async def pipeline(x):
    let doubled = await compute(x)
    return doubled + 10

IO.println(pipeline(5).value())       # 20

# Concurrent futures
let f1 = compute(10)
let f2 = compute(20)
IO.println(Future.all([f1, f2]))      # [20, 40]
```

### Resource Management

```ruby
class Connection(Object):
    def initialize(url: String):
        let @url = url
    def dispose():
        IO.println("connection closed")

using conn = Connection.new("db://localhost"):
    IO.println("using connection")
# dispose() called automatically
```

### Namespaces

```ruby
namespace Math:
    def square(x: Int) -> Int:
        return x * x

IO.println(Math.square(5))
```

### FFI (Foreign Function Interface)

```ruby
class LibM(NativeLibrary):
    load(mac: "libm.dylib", linux: "libm.so.6")
    @native def sqrt(x: Double) -> Double
    @native def pow(base: Double, exp: Double) -> Double
```

Cross-compile with `--target`:
```bash
emberc --target linux source.em -o output     # Cross-compile for Linux
emberc source.em -o output                     # Auto-detect host platform
```

### Serialization

```ruby
class Point(props: [serializable: json]):
    let x: Double
    let y: Double
    def initialize(@x: Double, @y: Double):
        let _ = 0

let p = Point.new(3.14, 2.72)
IO.println(p.to_json())                         # {"x":3.14,"y":2.72}

let q = Point.from_json("{\"x\":1.0,\"y\":2.5}")
IO.println(q.x)                                  # 1.0

# Runtime reflection
IO.println(Reflect.fields("Point"))               # [x, y]
IO.println(Reflect.get(p, "x"))                   # 3.14
```

Use `@json(name: "key")` to rename JSON keys:
```ruby
class User(props: [serializable: json]):
    @json(name: "user_name")
    let username: String
    let age: Int
```

### Type System

Built-in types: `Int`, `Float` (32-bit), `Double` (64-bit), `Bool`, `String`, `Nil`, `Array<T>`, `Hash<K, V>`, `Range`, `Block`, `Thread`, `Channel<T>`, `Future`, `IntPtr`

Stdlib generic types: `List<T>`, `Dict<K,V>`, `Set<T>`, `Tuple<A,B>`

FFI-only types: `Uint8` (i8), `Uint32` (i32)

## Current Status

| # | Test | Feature | Status |
|---|------|---------|--------|
| 01 | `hello_world` | Print, basic pipeline | **Pass** |
| 02 | `variables_and_types` | Variables, types, arithmetic | **Pass** |
| 03 | `classes_basic` | Class declaration, methods | **Pass** |
| 04 | `inheritance` | Class hierarchy, polymorphism | **Pass** |
| 05 | `control_flow` | if/elif/else, while, for-in | **Pass** |
| 06 | `closures_blocks` | Lambdas, blocks, capture | **Pass** |
| 07 | `collections` | Arrays, hashes | **Pass** |
| 08 | `exceptions` | try/catch/finally, throw | **Pass** |
| 09 | `operators` | Operator overloading | **Pass** |
| 10 | `type_inference` | Implicit type inference | **Pass** |
| 11 | `string_interpolation` | `"${expr}"` templates | **Pass** |
| 12 | `iterators` | Range/array iteration | **Pass** |
| 13 | `concurrency` | Threads, channels | **Pass** |
| 14 | `pattern_matching` | match/when statements | **Pass** |
| 15 | `fibonacci` | Recursion, loops | **Pass** |
| 16 | `namespaces` | Namespace declarations, qualified access | **Pass** |
| 17 | `ffi` | NativeLibrary, @native, C interop | **Pass** |
| 18 | `intptr` | IntPtr raw pointer operations | **Pass** |
| 19 | `structs` | Struct layout (props: [layout: struct]) | **Pass** |
| 20 | `datetime` | DateTime via C FFI + struct layout + Marshal | **Pass** |
| 21 | `packed_unions` | Packed/union layouts (props: [layout: packed/union]) | **Pass** |
| 22 | `finalize` | finalize, dispose, using blocks | **Pass** |
| 23 | `gc_pressure` | GC stress: string interp, objects, nested refs | **Pass** |
| 24 | `generics` | Generic classes (Box\<T\>), List, Set, Tuple | **Pass** |
| 25 | `stdlib_string` | String stdlib extensions (trim, replace, split...) | **Pass** |
| 26 | `stdlib_collections` | Array/Hash extended methods (sort, reduce, merge...) | **Pass** |
| 27 | `stdlib_math` | Math module, Int/Double numeric extensions | **Pass** |
| 28 | `stdlib_iterator` | Iterator protocol: custom classes in for..in | **Pass** |
| 29 | `stdlib_pure_methods` | Pure Ember stdlib methods, inline @\_\_builtin expression | **Pass** |
| 30 | `generators` | Generators (yield keyword, state machine desugaring) | **Pass** |
| 31 | `async_await` | Async functions, await, Future, error propagation | **Pass** |
| 32 | `sockets` | TCP server/client, echo test, IntPtr buffers | **Pass** |
| 33 | `http` | HTTP server, request parsing, routing, response | **Pass** |
| 34 | `serialization` | JSON serialization, `@json` rename, `Reflect` class | **Pass** |
| 35 | `async_closures` | Await inside block literals, async closures | **Pass** |

Run the test suite:
```bash
./test_samples/run_all.sh
```

## Documentation

- [Language Tutorial](docs/ember_tutorial.md) — Full language guide with examples
- [Grammar Specification](docs/ember_grammar.md) — EBNF grammar
- [Architecture Guide](ARCHITECTURE.md) — Compiler internals and pipeline
- [Anvil IR Specification](anvil_spec.md) — VM and IR design
- [Ecode Reference](anvil_ecode_ref.md) — Complete opcode listing

