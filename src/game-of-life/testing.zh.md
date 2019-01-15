# Testing Conway's Game of Life

现在我们在浏览器中使用JavaScript实现了生命游戏渲染的Rust实现,让我们来谈谈测试Rust生成的WebAssembly函数.

我们要测试一下`tick`功能,以确保它为我们提供了我们期望的输出.

接下来,我们将要在现有的内部创建一些setter和getter函数`impl Universe`阻止在`wasm_game_of_life/src/lib.rs`文件.我们要创建一个`set_width`和a`set_height`功能,所以我们可以创建`Universe`不同大小的.

```rust
#[wasm_bindgen]
impl Universe { 
    // ...

    /// Set the width of the universe.
    ///
    /// Resets all cells to the dead state.
    pub fn set_width(&mut self, width: u32) {
        self.width = width;
        self.cells = (0..width * self.height).map(|_i| Cell::Dead).collect();
    }

    /// Set the height of the universe.
    ///
    /// Resets all cells to the dead state.
    pub fn set_height(&mut self, height: u32) {
        self.height = height;
        self.cells = (0..self.width * height).map(|_i| Cell::Dead).collect();
    }

}
```

我们要创造另一个`impl Universe`阻挡我们的内心`wasm_game_of_life/src/lib.rs`文件没有`#[wasm_bindgen]`属性.我们需要一些测试所需的功能,我们不希望公开我们的JavaScript.Rust生成的WebAssembly函数无法返回借用的引用.尝试使用属性编译Rust生成的WebAssembly,并查看您获得的错误.

我们将编写实现`get_cells`得到的内容`cells`一个`Universe`.我们还会写一个`set_cells`功能所以我们可以设置`cells`在a的特定行和列中`Universe`成为`Alive.`

```rust
impl Universe {
    /// Get the dead and alive values of the entire universe.
    pub fn get_cells(&self) -> &[Cell] {
        &self.cells
    }

    /// Set cells to be alive in a universe by passing the row and column
    /// of each cell as an array.
    pub fn set_cells(&mut self, cells: &[(u32, u32)]) {
        for (row, col) in cells.iter().cloned() {
            let idx = self.get_index(row, col);
            self.cells[idx] = Cell::Alive;
        }
    }

}
```

现在我们要在中创建我们的测试`wasm_game_of_life/tests/web.rs`文件.

在我们这样做之前,文件中已经有一个工作测试.您可以通过运行确认Rust生成的WebAssembly测试是否正常工作`wasm-pack test --chrome --headless`在里面`wasm-game-of-life`目录.你也可以使用`--firefox`,`--safari`,和`--node`在这些浏览器中测试代码的选项.

在里面`wasm_game_of_life/tests/web.rs`文件,我们需要导出我们的`wasm_game_of_life`箱子和`Universe`类型.

```rust
extern crate wasm_game_of_life;
use wasm_game_of_life::Universe;
```

在里面`wasm_game_of_life/tests/web.rs`文件我们要创建一些太空船构建器功能.

我们需要一个我们称之为的输入太空船`tick`功能开启,我们希望在一次打勾后得到预期的太空船.我们选择了我们想要初始化的单元格`Alive`创造我们的宇宙飞船`input_spaceship`功能.宇宙飞船的位置`expected_spaceship`功能打勾后的功能`input_spaceship`是手动计算的.您可以自己确认一次打勾后输入飞船的细胞与预期的飞船相同.

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

现在我们将为我们编写实现`test_tick`功能.首先,我们创建一个我们的实例`input_spaceship()`和我们的`expected_spaceship()`.然后,我们打电话`tick`在...上`input_universe`.最后,我们使用了`assert_eq!`宏来打电话`get_cells()`为了确保`input_universe`和`expected_universe`有同样的`Cell`数组值.我们加上了`#[wasm_bindgen_test]`属性到我们的代码块,所以我们可以测试Rust生成的WebAssembly代码并使用它们`wasm-build test`测试WebAssembly代码.

```rust
#[wasm_bindgen_test]
pub fn test_tick() {
    // Let's create a smaller Universe with a small spaceship to test!
    let mut input_universe = input_spaceship();

    // This is what our spaceship should look like
    // after one tick in our universe.
    let expected_universe = expected_spaceship();

    // Call `tick` and then see if the cells in the `Universe`s are the same.
    input_universe.tick();
    assert_eq!(&input_universe.get_cells(), &expected_universe.get_cells());
}
```

在内部运行测试`wasm-game-of-life`目录通过运行`wasm-pack test --firefox --headless`.
