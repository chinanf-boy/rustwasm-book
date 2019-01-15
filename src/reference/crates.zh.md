
# 您应该知道的箱子

这是一个精选的箱子列表,关于Rust和WebAssembly开发. 

[您还可以浏览WebAssembly类别中,发布到crates.io的所有包. ][wasm-category]

## 与JavaScript和DOM交互

### `wasm-bindgen`\|[crates.io](https://crates.io/crates/wasm-bindgen)\|[github](https://github.com/rustwasm/wasm-bindgen)

`wasm-bindgen`促进Rust和JavaScript之间的高级交互. 它允许人们将JavaScript内容导入Rust和Rust内容导出到JavaScript. 

### `js-sys`\|[crates.io](https://crates.io/crates/js-sys)\|[github](https://github.com/rustwasm/wasm-bindgen/tree/master/crates/js-sys)

原生`wasm-bindgen`导入所有JavaScript全局类型和方法,例如`Object`,`Function`,`eval`等. 这些API可以在所有标准 ECMAScript环境 中移植,而不仅仅是Web,例如Node.js.

## 错误报告

### `console_error_panic_hook`\|[crates.io](https://crates.io/crates/console_error_panic_hook)\|[github](https://github.com/rustwasm/console_error_panic_hook)

这个箱子让你调试`wasm32-unknown-unknown`的panics,通过提供一个恐慌钩子, 来将恐慌消息转发到`console.error`. 

## 动态分配

### `wee_alloc`\|[crates.io](https://crates.io/crates/wee_alloc)\|[github](https://github.com/rustwasm/wee_alloc)

该 **W**asm-**E**nabled, **E**lfin分配器. 一个小的 (~1K未压缩`.wasm`)分配器实现,特点是代码大小比分配性能更受关注,. 

## 解析和生成`.wasm`二进制

### `parity-wasm`\|[crates.io](https://crates.io/crates/parity-wasm)\|[github](https://github.com/paritytech/parity-wasm)

用于序列化,反序列化和构建的低级WebAssembly格式库 - `.wasm`二进制文件. 对已知的自定义部分具有良好支持,例如"names"部分和"reloc.WHATEVER"部分. 

### `wasmparser`\|[crates.io](https://crates.io/crates/wasmparser)\|[github](https://github.com/yurydelendik/wasmparser.rs)

一个简单的事件驱动库,用于解析WebAssembly二进制文件. 例如,提供每个解析事物的字节偏移量,这在解释reloc时是必需的. 

## 解释和编译WebAssembly

### `wasmi`\|[crates.io](https://crates.io/crates/wasmi)\|[github](https://github.com/paritytech/wasmi)

来自Parity的可嵌入WebAssembly解释器. 

### `cranelift-wasm`\|[crates.io](https://crates.io/crates/cranelift-wasm)\|[github](https://github.com/CraneStation/cranelift)

将WebAssembly编译为本机主机的机器代码. Cranelift (néCretonne) 代码生成器项目的一部分. 

[wasm-category]: https://crates.io/categories/wasm
