# 调试 Rust-生成的 WebAssembly

本节包含，调试 Rust 生成的 WebAssembly 的提示.

## 带调试符号的构建

> ⚡ 调试时，请务必确保，是使用调试符号构建的!

如果您没有调试符号，那么自定义`"name"`部分不会出现在，已编译`.wasm`二进制中，还有堆栈跟踪的是像`wasm-function[42]`这样的函数名称，而不是函数的 Rust 名称，如`wasm_game_of_life::Universe::live_neighbor_count`。

使用"调试"版本时(又称`wasm-pack build --debug`或`cargo build`)，当然默认情况下启用调试符号。

使用"release"构建时，默认情况下不启用调试符号。要启用调试符号，请确保您在`Cargo.toml`的`[profile.release]`部分下，具有`debug = true`:

```toml
[profile.release]
debug = true
```

## 使用 `console` APIs 记录

记录是我们用来证明和反驳我们的程序错误原因的最有效工具之一。在网上，[`console.log`
函数](https://developer.mozilla.org/en-US/docs/Web/API/Console/log)是将消息记录到，浏览器的开发人员工具控制台的方法。

我们可以用[`web-sys`箱][web-sys]可获得`console`记录函数的访问权限:

```rust
extern crate web_sys;

web_sys::console::log_1(&"Hello， world!".into());
```

或者，[`console.error`
函数](https://developer.mozilla.org/en-US/docs/Web/API/Console/error)与`console.log`具有相同的函数签名，不同的是，`console.error`开发人员工具更倾向于在记录消息的同时，捕获并显示堆栈跟踪。

### 参考

- 运用`console.log`随着`web-sys`箱:
  - [`web_sys::console::log` 用一个数组去 log](https://rustwasm.github.io/wasm-bindgen/api/web_sys/console/fn.log.html)
  - [`web_sys::console::log_1` logs 单个值](https://rustwasm.github.io/wasm-bindgen/api/web_sys/console/fn.log_1.html)
  - [`web_sys::console::log_2` logs 两个值](https://rustwasm.github.io/wasm-bindgen/api/web_sys/console/fn.log_2.html)
  - 等等...
- 运用`console.error`随着`web-sys`箱:
  - [`web_sys::console::error` 用一个数组去 log](https://rustwasm.github.io/wasm-bindgen/api/web_sys/console/fn.error.html)
  - [`web_sys::console::error_1` logs 单个值](https://rustwasm.github.io/wasm-bindgen/api/web_sys/console/fn.error_1.html)
  - [`web_sys::console::error_2` logs 两个值](https://rustwasm.github.io/wasm-bindgen/api/web_sys/console/fn.error_2.html)
  - 等等...
- [MDN 的`console` 对象 ](https://developer.mozilla.org/en-US/docs/Web/API/Console)
- [Firefox 开发工具 — Web Console](https://developer.mozilla.org/en-US/docs/Tools/Web_Console)
- [Microsoft Edge 开发工具 — Console](https://docs.microsoft.com/en-us/microsoft-edge/devtools-guide/console)
- [入门 Chrome DevTools Console](https://developers.google.com/web/tools/chrome-devtools/console/get-started)

## 记录 Panics

[该`console_error_panic_hook`crate 将意外的恐慌，记录到开发者控制台`console.error`。][panic-hook]，不再是神秘，难以调试`RuntimeError: unreachable executed`错误消息，给你 Rust 的格式化恐慌消息。

您需要做的就是，在初始化函数或公共代码路径中，通过调用`console_error_panic_hook::set_once()`来安装钩子:

```rust
#[wasm_bindgen]
pub fn init() {
    console_error_panic_hook::set_once();
}
```

[panic-hook]: https://github.com/rustwasm/console_error_panic_hook

## 使用一个 调试器

不幸的是，WebAssembly 的调试'故事'仍然不成熟。在大多数 Unix 系统上，[DWARF][dwarf]是用来编码信息的，信息是调试器提供正运行程序的源级检查的。在 Windows 上有一种替代格式，具有类似的编码信息。目前，没有属于 WebAssembly 的等价物。因此，调试器目前提供有限的实用程序，我们最终逐步执行编译器发出的原始 WebAssembly 指令，而不是我们编写的 Rust 源文本。

> 有一个[用于调试的 W3C WebAssembly 组的子章节][debugging-subcharter]，所以期待这个'故事'在未来有所改善!

[debugging-subcharter]: https://github.com/WebAssembly/debugging
[dwarf]: http://dwarfstd.org/

尽管如此，调试器仍然可用于检查与 WebAssembly 交互的 JavaScript，以及检查原始状态.

### 参考

- [Firefox 开发工具 — Debugger](https://developer.mozilla.org/en-US/docs/Tools/Debugger)
- [Microsoft Edge 开发工具 — Debugger](https://docs.microsoft.com/en-us/microsoft-edge/devtools-guide/debugger)
- [入门：调试 JavaScript in Chrome DevTools](https://developers.google.com/web/tools/chrome-devtools/javascript/)

## 首先避免调试 Webassembly

如果该错误来自与 JavaScript 或 Web API 的交互，那么先[用`wasm-bindgen-test`写测试吧。][wbg-test]

如果bug ，*不*涉及与 JavaScript 或 Web API 的交互，那尝试将其重现为正常的 Rust`#[test]`函数，您可以在调试时，利用操作系统的成熟本机工具。使用测试箱[`quickcheck`][quickcheck]和它的测试案例 shrinkers 呆板地减少测试用例。最终，如果您可以在不需要与 JavaScript 交互的较小测试用例中隔离它们，您将更容易找到，并修复错误。

请注意，为了没有编译器和链接器错误情况下，良好运行本机`#[test]`，您需要确保这`"rlib"`在`[lib.crate-type]`数组内，`Cargo.toml`文件如下：

```toml
[lib]
crate-type ["cdylib"， "rlib"]
```

[quickcheck]: https://crates.io/crates/quickcheck
[web-sys]: https://rustwasm.github.io/wasm-bindgen/web-sys/index.html
[wbg-test]: https://rustwasm.github.io/wasm-bindgen/wasm-bindgen-test/index.html
