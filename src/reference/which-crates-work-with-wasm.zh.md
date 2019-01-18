# 有哪些 Crates 现在就能为 WebAssembly 工作?

最容易列出的事情，正是目前*不*使用 WebAssembly 的箱; 避免这些东西的箱，是可以移植到 WebAssembly 的 ，但通常*只能到及格线*。一个好的经验法则是，如果一个箱支持嵌入式和`#![no_std]`用法，它极可能也支持 WebAssembly。

## 一个 Crate 最可能会的事情，就是不能与 WebAssembly 一起工作

### C 和 系统 库 依赖项

wasm 中没有系统库，因此 任何 crate 尝试到系统库的绑定，都不起作用。

使用 C 库也可能无法工作，因为 wasm 没有用于跨语言通信的稳定 ABI，并且 wasm 的跨语言链接非常挑剔。每个人都希望这些地基，最终能够建成，尤其是`clang`，也正在运送他们的`wasm32`目标，现在开始为默认设置，但'故事'还没有完成罢了。

### File I/O

WebAssembly 无权访问文件系统，因此假设存在文件系统 &mdash; 并且没有 wasm 特定的解决方法 &mdash; 不管用。

### 派生线程

有[计划向 WebAssembly 添加线程][wasm-threading]，但尚未发货。若试图在`wasm32-unknown-unknown`目标的一个线程上派生会引起恐慌，这会触发一个 wasm 陷阱。

[wasm-threading]: https://rustwasm.github.io/2018/10/24/multithreading-rust-and-wasm.html

## 所以，要 Crates 现在就能为 WebAssembly 工作，有那些常用方式?

### 算法和数据结构

提供一个特定[算法](https://crates.io/categories/algorithms)实现或[数据 结构](https://crates.io/categories/data-structures)的 crates 例如, A\* 图搜索或 splay 树，往往也适用于 WebAssembly。

### `#![no_std]`

[不依靠标准库的 Crates ](https://crates.io/categories/no-std)，往往也适用于 WebAssembly。

### Parsers(解析器)

[Parsers](https://crates.io/categories/parser-implementations) -只要他们只是接受输入，并且不执行他们自己的 I/O。往往也适用于 WebAssembly。

### Text Processing(文本处理中)

[当 要文本形态的明确性，处理人类语言的复杂方面的 Crates ](https://crates.io/categories/text-processing)，往往也适用于 WebAssembly。

### Rust Patterns(Rust 模式)

[分享 在 Rust 编程环境下，特定情况的解决方案](https://crates.io/categories/rust-patterns)，往往也适用于 WebAssembly。
