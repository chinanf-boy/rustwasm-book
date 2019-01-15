# 调试

在我们编写更多代码之前,我们需要在出现问题时使用一些调试工具.

## 使用调试符号构建

> ⚡ 调试时,请务必确保使用调试符号构建!

如果您没有启用调试符号,那么`"name"`部分将不会出现在`.wasm`二进制编译中和堆栈跟踪将具有类似的函数名称`wasm[42]`而不是`wasm_game_of_life::Universe::live_neighbor_count`.

使用"debug"版本时 (又称`npm run build-debug`) 默认情况下启用调试符号.

使用"release"构建时,默认情况下不启用调试符号. 要启用调试符号,请确保您`debug = true`在`Cargo.toml`的`[profile.release]`:

```toml
[profile.release]
debug = true
```

默认情况下我们一直在使用的项目模板添加了这个`Cargo.toml`,为方便起见.

## 记录

记录是我们用来证明和反驳我们的程序错误原因的最有效工具之一. 在 web 上,`console.log` 是将消息记录到浏览器的开发人员工具控制台的方法. 我们可以用`wasm_bindgen`导入对它的引用,如下所示:

```rust
#[wasm_bindgen]
extern {
    #[wasm_bindgen(js_namespace = console)]
    fn log(msg: &str);
}

// A macro to 提供 `println!(..)`-style syntax 给 `console.log` logging.
macro_rules! log {
    ($($t:tt)*) => (log(&format!($($t)*)))
}
```

然后,我们可以通过在 Rust 代码中,插入`log`调用将消息记录到控制台. 例如,要记录每个单元的状态,活动邻居数和下一个状态,我们可以像这样修改我们的代码:

```diff
diff --git a/src/lib.rs b/src/lib.rs
index f757641..a30e107 100755
--- a/src/lib.rs
+++ b/src/lib.rs
@@ -63,6 +63,11 @@ impl Universe {
                 let cell = self.cells[idx];
                 let live_neighbors = self.live_neighbor_count(row, col);

+                log!(
+                    "cell[{}, {}] is initially {:?} and has {} live neighbors",
+                    row, col, cell, live_neighbors
+                );
+
                 let next_cell = match (cell, live_neighbors) {
                     // Rule 1: Any live cell with fewer than two live neighbours
                     // dies, as if caused by underpopulation.
@@ -80,6 +85,8 @@ impl Universe {
                     (otherwise, _) => otherwise,
                 };

+                log!("    it becomes {:?}", next_cell);
+
                 next[idx] = next_cell;
             }
         }
```

`console.log`或者`console.error`函数具有相同的接口`,但是,`console.error`在开发人员工具也倾向于在记录消息时捕获并显示堆栈跟踪用来.

### 参考

- [该`console`对象](https://developer.mozilla.org/en-US/docs/Web/API/Console)
- [Firefox 开发人员工具 - Web 控制台](https://developer.mozilla.org/en-US/docs/Tools/Web_Console)
- [Microsoft Edge 开发人员工具 - 控制台](https://docs.microsoft.com/en-us/microsoft-edge/devtools-guide/console)
- [开始使用 Chrome DevTools 控制台](https://developers.google.com/web/tools/chrome-devtools/console/get-started)

## 崩溃日志

[这 `console_error_panic_hook` 箱子 通过`console.error`为开发者记录了异常的崩溃][panic-hook] 而不是得到 怪异的,
难以调试的 `RuntimeError: unreachable executed` 错误信息, 它会给你格式化的崩溃信息.

做到这些, 你只需要在初始化函数中载入钩子或者适应下面的通用代码:

```rust
extern crate console_error_panic_hook;

// Expose an `init` function to be called by JS after instantiating the wasm
// module which installs the `console.error` panic hook.
#[wasm_bindgen]
pub fn init() {
    use std::panic;
    panic::set_hook(Box::new(console_error_panic_hook::hook));
}

// OR

// Ensure the the panic hook is set on some common code path. Under the hood,
// this makes sure that it is only installed once.
console_error_panic_hook::set_once();
```

[panic-hook]: https://github.com/rustwasm/console_error_panic_hook

## 使用调试器

不幸的是,WebAssembly 的调试仍然不成熟. 在大多数 Unix 系统上,[DWARF][dwarf]用于编码调试器提供正在运行的程序的源级检查的信息. 在 Windows 上有一种替代格式可以编码类似的信息. 目前, WebAssembly 没有等价物. 因此,调试器目前提供有限的实用程序,我们最终逐步执行编译器发出的原始 WebAssembly 指令,而不是我们编写的 Rust 源文本.

尽管如此,调试器仍然可用于检查 与 WebAssembly 交互的 JavaScript. 例如,我们可以使用调试器在我们的`renderLoop`函数每次迭代中暂停. 这为我们提供了一个方便的检查点,用于检查记录的消息,并将当前呈现的帧与前一帧进行比较.

[dwarf]: http://dwarfstd.org/

[![Screenshot of debugging the Game of Life](./images/game-of-life/debugging.png)](./images/game-of-life/debugging.png)

### 参考

- [Firefox 开发者工具 - 调试器](https://developer.mozilla.org/en-US/docs/Tools/Debugger)
- [Microsoft Edge 开发人员工具 - 调试器](https://docs.microsoft.com/en-us/microsoft-edge/devtools-guide/debugger)
- [开始在 Chrome DevTools 中调试 JavaScript](https://developers.google.com/web/tools/chrome-devtools/javascript/)

## 首先避免调试 WebAssembly

虽然一些错误特定于 JavaScript 和 WebAssembly 的接口,但经验表明大多数错误都没有. 尝试将 bug 重现 通过正常的 Rust`#[test]`函数,您可以在调试时利用操作系统的成熟工具. 使用测试箱子[`quickcheck`][quickcheck]练习您向 JavaScript 公开的接口. 最终,如果您可以在不需要与 JavaScript 交互 的小测试用例中隔离它们,您将更容易找到并修复错误.

注意,为了运行`#[test]`没有编译器和链接器错误,你需要添加`#![wasm_bindgen]`注释和`crate-type = "cdylib"`.

[quickcheck]: https://crates.io/crates/quickcheck

## 练习

- 添加日志记录到`tick`函数,记录每个单元格的行和列 - 状态从活动转换为死亡,反之亦然.

- 介绍一个`panic!()`在`Universe::new`方法里面. 在 Web 浏览器的 JavaScript 调试器 中检查恐慌的回溯. 禁用调试符号,重建并再次检查堆栈跟踪. 不是很有用,是吗?

- checkout `chapter-one-with-bug` branch. 重建并重新加载网页. 现在很明显, 这个分支的实现包含一个`bug`, 每个单元格显然都是活着的. 这是您的作者在最初创建示例代码时, 所犯的真实世界 (tm) 错误. 找到错误并修复它. _不要看提交历史! 那是作弊 ;-) _
