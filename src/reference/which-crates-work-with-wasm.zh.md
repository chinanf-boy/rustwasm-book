# Which Crates Will Work Off-the-Shelf with WebAssembly?

列出所做的事情是最容易的*不*目前使用 WebAssembly;避免这些东西的板条箱通常可以移植到 WebAssembly 上*干得好*.一个好的经验法则是,如果一个板条箱支持嵌入式和`#![no_std]`用法,它可能也支持 WebAssembly.

## Things a Crate Might do that Won't Work with WebAssembly

### C and System Library Dependencies

wasm 中没有系统库,因此尝试绑定到系统库的任何包都不起作用.

使用 C 库也可能无法工作,因为 wasm 没有用于跨语言通信的稳定 ABI,并且 ism 的跨语言链接非常挑剔.每个人都希望这最终能够发挥作用,`clang`正在运送他们的`wasm32`现在默认为目标,但故事尚未完全存在.

### File I/O

WebAssembly 无权访问文件系统,因此假设存在文件系统-并且没有特定于解决方案的解决方法-不管用.

### Spawning Threads

有[计划向 WebAssembly 添加线程][wasm-threading],但尚未发货.试图在一个线程上产生`wasm32-unknown-unknown`目标会引起恐慌,这会触发一个陷阱.

[wasm-threading]: https://rustwasm.github.io/2018/10/24/multithreading-rust-and-wasm.html

## So Which General Purpose Crates Tend to Work Off-the-Shelf with WebAssembly?

### Algorithms and Data Structures

提供特定实施的板条箱[algorithm](https://crates.io/categories/algorithms)要么[data
structure](https://crates.io/categories/data-structures)例如,A \*图搜索或 splay 树,往往适用于 WebAssembly.

### `#![no_std]`

[Crates that do not rely on the standard
library](https://crates.io/categories/no-std)倾向于与 WebAssembly 一起使用.

### Parsers

[Parsers](https://crates.io/categories/parser-implementations) -只要他们只是接受输入并且不执行他们自己的 I / O.-倾向于与 WebAssembly 一起使用.

### Text Processing

[Crates that deal with the complexities of human language when expressed in
textual form](https://crates.io/categories/text-processing)倾向于与 WebAssembly 一起使用.

### Rust Patterns

[Shared solutions for particular situations specific to programming in
Rust](https://crates.io/categories/rust-patterns)倾向于与 WebAssembly 一起使用.
