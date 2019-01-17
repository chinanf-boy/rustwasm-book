# 您应该知道的箱子

这是一个精选的箱子列表,关于 Rust 和 WebAssembly 开发.

[您还可以浏览 WebAssembly 类别中,发布到 crates.io 的所有包. ][wasm-category]

## 与 JavaScript 和 DOM 交互

### `wasm-bindgen`\|[crates.io](https://crates.io/crates/wasm-bindgen)\|[github](https://github.com/rustwasm/wasm-bindgen)

`wasm-bindgen`促进 Rust 和 JavaScript 之间的高级交互. 它允许人们将 JavaScript 内容导入 Rust 和 Rust 内容导出到 JavaScript.

### `js-sys`\|[crates.io](https://crates.io/crates/js-sys)\|[github](https://github.com/rustwasm/wasm-bindgen/tree/master/crates/js-sys)

原生`wasm-bindgen`导入所有 JavaScript 全局类型和方法,例如`Object`,`Function`,`eval`等. 这些 API 可以在所有标准 ECMAScript 环境 中移植,而不仅仅是 Web,例如 Node.js.

## 错误报告

### `console_error_panic_hook`\|[crates.io](https://crates.io/crates/console_error_panic_hook)\|[github](https://github.com/rustwasm/console_error_panic_hook)

这个箱子让你调试`wasm32-unknown-unknown`的 panics,通过提供一个恐慌钩子, 来将恐慌消息转发到`console.error`.

## 动态分配

### `wee_alloc`\|[crates.io](https://crates.io/crates/wee_alloc)\|[github](https://github.com/rustwasm/wee_alloc)

该 **W**asm-**E**nabled, **E**lfin 分配器. 一个小的 (~1K 未压缩`.wasm`)分配器实现,特点是代码大小比分配性能更受关注,.

## 解析和生成`.wasm`二进制

### `parity-wasm`\|[crates.io](https://crates.io/crates/parity-wasm)\|[github](https://github.com/paritytech/parity-wasm)

用于序列化,反序列化和构建的低级 WebAssembly 格式库 - `.wasm`二进制文件. 对已知的自定义部分具有良好支持,例如"names"部分和"reloc.WHATEVER"部分.

### `wasmparser`\|[crates.io](https://crates.io/crates/wasmparser)\|[github](https://github.com/yurydelendik/wasmparser.rs)

一个简单的事件驱动库,用于解析 WebAssembly 二进制文件. 例如,提供每个解析事物的字节偏移量,这在解释 reloc 时是必需的.

## 解释和编译 WebAssembly

### `wasmi`\|[crates.io](https://crates.io/crates/wasmi)\|[github](https://github.com/paritytech/wasmi)

来自 Parity 的可嵌入 WebAssembly 解释器.

### `cranelift-wasm`\|[crates.io](https://crates.io/crates/cranelift-wasm)\|[github](https://github.com/CraneStation/cranelift)

将 WebAssembly 编译为本机主机的机器代码. Cranelift (néCretonne) 代码生成器项目的一部分.

[wasm-category]: https://crates.io/categories/wasm
