# 测试 Conway's 生命游戏

现在我们在浏览器中使用 JavaScript 运用 Rust(编译出 wasm) 实现了生命游戏渲染，接下来让我们来谈谈测试 Rust 生成的 WebAssembly 函数。

我们要测试一下`tick`函数，以确保它为我们提供预期的输出。

下一步,我们将要在`wasm_game_of_life/src/lib.rs`文件的`impl Universe`代码区块内部，创建一些 setter 和 getter 函数。我们准备创建一个`set_width`和 一个`set_height`函数，所以我们可以创建不同大小的`Universe`(宇宙)。

```rust
#[wasm_bindgen]
impl Universe {
    // ...

    /// 设置 宇宙 的 宽度.
    ///
    /// 将所有的单元，重新设为 死亡 状态
    pub fn set_width(&mut self, width: u32) {
        self.width = width;
        self.cells = (0..width * self.height).map(|_i| Cell::Dead).collect();
    }

    /// 设置 宇宙 的 高度.
    ///
    /// 将所有的单元，重新设为 死亡 状态
    pub fn set_height(&mut self, height: u32) {
        self.height = height;
        self.cells = (0..self.width * height).map(|_i| Cell::Dead).collect();
    }

}
```

我们打算创造另一个，一样是`wasm_game_of_life/src/lib.rs`文件中`impl Universe`代码区块内部，但没有`#[wasm_bindgen]`属性。我们需要一些测试所需的函数，但不希望这些函数给到我们的 JavaScript。Rust 生成的 WebAssembly 函数无法返回 **借用的** 引用。尝试下使用属性编译 Rust 生成的 WebAssembly，并查看您获得的错误。

我们将编写实现`get_cells`，主要用来得到一个`Universe`中的`cells`的内容。我们还会写一个`set_cells`函数，这样我们才可以设置一个`Universe`中，特定行列的`cells`为`Alive.`(活的)

```rust
impl Universe {
    /// 给出 全部宇宙 死和活的值
    pub fn get_cells(&self) -> &[Cell] {
        &self.cells
    }

    /// 通过传递 作为数组的 单元(行与列)，可在一个宇宙内设置该单元为活的
    pub fn set_cells(&mut self, cells: &[(u32, u32)]) {
        for (row, col) in cells.iter().cloned() {
            let idx = self.get_index(row, col);
            self.cells[idx] = Cell::Alive;
        }
    }

}
```

现在我们要在`wasm_game_of_life/tests/web.rs`文件中创建我们的测试。

在我们这样做之前,文件中已经有一个工作测试。您可以通过在`wasm-game-of-life`目录里面，运行`wasm-pack test --chrome --headless`确认 Rust 生成的 WebAssembly 测试是否正常工作。你也可以使用`--firefox`,`--safari`,和`--node`在这些浏览器中测试代码的选项.

在`wasm_game_of_life/tests/web.rs`文件里面，我们需要导出我们的`wasm_game_of_life`箱子和`Universe`类型.

```rust
extern crate wasm_game_of_life;
use wasm_game_of_life::Universe;
```

在`wasm_game_of_life/tests/web.rs`文件里面，我们要创建一些太空船(spaceship)构建器函数。

先要一个 input_spaceship(输入的宇宙飞船) ，这样我们会让`tick`函数开启调用，还有我们在一次 tick 后，要获取 expected_spaceship(预期的宇宙飞船) 。我们选择了想要初始化为`Alive`的单元格，在`input_spaceship`函数中创造我们的宇宙飞船。在手动的`input_spaceship`过了一次 tick 后， 宇宙飞船的位置就在`expected_spaceship`函数。您可以自己确认下，在一次 tick 后，输入飞船的单元与预期的相同.

```rust
#[cfg(test)]
pub fn input_spaceship() -> Universe {
    let mut universe = Universe::new();
    universe.set_width(6);
    universe.set_height(6);
    universe.set_cells(&[(1,2), (2,3), (3,1), (3,2), (3,3)]);
    universe
}

#[cfg(test)]
pub fn expected_spaceship() -> Universe {
    let mut universe = Universe::new();
    universe.set_width(6);
    universe.set_height(6);
    universe.set_cells(&[(2,1), (2,3), (3,2), (3,3), (4,2)]);
    universe
}
```

现在我们实现`test_tick`函数。首先,我们创建`input_spaceship()`和`expected_spaceship()`，各一个实例。然后,我们在`input_universe`上调用`tick`。最后,我们使用了`assert_eq!`宏来调用`get_cells()`，确保`input_universe`和`expected_universe`有同样的`Cell`数组值。我们加上了`#[wasm_bindgen_test]`属性到我们的代码块，所以我们可以测试 Rust 生成的 WebAssembly 代码，并使用`wasm-build test`测试 WebAssembly 代码。

```rust
#[wasm_bindgen_test]
pub fn test_tick() {
    // 让我们创建一个小点 的宇宙，带着我们微飞船测试吧!
    let mut input_universe = input_spaceship();

    // 我们宇宙的一次滴答后，我们的飞船应该变成了这样
    let expected_universe = expected_spaceship();

    // 调用 `tick` ，然后看看 `Universe`的单元是否一致
    input_universe.tick();
    assert_eq!(&input_universe.get_cells(), &expected_universe.get_cells());
}
```

在`wasm-game-of-life`目录中，通过运行`wasm-pack test --firefox --headless`运行测试。
