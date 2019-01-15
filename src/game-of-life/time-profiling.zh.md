# 时间分析

本章介绍如何使用 Rust 和 WebAssembly 来分析 Web 页面,其目标是提高吞吐量或降低延迟.

> ⚡ 始终确保使用的是`--release`分析! 使用我们的项目模板,这意味着使用`npm run build-release`代替`npm run build-debug`.

## 可用工具

### 该`performance.now()`计时器

该[`performance.now()`][perf-now]函数返回,自加载网页以来以毫秒为单位测量的单调时间戳. 我们可以使用它来计算各种操作的时间,我们可以通过以下方式 从 Rust 访问它,通过`wasm_bindgen`导入申报:

```rust
#[wasm_bindgen]
extern {
    #[wasm_bindgen(js_namespace = performance)]
    fn now() -> f64;
}
```

调用`performance.now`开销很小,因此我们可以从中创建简单的测量,而不会扭曲系统其他部分的性能.

例如,我们可以创建一个简单的 `FPS-帧数` 计数器,我们在每次迭代`renderLoop`时更新.

让我们开始在`index.js`添加一个`fps`对象

```js
const fps = new class {
  constructor() {
    this.fps = document.getElementById('fps');
    this.frames = [];
    this.lastFrameTimeStamp = performance.now();
  }

  render() {
    // 时间计算.
    const now = performance.now();
    const delta = now - this.lastFrameTimeStamp;
    this.lastFrameTimeStamp = now;
    const fps = (1 / delta) * 1000;

    // Save only the latest 100 timings.
    this.frames.push(fps);
    if (this.frames.length > 100) {
      this.frames.shift();
    }

    // 找出我们100个最新时间的最大值，最小值和平均值.
    let min = Infinity;
    let max = -Infinity;
    let sum = 0;
    for (let i = 0; i < this.frames.length; i++) {
      sum += this.frames[i];
      min = Math.min(this.frames[i], min);
      max = Math.max(this.frames[i], max);
    }
    let mean = sum / this.frames.length;

    // Render the statistics.
    this.fps.textContent = `
Frames per Second:
         latest = ${Math.round(fps)}
avg of last 100 = ${Math.round(mean)}
min of last 100 = ${Math.round(min)}
max of last 100 = ${Math.round(max)}
`.trim();
  }
}();
```

接下来 我们运行 `fps` `render` 函数 ,在每次迭代`renderLoop`时更新:

```js
const renderLoop = () => {
  fps.render(); //new

  universe.tick();
  drawCells();
  drawGrid();

  animationId = requestAnimationFrame(renderLoop);
};
```

最后, 不要忘记添加 `fps` 元素 到 `index.html`:

```html
    <div id="fps"><div>
```

噢耶! 现在你有了一个 FPS 计数器!

[perf-now]: https://developer.mozilla.org/en-US/docs/Web/API/Performance/now

### 开发人员工具 Profilers

所有 Web 浏览器 的内置开发人员工具都包含一个分析器. 这些分析器显示哪些函数花费最多时间的可视化,如调用树和火焰图. 如果你[用调试符号构建][symbols],然后这些分析器应该显示 Rust 函数名称 而不是像`wasm-function[123]`. 请注意这些分析器*惯于*显示内联函数,由于 Rust 和 LLVM 依赖于如此大量的内联,结果可能会有点令人困惑.

[symbols]: ./game-of-life/debugging.html#building-with-debug-symbols

[![Screenshot of profiler with Rust symbols](./images/game-of-life/profiler-with-rust-names.png)](./images/game-of-life/profiler-with-rust-names.png)

#### 资源

