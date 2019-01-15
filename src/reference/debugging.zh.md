# Debugging Rust-Generated WebAssembly

本节包含调试Rust生成的WebAssembly的提示.

## Building with Debug Symbols

> ⚡调试时,请务必确保使用调试符号构建!

如果您没有启用调试符号,那么`"name"`自定义部分不会出现在编译中`.wasm`二进制和堆栈跟踪将具有类似的函数名称`wasm-function[42]`而不是函数的Rust名称,比如`wasm_game_of_life::Universe::live_neighbor_count`.

使用"调试"版本时(又称`wasm-pack build --debug`要么`cargo build`)默认情况下启用调试符号.

使用"release"构建时,默认情况下不启用调试符号.要启用调试符号,请确保您`debug = true`在里面`[profile.release]`你的部分`Cargo.toml`:

```toml
[profile.release]
debug = true
```

## Logging with the `console` APIs

记录是我们用来证明和反驳我们的程序错误原因的最有效工具之一.在网上,[the `console.log`
function](https://developer.mozilla.org/en-US/docs/Web/API/Console/log)是将消息记录到浏览器的开发人员工具控制台的方法.

我们可以用[该`web-sys`箱][web-sys]获得访问权限`console`记录功能:

```rust
extern crate web_sys;

web_sys::console::log_1(&"Hello, world!".into());
```

或者,[the `console.error`
function](https://developer.mozilla.org/en-US/docs/Web/API/Console/error)与...有相同的签名`console.log`但是,开发人员工具也倾向于在记录消息的同时捕获并显示堆栈跟踪`console.error`用来.

### References

-   运用`console.log`随着`web-sys`箱:
    -   [`web_sys::console::log` takes an array of values to log](https://rustwasm.github.io/wasm-bindgen/api/web_sys/console/fn.log.html)
    -   [`web_sys::console::log_1` logs a single value](https://rustwasm.github.io/wasm-bindgen/api/web_sys/console/fn.log_1.html)
    -   [`web_sys::console::log_2` logs two values](https://rustwasm.github.io/wasm-bindgen/api/web_sys/console/fn.log_2.html)
    -   等等...
-   运用`console.error`随着`web-sys`箱:
    -   [`web_sys::console::error` takes an array of values to log](https://rustwasm.github.io/wasm-bindgen/api/web_sys/console/fn.error.html)
    -   [`web_sys::console::error_1` logs a single value](https://rustwasm.github.io/wasm-bindgen/api/web_sys/console/fn.error_1.html)
    -   [`web_sys::console::error_2` logs two values](https://rustwasm.github.io/wasm-bindgen/api/web_sys/console/fn.error_2.html)
    -   等等...
-   [The `console` object on MDN](https://developer.mozilla.org/en-US/docs/Web/API/Console)
-   [Firefox Developer Tools — Web Console](https://developer.mozilla.org/en-US/docs/Tools/Web_Console)
-   [Microsoft Edge Developer Tools — Console](https://docs.microsoft.com/en-us/microsoft-edge/devtools-guide/console)
-   [Get Started with the Chrome DevTools Console](https://developers.google.com/web/tools/chrome-devtools/console/get-started)

## Logging Panics

[该`console_error_panic_hook`crate将意外的恐慌记录到开发者控制台`console.error`.][panic-hook]而不是变得神秘,难以调试`RuntimeError: unreachable executed`错误消息,这给你Rust的格式化恐慌消息.

您需要做的就是通过调用来安装钩子`console_error_panic_hook::set_once()`在初始化函数或公共代码路径中:

```rust
#[wasm_bindgen]
pub fn init() {
    console_error_panic_hook::set_once();
}
```

[panic-hook]: https://github.com/rustwasm/console_error_panic_hook

## Using a Debugger

不幸的是,WebAssembly的调试故事仍然不成熟.在大多数Unix系统上,[侏儒][dwarf]用于编码调试器需要提供正在运行的程序的源级检查的信息.在Windows上有一种替代格式可以编码类似的信息.目前,没有WebAssembly的等价物.因此,调试器目前提供有限的实用程序,我们最终逐步执行编译器发出的原始WebAssembly指令,而不是我们编写的Rust源文本.

> 有一个[用于调试的W3C WebAssembly组的子章程][debugging-subcharter],所以期待这个故事在未来有所改善!

[debugging-subcharter]: https://github.com/WebAssembly/debugging

[dwarf]: http://dwarfstd.org/

尽管如此,调试器仍然可用于检查与WebAssembly交互的JavaScript,以及检查原始状态.

### References

-   [Firefox Developer Tools — Debugger](https://developer.mozilla.org/en-US/docs/Tools/Debugger)
-   [Microsoft Edge Developer Tools — Debugger](https://docs.microsoft.com/en-us/microsoft-edge/devtools-guide/debugger)
-   [Get Started with Debugging JavaScript in Chrome DevTools](https://developers.google.com/web/tools/chrome-devtools/javascript/)

## Avoid the Need to Debug WebAssembly in the First Place

如果该错误特定于与JavaScript或Web API的交互,那么[写测试`wasm-bindgen-test`.][wbg-test]

如果有bug的话*不*涉及与JavaScript或Web API的交互,然后尝试将其重现为正常的Rust`#[test]`功能,您可以在调试时利用操作系统的成熟本机工具.使用测试板条箱[`quickcheck`][quickcheck]和它的测试案例shrinkers机械减少测试用例.最终,如果您可以在不需要与JavaScript交互的较小测试用例中隔离它们,您将更容易找到并修复错误.

请注意,为了运行本机`#[test]`如果没有编译器和链接器错误,您需要确保这一点`"rlib"`包括在内`[lib.crate-type]`数组在你的`Cargo.toml`文件.

```toml
[lib]
crate-type ["cdylib", "rlib"]
```

[quickcheck]: https://crates.io/crates/quickcheck

[web-sys]: https://rustwasm.github.io/wasm-bindgen/web-sys/index.html

[wbg-test]: https://rustwasm.github.io/wasm-bindgen/wasm-bindgen-test/index.html
