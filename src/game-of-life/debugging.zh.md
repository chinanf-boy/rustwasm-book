# 调试

在我们编写更多代码之前，我们需要一些，在出现问题时使用的调试工具。花一点时间回顾下[参考页面：可用于调试 Rust 生成的 WebAssembly 的工具和方法][reference-debugging]。

[reference-debugging]: ../reference/debugging.zh.html

## 为恐慌(panic)，启用日志记录，

[如果我们的代码发生恐慌，我们希望在开发者控制台中，显示信息性错误消息。](../reference/debugging.zh.html#logging-panics)

我们的`wasm-pack-template`附带一个[`console_error_panic_hook`箱][panic-hook]，其有个可选的，默认启用的依赖项，你可在`wasm-game-of-life/src/utils.rs`看到已配置了。我们需要做的就是在初始化函数或常用代码路径中安装钩子。我们可以在`wasm-game-of-life/src/lib.rs`中的，`Universe::new`构造函数里面调用它：

```rust
pub fn new() -> Universe {
    utils::set_panic_hook();

    // ...
}
```

[panic-hook]: https://github.com/rustwasm/console_error_panic_hook

## 记录功能，添加到我们的生命游戏中

[运用`console.log`函数的方式，是由`web-sys`添加一些记录日志][logging]，记录我们`Universe::tick`函数的每个细胞。

首先，在`wasm-game-of-life/Cargo.toml`添加`web-sys`依赖项，并启用它的`"console"`功能(特性)：

```toml
[dependencies.web-sys]
version = "0.3"
features = [
  "console",
]
```

为了更符合偷懒，我们将包装`console.log`到类`println!`形式的宏中：

[logging]: ../reference/debugging.zh.html#logging-with-the-console-apis

```rust
extern crate web_sys;

// 一个 macro(宏) 提供 `println!(..)`-形式 语法，给到 `console.log` 日志功能.
macro_rules! log {
    ( $( $t:tt )* ) => {
        web_sys::console::log_1(&format!( $( $t )* ).into());
    }
}
```

现在，我们可以在 Rust 代码中，插入`log`调用，开始 console 的信息记录。例如，要记录每个单元(cell)的状态，实时邻居计数，以及下一个状态，我们可以修改`wasm-game-of-life/src/lib.rs`像这样：

```diff
diff --git a/src/lib.rs b/src/lib.rs
index f757641..a30e107 100755
--- a/src/lib.rs
+++ b/src/lib.rs
@@ -123,6 +122,14 @@ impl Universe {
                 let cell = self.cells[idx];
                 let live_neighbors = self.live_neighbor_count(row, col);

+                log!(
+                    "cell[{}, {}] is initially {:?} and has {} live neighbors",
+                    row,
+                    col,
+                    cell,
+                    live_neighbors
+                );
+
                 let next_cell = match (cell, live_neighbors) {
                     // Rule 1: Any live cell with fewer than two live neighbours
                     // dies, as if caused by underpopulation.
@@ -140,6 +147,8 @@ impl Universe {
                     (otherwise, _) => otherwise,
                 };

+                log!("    it becomes {:?}", next_cell);
+
                 next[idx] = next_cell;
             }
         }
```

## 在每个 Tick 之间，能用调试器暂停下

[浏览器的步进调试器，对检查 Rust 生成的 WebAssembly 与 JavaScript 的交互 非常有用。](../reference/debugging.zh.html#using-a-debugger)

例如，我们可用调试器，在我们的`renderLoop`函数每次迭代时，暂停。只需要放置[一个 JavaScript 的 `debugger;`声明][dbg-stmt]，位于我们的`universe.tick()`之上。

```js
const renderLoop = () => {
  debugger;
  universe.tick();

  drawGrid();
  drawCells();

  requestAnimationFrame(renderLoop);
};
```

这为我们提供了一个方便的检查点，用于检查记录的消息，并将当前渲染的'帧'与前一'帧'进行比较。

[dbg-stmt]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/debugger

[![Screenshot of debugging the Game of Life](../images/game-of-life/debugging.png)](../images/game-of-life/debugging.png)

## 练习

- 添加日志记录到`tick`函数，这样就可以记录每个单元格的行和列，知道其状态是从活转换为死，或是相反。

- 声明一个`panic!()`到`Universe::new`方法里面。在 Web 浏览器的 JavaScript 调试器中，检查恐慌的回溯。要做到禁用调试符号，只用不带`console_error_panic_hook`可选依赖项，重新构建就好了，并再次检查堆栈跟踪。可以吧！？
