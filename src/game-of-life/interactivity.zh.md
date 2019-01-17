# 增加交互性

我们将通过在 Game of Life 实现中, 添加一些交互功能来继续探索 JavaScript 和 WebAssembly 接口。 我们想让用户通过单击，来切换单元格是活的还是死亡，并允许暂停游戏，这使得绘制单元格模式更加容易。

## 暂停和恢复游戏

让我们添加一个按钮，来切换游戏是正在播放还是暂停。 `index.html`的`<canvas>`上方添加按钮:

```html
<button id="play-pause"></button>
```

在 JavaScript 中，我们将进行以下更改:

- 跟踪最新调用`requestAnimationFrame`返回的标识符,
  以便我们可以以此，调用`cancelAnimationFrame`来取消那个标识符动画。

- 单击播放/暂停按钮时，检查我们是否具有排队动画帧的标识符。 1.点击时，游戏当前正在播放，取消动画帧`renderLoop`，有效地暂停游戏. 2.点击时，当前暂停，若没有排队动画帧的标识符，我们想运行`requestAnimationFrame`恢复比赛。

因为 JavaScript 正在驱动 Rust 和 WebAssembly，这就是我们需要做的所有，不过我们不需要更改 Rust 源代码。

我们介绍一下`animationId`变量来跟踪`requestAnimationFrame`返回的标识符。 当没有排队的动画帧时，我们将此变量设置为`null`。

```js
let animationId = null;

// 这个函数与之前的一样, 除了把`requestAnimationFrame`的结果
// 分配到 `animationId`.
const renderLoop = () => {
  universe.tick();

  drawCells();
  drawGrid();

  animationId = requestAnimationFrame(renderLoop);
};
```

在任何时刻，我们都可以通过`animationId`检查游戏，来判断游戏是否暂停:

```js
const isPaused = () => {
  return animationId === null;
};
```

现在，当点击 播放/暂停 按钮时，我们会检查游戏当前是暂停还是正在播放，要么继续播放`renderLoop`动画，要么取消下一个动画帧。
此外，我们更新按钮的文本图标，以反映按钮在下次单击时将执行的操作。

```js
const playPauseButton = document.getElementById('play-pause');

const play = () => {
  playPauseButton.textContent = '⏸';
  renderLoop();
};

const pause = () => {
  playPauseButton.textContent = '▶';
  cancelAnimationFrame(animationId);
  animationId = null;
};

playPauseButton.addEventListener('click', event => {
  if (isPaused()) {
    play();
  } else {
    pause();
  }
});
```

最后，我们直接调用`requestAnimationFrame(renderLoop)`用来启动之前的游戏及其动画，
但我们想用`play`替换它，以便按钮获得正确的初始文本图标。

```diff
// This used to be `requestAnimationFrame(renderLoop)`.
play();
```

刷新[http://localhost:8080/](http://localhost:8080/)，现在你应该可以通过点击按钮来暂停和恢复游戏!

## 切换一个 Cell 的状态`"click"`活动

现在我们可以暂停游戏了，现在是时候添加通过点击它们来改变细胞的能力了。

切换单元格是将其状态从活动状态转换为死亡状态，或从死亡状态转换为活动状态:

```rust
impl Cell {
    fn toggle(&mut self) {
        *self = match *self {
            Cell::Dead => Cell::Alive,
            Cell::Alive => Cell::Dead,
        };
    }
}
```

要切换给定行和列的单元格状态，我们将行和列对转换为单元格向量的索引，并在该索引处的单元格上调用`toggle`方法:

```rust
/// 公有方法 methods， 导出到 JavaScript.
#[wasm_bindgen]
impl Universe {
    // ...

    pub fn toggle_cell(&mut self, row: u32, column: u32) {
        let idx = self.get_index(row, column);
        self.cells[idx].toggle();
    }
}
```

这个方法是在`impl`带有`#[wasm_bindgen]`注释的区块内，这样它就可以被 JavaScript 调用。

在 JavaScript 中，我们会监听 点击事件`<canvas>`元素，将 `click事件`的页面 相对坐标转换为画布相对坐标，
然后转换为行和列，调用`toggle_cell`方法，最后重绘场景。

```js
canvas.addEventListener('click', event => {
  const boundingRect = canvas.getBoundingClientRect();

  const scaleX = canvas.width / boundingRect.width;
  const scaleY = canvas.height / boundingRect.height;

  const canvasLeft = (event.clientX - boundingRect.left) * scaleX;
  const canvasTop = (event.clientY - boundingRect.top) * scaleY;

  const row = Math.min(Math.floor(canvasTop / (CELL_SIZE + 1)), height - 1);
  const col = Math.min(Math.floor(canvasLeft / (CELL_SIZE + 1)), width - 1);

  universe.toggle_cell(row, col);

  drawCells();
  drawGrid();
});
```

再次刷新[http://localhost:8080/](http://localhost:8080/),您现在可以通过单击单元格，并切换其状态来绘制自己的模式.

您可以在 checkout `chapter-two` 分支 , 找到此实现的完整源代码.

## 练习

- 添加一个[`<input type="range">`][input-range]用于控制每个动画帧出现多少`tick`的小部件。

- 添加一个将`Universe`重置为随机初始状态的按钮。 另一个按钮将宇宙重置为所有死细胞。

- `Ctrl + Click`，在目标细胞的中心插入一个[滑翔机(glider)](<https://en.wikipedia.org/wiki/Glider_(Conway%27s_Life)>)。 `Shift + Click`，插入一个脉冲星(pulsar)。

[input-range]: https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input/range
