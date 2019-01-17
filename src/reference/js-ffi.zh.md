# JavaScript 的互操作

### 导入和导出 JS 函数

#### 从 Rust 方面来看

在 JS 环境 中使用 wasm 时，从 Rust 导入和导出函数很简单: 它的工作方式与 C 完全相同

WebAssembly 模块定义了导入的一个系列，每个导入带有一个 _模块名_ 和 一个*导入名称*。 但我们可以使用一个`extern { ... }`区块和 [`#[link(wasm_import_module)]`][wasm_import_module]来声明模块名， 目前
它默认为"env"。

导出只需要一个名称。 除了其他`extern`函数之外，WebAssembly 实例的线性内存默认导出为"memory"

[wasm_import_module]: https://github.com/rust-lang/rust/issues/52090

```rust
// 从 `mod`模块导入一个函数 `foo`
#[link(wasm_import_module = "mod")]
extern { fn foo(); }

// 导出 一个 Rust 函数 `bar`
#[no_mangle]
pub extern fn bar() { /* ... */ }
```

由于 wasm 的有限值类型，这些函数必须仅在 原始数字类型上运行。

#### 从 JS 方面来看

在 JS 中，wasm 二进制文件变成了 ES6 模块。

其中，线性内存的*实例化*和一组 JS 函数，一定是预期导入吻合的。 有关实例化的详细信息，请访问[MDN][instantiation]。

[instantiation]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/WebAssembly/instantiate

生成的 ES6 模块将包含从 Rust 导出的所有函数，现在可用作 JS 函数。

[这里][hello world]是整个设置的一个非常简单的例子。

[hello world]: https://www.hellorust.com/demos/add/index.html

### 超越数字

在 JS 中使用`wasm`时，`wasm`模块的内存与 JS 内存之间存在明显的分歧:

- 每个`wasm`模块都有一个线性内存 (在本文档的顶部描述) ，它在实例化期间初始化。 **JS 代码可以自由地读写这个内存**。

- 相比之下，`wasm`代码没有*直接*访问 JS 对象。

因此，复杂的互操作以两种主要方式发生:

- 将二进制数据复制或输出到`wasm`内存。 例如，这是一种`String`提供所有权的到 Rust 的方式。

- 设置 JS 对象的显式"堆"，然后给出"地址"。 这允许`wasm`代码间接引用 JS 对象 (使用整数) ，并通过 调用导入的 JS 函数 对 这些对象 进行操作。

幸运的是，这个互操作故事非常适合通过通用的"bindgen"式框架进行处理: [wasm-bindgen]。 该框架可以自动编写 惯用 Rust 函数签名 映射 惯用 JS 函数 的

[wasm-bindgen]: https://github.com/alexcrichton/wasm-bindgen