- [Firefox 开发者工具 - 性能](https://developer.mozilla.org/en-US/docs/Tools/Performance)
- [Microsoft Edge 开发人员工具 - 性能](https://docs.microsoft.com/en-us/microsoft-edge/devtools-guide/performance)
- [Chrome DevTools JavaScript Profiler](https://developers.google.com/web/tools/chrome-devtools/rendering-tools/js-execution)

### `console.time`和`console.timeEnd`功能

`console.time`和`console.timeEnd`函数允许您将命名操作的时间记录到浏览器的开发人员工具控制台.

你可以用它将它们导入 Rust`wasm-bindgen` 声明中:

```rust
#[wasm_bindgen]
extern {
    #[wasm_bindgen(js_namespace = console)]
    fn time(name: &str);

    #[wasm_bindgen(js_namespace = console)]
    fn timeEnd(name: &str);
}
```

因为`console.timeEnd`有对应的`console.time`调用,将它们包装在 RAII 类型中很方便:

```rust
pub struct Timer<'a> {
    name: &'a str,
}

impl<'a> Timer<'a> {
    pub fn new(name: &'a str) -> Timer<'a> {
        time(name);
        Timer { name }
    }
}

impl<'a> Drop for Timer<'a> {
    fn drop(&mut self) {
        timeEnd(self.name);
    }
}
```

然后,我们可以计算每个时间`Universe::tick`,将此代码段添加到方法的顶部:

```rust
let _timer = Timer::new("Universe::tick");
```

现在每次运行的时间`Universe::tick`都会记录了:

[![Screenshot of console.time logs](./images/game-of-life/console-time.png)](./images/game-of-life/console-time.png)

另外,`console.time`和`console.timeEnd`对将显示在,浏览器的分析器的"timeline"或"waterfall"视图中:

[![Screenshot of console.time logs](./images/game-of-life/console-time-in-profiler.png)](./images/game-of-life/console-time-in-profiler.png)

### 运用`#[bench]`与本地代码

我们通常也可以通过编写,来利用我们操作系统的本机代码调试工具`#[test]`而不是在 Web 上调试,我们也可以通过编写`#[bench]`功能.

然而!在为本机代码分析投入大量精力之前,请确保您知道瓶颈在 WebAssembly 中! 使用浏览器的分析器确认这一点,否则您可能会浪费时间来优化不热的代码.

## 发展我们的生命博弈宇宙

如果我们让我们的生命游戏世界更大,会发生什么? 使用 128 x 128 宇宙替换 64 x 64 宇宙导致 FPS 从 平滑的 60 下降到 40-ish.

如果我们记录一个配置文件并查看瀑布视图,我们会看到每个动画帧花费超过 20 毫秒. 回想一下,每秒 60 帧,每帧大概 16 毫秒. 16 毫秒内不仅仅是有我们的 JavaScript 和 WebAssembly ,还有浏览器正在做的其他事情,比如绘制页面.

[![Screenshot of a waterfall view of rendering a frame](./images/game-of-life/drawCells-before-waterfall.png)](./images/game-of-life/drawCells-before-waterfall.png)

如果我们看一下单个动画帧中发生的事情,我们就会看到`CanvasRenderingContext2D.fillStyle` 很长!

[![Screenshot of a flamegraph view of rendering a frame](./images/game-of-life/drawCells-before-flamegraph.png)](./images/game-of-life/drawCells-before-flamegraph.png)

我们可以通过查看调用树的多个帧的聚合来确认这不是异常:

[![Screenshot of a flamegraph view of rendering a frame](./images/game-of-life/drawCells-before-calltree.png)](./images/game-of-life/drawCells-before-calltree.png)

我们将近 40%的时间都花在了这个函数身上!

> ⚡ 我们可能对此有所期待`tick`方法是性能瓶颈,但事实并非如此. 让分析引导您的注意力,而不是把时间花在您想当然的地方.

在`drawCells`函数,在每个动画帧上,`fillStyle`为宇宙中的每个单元格设置一次:

```js
for (let row = 0; row < height; row++) {
  for (let col = 0; col < width; col++) {
    const idx = getIndex(row, col);

    ctx.fillStyle = cells[idx] === DEAD ? DEAD_COLOR : ALIVE_COLOR;

    ctx.fillRect(
      col * (CELL_SIZE + 1) + 1,
      row * (CELL_SIZE + 1) + 1,
      CELL_SIZE,
      CELL_SIZE
    );
  }
}
```

现在我们已经发现了这个`fillStyle`是如此昂贵,我们可以做些什么来避免经常设置它? 改变`fillStyle`取决于细胞是活着还是死亡. 如果我们设定`fillStyle = ALIVE_COLOR`, 然后在一次总绘制每个活细胞,然后设置`fillStyle = DEAD_COLOR`,并在另一次总绘制每个死细胞,那么我们只设置`fillStyle`两次,而不是每个细胞一次.

```js
// Alive cells.
ctx.fillStyle = ALIVE_COLOR;
for (let row = 0; row < height; row++) {
  for (let col = 0; col < width; col++) {
    const idx = getIndex(row, col);
    if (cells[idx] !== ALIVE) {
      continue;
    }

    ctx.fillRect(
      col * (CELL_SIZE + 1) + 1,
      row * (CELL_SIZE + 1) + 1,
      CELL_SIZE,
      CELL_SIZE
    );
  }
}

// Dead cells.
ctx.fillStyle = DEAD_COLOR;
for (let row = 0; row < height; row++) {
  for (let col = 0; col < width; col++) {
    const idx = getIndex(row, col);
    if (cells[idx] !== DEAD) {
      continue;
    }

    ctx.fillRect(
      col * (CELL_SIZE + 1) + 1,
      row * (CELL_SIZE + 1) + 1,
      CELL_SIZE,
      CELL_SIZE
    );
  }
}
```

保存这些更改并刷新后[http://localhost:8080/](http://localhost:8080/),渲染回到平滑每秒 60 帧.

如果我们采用另一个配置文件,我们可以看到现在每个动画帧只花费大约 10 毫秒.

[![Screenshot of a waterfall view of rendering a frame after the drawCells changes](./images/game-of-life/drawCells-after-waterfall.png)](./images/game-of-life/drawCells-after-waterfall.png)

打破了超时帧,我们看到了`fillStyle`昂贵的成本消失了,我们的大部分时间花在了内部`fillRect`,绘制每个单元格的矩形.

[![Screenshot of a flamegraph view of rendering a frame after the drawCells changes](./images/game-of-life/drawCells-after-flamegraph.png)](./images/game-of-life/drawCells-after-flamegraph.png)

## 让时间更快

有些人不喜欢等待,认为每个动画帧是 9 个 `tick`. 我们可以修改`renderLoop`函数,这很容易做到这一点:

```js
for (let i = 0; i < 9; i++) {
  universe.tick();
}
```

在我的机器上,这使我们恢复到每秒 35 帧. 不好. 我们想要那个美丽的 `60!`

现在我们知道时间花在了`Universe::tick`,让我们添加一些`Timer`,用它来包装,给予`console.time`和`console.timeEnd`运行,引导我们. 我的假设是,分配一个新的细胞载体,并在每个滴答上释放旧的是昂贵的成本,占用了我们时间预算的很大一部分.

```rust
pub fn tick(&mut self) {
    let _timer = Timer::new("Universe::tick");

    let mut next = {
        let _timer = Timer::new("allocate next cells");
        self.cells.clone()
    };

    {
        let _timer = Timer::new("new generation");
        for row in 0..self.height {
            for col in 0..self.width {
                let idx = self.get_index(row, col);
                let cell = self.cells[idx];
                let live_neighbors = self.live_neighbor_count(row, col);

                let next_cell = match (cell, live_neighbors) {
                    // Rule 1: Any live cell with fewer than two live neighbours
                    // dies, as if caused by underpopulation.
                    (Cell::Alive, x) if x < 2 => Cell::Dead,
                    // Rule 2: Any live cell with two or three live neighbours
                    // lives on to the next generation.
                    (Cell::Alive, 2) | (Cell::Alive, 3) => Cell::Alive,
                    // Rule 3: Any live cell with more than three live
                    // neighbours dies, as if by overpopulation.
                    (Cell::Alive, x) if x > 3 => Cell::Dead,
                    // Rule 4: Any dead cell with exactly three live neighbours
                    // becomes a live cell, as if by reproduction.
                    (Cell::Dead, 3) => Cell::Alive,
                    // All other cells remain in the same state.
                    (otherwise, _) => otherwise,
                };

                next[idx] = next_cell;
            }
        }
    }

    let _timer = Timer::new("free old cells");
    self.cells = next;
}
```

看看时间,很明显我的假设是不正确的: 实际上绝大部分时间花在,计算下一代细胞上. 令人惊讶的是,在每个单元上分配和释放载体似乎具有可忽略的成本. 再一次告诉指导我们的分析工作!

[![Screenshot of a Universe::tick timer results](./images/game-of-life/console-time-in-universe-tick.png)](./images/game-of-life/console-time-in-universe-tick.png)

我们可以写一个本机代码`#[bench]`来做同样的事情 - 我们的 WebAssembly 正在做的, 但我们可以使用更成熟的分析工具. 这就是新的`benches/bench.rs`:

```rust
#![feature(test)]

extern crate test;
extern crate wasm_game_of_life;

#[bench]
fn universe_ticks(b: &mut test::Bencher) {
    let mut universe = wasm_game_of_life::Universe::new();

    b.iter(|| {
        universe.tick();
    });
}
```

我们还要给上所有的`#[wasm_bindgen]`注释和`"cdylib"` - 来自`Cargo.toml`, 否则构建本机代码将失败并出现链接错误.

有了这一切,我们就可以运行了`cargo bench`编译并运行我们的基准测试!

    $ cargo bench
        Finished release [optimized + debuginfo] target(s) in 0.0 secs
         Running target/release/deps/wasm_game_of_life-91574dfbe2b5a124

    running 0 tests

    test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out

         Running target/release/deps/bench-8474091a05cfa2d9

    running 1 test
    test universe_ticks ... bench:     664,421 ns/iter (+/- 51,926)

    test result: ok. 0 passed; 0 failed; 0 ignored; 1 measured; 0 filtered out

这也告诉我们二进制文件的位置,我们可以再次运行基准测试,但这次是在我们的操作系统的分析器下. 就我而言,我运行在 Linux,所以[`perf`][perf]是我将使用的探查器:

[perf]: https://perf.wiki.kernel.org/index.php/Main_Page

    $ perf record -g target/release/deps/bench-8474091a05cfa2d9 --bench
    running 1 test
    test universe_ticks ... bench:     635,061 ns/iter (+/- 38,764)

    test result: ok. 0 passed; 0 failed; 0 ignored; 1 measured; 0 filtered out

    [ perf record: Woken up 1 times to write data ]
    [ perf record: Captured and wrote 0.178 MB perf.data (2349 samples) ]

使用`perf report`加载配置文件后,表明我们所有的时间都花在了`Universe::tick`,如预期的那样:

[![Screenshot of perf report](./images/game-of-life/bench-perf-report.png)](./images/game-of-life/bench-perf-report.png)

`perf`注释说明函数的时间,通过指令`a`:

[![Screenshot of perf's instruction annotation](./images/game-of-life/bench-perf-annotate.png)](./images/game-of-life/bench-perf-annotate.png)

这告诉我们 `26.67%` 的时间用于求和相邻单元格的值,`23.41%` 的时间用于获取邻居的列索引,另外 `15.42%` 的时间用于获取邻居的行索引. 在这三个最昂贵的指令中,第二个和第三个指令都很昂贵之于`div 说明`. `div`实现模数索引逻辑在`Universe::live_neighbor_count`.

回想一下`live_neighbor_count`定义:

```rust
fn live_neighbor_count(&self, row: u32, column: u32) -> u8 {
    let mut count = 0;
    for delta_row in [self.height - 1, 0, 1].iter().cloned() {
        for delta_col in [self.width - 1, 0, 1].iter().cloned() {
            if delta_row == 0 && delta_col == 0 {
                continue;
            }

            let neighbor_row = (row + delta_row) % self.height;
            let neighbor_col = (column + delta_col) % self.width;
            let idx = self.get_index(neighbor_row, neighbor_col);
            count += self.cells[idx] as u8;
        }
    }
    count
}
```

我们使用 多条件 的原因是为了避免使代码混乱, 通过使用`if`,第一行或最后一行或列边缘情况的条件. 但我们付出的代价是`div`, 也就是最常见的情况,既不是`row`也不是`column`,而是在宇宙的边缘,他们不需要模数包裹处理. 相反,如果我们使用多个`if`为边缘情况,并展开此循环,CPU 的分支预测器*应该*可以很好地预测它.

我们改写`live_neighbor_count`,像这样这个:

```rust
fn live_neighbor_count(&self, row: u32, column: u32) -> u8 {
    let mut count = 0;

    let north = if row == 0 {
        self.height - 1
    } else {
        row - 1
    };

    let south = if row == self.height - 1 {
        0
    } else {
        row + 1
    };

    let west = if column == 0 {
        self.width - 1
    } else {
        column - 1
    };

    let east = if column == self.width - 1 {
        0
    } else {
        column + 1
    };

    let nw = self.get_index(north, west);
    count += self.cells[nw] as u8;

    let n = self.get_index(north, column);
    count += self.cells[n] as u8;

    let ne = self.get_index(north, east);
    count += self.cells[ne] as u8;

    let w = self.get_index(row, west);
    count += self.cells[w] as u8;

    let e = self.get_index(row, east);
    count += self.cells[e] as u8;

    let sw = self.get_index(south, west);
    count += self.cells[sw] as u8;

    let s = self.get_index(south, column);
    count += self.cells[s] as u8;

    let se = self.get_index(south, east);
    count += self.cells[se] as u8;

    count
}
```

现在让我们再次运行基准测试!

    $ cargo bench
       Compiling wasm_game_of_life v0.1.0 (file:///home/fitzgen/wasm_game_of_life)
        Finished release [optimized + debuginfo] target(s) in 0.82 secs
         Running target/release/deps/wasm_game_of_life-91574dfbe2b5a124

    running 0 tests

    test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out

         Running target/release/deps/bench-8474091a05cfa2d9

    running 1 test
    test universe_ticks ... bench:      87,258 ns/iter (+/- 14,632)

    test result: ok. 0 passed; 0 failed; 0 ignored; 1 measured; 0 filtered out

看起来好多了! 我们可以看到它有多好,通过[`cargo benchcmp`][benchcmp]工具:

[benchcmp]: https://github.com/BurntSushi/cargo-benchcmp

    $ cargo benchcmp before.txt after.txt
     name            before.txt ns/iter  after.txt ns/iter  diff ns/iter   diff %  speedup
     universe_ticks  664,421             87,258                 -577,163  -86.87%   x 7.61

哇! 7.61 倍加速!

WebAssembly 映射到常见的硬件架构,但我们确实需要为这个本机代码加速.

让我们恢复所有`#[wasm_bindgen]`注释,重建`.wasm`同`npm run build-release`,并刷新[http://localhost:8080/](http://localhost:8080/). 在我的机器上,页面再次以每秒 60 帧的速度运行,并且使用浏览器的分析器记录,另一个配置文件显示每个动画帧大约需要 10 毫秒.

成功!

[![Screenshot of a waterfall view of rendering a frame after replacing modulos with branches](./images/game-of-life/waterfall-after-branches-and-unrolling.png)](./images/game-of-life/waterfall-after-branches-and-unrolling.png)

## 练习

- 在这一点上,下一个加速`Universe::tick`是删除分配和免费. 实现细胞的双缓冲,其中`Universe`维护两个向量,并且不释放它们中的任何一个,那么就永远不会分配新的缓冲区`tick`.

- 从"实现生命"一章的实现是 基于 delta 的设计,其中 Rust 代码 返回 将状态更改为 JavaScript 的单元格列表. 这会使渲染`<canvas>`更快? 你可以实现这个设计, 而不必在每个滴答上分配一个新的增量列表吗 ?

- 正如我们的分析向我们展示的那样,2D`<canvas>`渲染速度不是特别快. 用 [WebGL] 渲染器替换 2 canvas 渲染器. WebGL 版本的速度有多快 ? 在 WebGL 渲染成为瓶颈之前,你能创造多大的宇宙 ?

[webgl]: https://developer.mozilla.org/en-US/docs/Web/API/WebGL_API
