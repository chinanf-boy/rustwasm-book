# 实施康威的生命游戏

## 设计

在我们深入之前,我们有一些设计选择需要考虑.

### 无限的宇宙

生命游戏是在无限的宇宙中进行的,但我们没有无限的记忆和计算能力. 解决这个相当恼人的限制通常有以下三种风格之一:

1.  跟踪宇宙的哪个子集发生了有趣的事情,并根据需要,扩展此区域. 在最坏的情况下,这种扩展是无限制的,实现将变得越来越慢,最终耗尽内存.

2.  创建固定大小的 Universe，边缘上的单元格具有较少的邻居
    比中间的细胞。 这种方法的缺点是无限
    像滑翔机一样到达宇宙尽头的模式被扼杀了。

3.  创建一个固定大小的周期性 Universe `移动`,其中边缘上的单元格具有环绕到 Universe 另一侧的邻居. 因为邻居环绕宇宙的边缘,滑翔机可以永远运行.

我们将实施`第三种`选择.

### 连接 Rust 和 JavaScript

> ⚡ 这是理解和从本教程中你获取的最重要的概念之一!

JavaScript 的垃圾收集堆 - `Object`,`Array`和 DOM 节点 被分配 - 不同于 WebAssembly 的线性内存空间,我们的 Rust 值 存在于其中. WebAssembly 目前无法直接访问垃圾收集堆 (截至 2018 年 4 月,预计会随之改变["主机绑定 host-bindings"提案][host-bindings]) . 另一方面,JavaScript 可以读取和写入 WebAssembly 线性存储空间,但仅作为一个[`ArrayBuffer`][array-buf]标量值 (`u8`,`i32`,`f64`等等......) WebAssembly 函数也接受,并返回标量值. 这些是构成 WebAssembly 和 JavaScript 通信 的所有构建块.

[host-bindings]: https://github.com/WebAssembly/host-bindings/blob/master/proposals/host-bindings/Overview.md
[array-buf]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/ArrayBuffer

`wasm_bindgen`定义了如何在这个边界上,使用复合结构的共同理解. 它涉及装箱 Rust 结构,将指针包装在 JavaScript 类 中以实现可用性,或者 索引到 Rust 中的 JavaScript 对象表. `wasm_bindgen`非常方便,但它不需要考虑我们的数据表示,以及跨越这个边界传递什么值和结构. 相反,将其视为实现您选择的接口设计的工具.

在设计 WebAssembly 和 JavaScript 之间的接口时,我们希望针对以下属性进行优化:

1.  **最小化复制 WebAssembly 线性存储器的 进/出 .**不必要的副本会产生不必要的开销.

2.  **最小化序列化和反序列化.**与副本类似,序列化和反序列化也会产生开销,并且通常也会进行复制. 如果我们可以将不透明控制传递给数据结构 - 而不是一侧序列化后,将其复制到 WebAssembly 线性存储器中的某个已知位置,并在另一侧进行反序列化 - 我们通常可以减少大量开销. `wasm_bindgen`帮助我们 定义和使用 JavaScript 的不透明控制`Object`或 盒装 Rust 结构.

作为一般的经验法则,一个好的 JavaScript↔WebAssembly 接口设计通常是将大型,长寿命的数据结构实现为 生活在 WebAssembly 线性内存 中的 Rust 类型,并作为不透明控制暴露给 JavaScript. JavaScript 调用导出的 WebAssembly 函数,这些函数接受这些不透明的控制,转换数据,执行繁重的计算,查询数据,最终返回一个小的可复制结果. 通过仅返回计算的小结果,我们避免在 JavaScript 垃圾收集堆和 WebAssembly 线性存储器 之间,来回复制和序列化所有内容.

### 在我们的生命游戏中连接 Rust 和 JavaScript

让我们首先列举一些要避免的危险. 我们不希望在每个`tick`上将整个 Universe 复制到 WebAssembly 线性内存 中. 我们不希望为宇宙中的每个单元分配对象,也不想强加函数调用来读写每个单元.

这给我们留下了什么? 我们可以将 Universe 表示为位于 WebAssembly 线性内存中的平面数组,并且每个单元格都有一个字节. `0`是一个死细胞,`1`是一个活细胞.

以下是 4 x 4 宇宙在内存中的样子:

![Screenshot of a 4 by 4 universe](./images/game-of-life/universe.png)

要在 Universe 中的给定行和列中查找单元格的数组索引,我们可以使用以下公式:

```text
index(row, column, universe) = row * width(universe) + column
```

我们有几种方法可以将 Universe 的单元格暴露给 JavaScript. 首先,我们添加实施[`std::fmt::Display`][`display`]到`Universe`,我们可以用来展示 一个 Rust`String` 呈现为文本字符的单元格. 然后将此 Rust String 从 WebAssembly 线性内存 复制到 JavaScript 的垃圾回收堆中 的 JavaScript String 中,然后通过设置`HTML`显示`textContent`. 在本章的后面,我们将推演这个实现,以避免在堆之间复制 Universe 的单元格,再渲染`<canvas>`.

_另一个可行的设计替代方案是 Rust 返回每个滴答后,更改状态的每个单元格的列表,而不是将整个 Universe 暴露给 JavaScript. 这样,JavaScript 在渲染时不需要遍历整个 Universe ,只需要相关的子集. 权衡的问题是这种 基于 delta 的设计实施起来稍微困难一些._

## Rust 实现

在上一章中,我们克隆了一个初始项目模板. 我们现在将修改该项目模板.

让我们开始删除 `alert` import 和`greet` 函数 - `src/lib.rs`, 并用单元格的类型定义替换它们:

```rust
#[repr(u8)]
#[derive(Clone, Copy, Debug, PartialEq, Eq)]
pub enum Cell {
    Dead = 0,
    Alive = 1,
}
```

重要的是我们拥有`#[repr(u8)]`,以便每个单元格表示为单个字节. 同样重要的是`Dead`代表`0`,那个`Alive`是`1`,这样我们就可以轻松地计算一个细胞的活体邻居.

接下来,让我们定义宇宙. 宇宙具有宽度和高度,以及长度为单元格的向量`width * height`.

```rust
#[wasm_bindgen]
pub struct Universe {
    width: u32,
    height: u32,
    cells: Vec<Cell>,
}
```

要访问给定行和列的单元格,我们将 行和列 转换为 单元格向量 的索引,如前所述:

```rust
impl Universe {
    fn get_index(&self, row: u32, column: u32) -> usize {
        (row * self.width + column) as usize
    }

    // ...
}
```

为了计算单元格的下一个状态,我们需要计算 其邻居有多少 是活着的. 我们来写一个`live_neighbor_count`做到这一点的方法!

```rust
impl Universe {
    // ...

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
}
```

该`live_neighbor_count`方法使用 增量和模数来 避免 宇宙的边缘情况. 当 delta 应用 `-1`,我们*添加*
`self.height - 1`让 模数 做它的事,而不是试图减去`1`. `row`和`column`可以为`0`,如果我们试图减去`1`, 从他们来看,会有一个 无符号整数 下溢.

现在我们拥有了从当前计算下一代所需的一切! 每个游戏的规则遵循 直接转换条件用`match`表达. 另外,因为我们希望 JavaScript 控制滴答时间,我们将把这个方法放在一个`#[wasm_bindgen]`注释下,以便它暴露给 JavaScript.

```rust
/// Public methods, exported to JavaScript.
#[wasm_bindgen]
impl Universe {
    pub fn tick(&mut self) {
        let mut next = self.cells.clone();

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

        self.cells = next;
    }

    // ...
}
```

到目前为止,宇宙的状态被表示为 细胞 的载体. 为了使这个可读,让我们实现一个基本的文本渲染器. 我们的想法是逐行将 Universe 写成文本,对于每个活着的单元格,打印 Unicode 字符`◼️` ("黑色方格") . 对于死细胞,我们将打印`◻️` ("白色方格") .

通过实施[`Display`]是来自 Rust 标准库 的特性,我们可以添加一种以 面向用户的方式 格式化结构的方法. 这也会自动给我们一个[`to_string`]方法.

[`display`]: https://doc.rust-lang.org/1.25.0/std/fmt/trait.Display.html
[`to_string`]: https://doc.rust-lang.org/1.25.0/std/string/trait.ToString.html

```rust
use std::fmt;

impl fmt::Display for Universe {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        for line in self.cells.as_slice().chunks(self.width as usize) {
            for &cell in line {
                let symbol = if cell == Cell::Dead { "◻️" } else { "◼️" };
                write!(f, "{}", symbol)?;
            }
            write!(f, "\n")?;
        }

        Ok(())
    }
}
```

最后,我们定义一个构造函数,用一个有趣的 活细胞和死细胞 模式来初始化宇宙,以及`render`方法:

```rust
/// Public methods, exported to JavaScript.
#[wasm_bindgen]
impl Universe {
    // ...

    pub fn new() -> Universe {
        let width = 64;
        let height = 64;

        let cells = (0..width * height)
            .map(|i| {
                if i % 2 == 0 || i % 7 == 0 {
                    Cell::Alive
                } else {
                    Cell::Dead
                }
            })
            .collect();

        Universe {
            width,
            height,
            cells,
        }
    }

    pub fn render(&self) -> String {
        self.to_string()
    }
}
```

有了这个,我们的生命游戏 Rust 实现的一半就完成了!

## 使用 JavaScript 渲染

首先,让我们添加一个`<pre>`HTML 用于渲染:

```html
<body>
    <pre id="game-of-life-canvas"></pre>
    <script src='./bootstrap.js'></script>
</body>
```

另外,我们想要的`<pre>`以网页中间为中心. 我们可以使用 CSS flex 来完成这项任务. 添加以下内容`<style>`在`index.html`的`<head>`里面:

```html
<style>
    body {
        width: 100%;
        height: 100%;
        display: flex;
        align-items: center;
        justify-content: center;
    }
</style>
```

在顶端`index.js`,让我们修复我们的导入来引入`Universe`而不是旧的`greet`功能:

```js
import {Universe} from './wasm_game_of_life';
```

另外,让我们获取`<pre>`并实例化新 Universe 的元素:

```js
const pre = document.getElementById('game-of-life-canvas');
const universe = Universe.new();
```

JavaScript 运行于`requestAnimationFrame`循环. 在每次迭代中,它将当前的 Universe 绘制到`<pre>`,然后运行`Universe::tick`.

```js
const renderLoop = () => {
  pre.textContent = universe.render();
  universe.tick();

  requestAnimationFrame(renderLoop);
};
```

要开始渲染过程,我们所要做的就是为渲染循环的第一次迭代进行初始调用:

```js
requestAnimationFrame(renderLoop);
```

这就是它现在的样子:

[![Screenshot of the Game of Life implementation with text rendering](./images/game-of-life/initial-game-of-life-pre.png)](./images/game-of-life/initial-game-of-life-pre.png)

## 直接从内存渲染到画布

在 Rust 中生成 (和分配) 一个`String`, 然后有`wasm-bindgen`将其转换为有效的 JavaScript 字符串 ,来会生成 Universe 单元格 的不必要副本. 其实在 JavaScript 代码 知道 Universe 的宽度和高度,并且可以直接从 JavaScript 中读取 WebAssembly 线性内存 中的单元格字节, 我们就可以修改`render`方法,用来返回 单元数组的开头指针.

还有,我们将切换到使用[Canvas API]. 而不是渲染 unicode 文本. 我们将在本教程的其余部分中使用此设计.

[canvas api]: https://developer.mozilla.org/en-US/docs/Web/API/Canvas_API

首先,让我们替换`pre`,换成了一个`<canvas>` (它也应该在`<body>`, `<script>`加载我们的 JavaScript 之前) :

```html
<body>
    <canvas id="game-of-life-canvas"></canvas>
    <script src='./bootstrap.js'></script>
</body>
```

为了从 Rust 实现 中获取必要的信息,我们需要为 Universe 的宽度,高度和指向 其单元数组 的指针 添加一些更多的 `getter`函数. 所有这些都暴露在 JavaScript 中.

```rust
/// Public methods, exported to JavaScript.
#[wasm_bindgen]
impl Universe {
    // ...

    pub fn width(&self) -> u32 {
        self.width
    }

    pub fn height(&self) -> u32 {
        self.height
    }

    pub fn cells(&self) -> *const Cell {
        self.cells.as_ptr()
    }
}
```

接下来,让我们在`import`代码行下面定义 JavaScript 在渲染画布时将使用的一些常量:

```js
import {Universe} from './wasm_game_of_life';

const CELL_SIZE = 5; // px
const GRID_COLOR = '#CCCCCC';
const DEAD_COLOR = '#FFFFFF';
const ALIVE_COLOR = '#000000';

// 这些必须匹配 `Cell::Alive` 和 `Cell::Dead` in `src/lib.rs`.
const DEAD = 0;
const ALIVE = 1;
```

现在,让我们重写当前的 JS 代码 (导入除外) ,不再写入`<pre>`而是专注在`<canvas>`:

```js
// 构造 the universe, and get its width and height.
const universe = Universe.new();
const width = universe.width();
const height = universe.height();

// Give the canvas room for all of our cells and a 1px border
// around each of them.
const canvas = document.getElementById('game-of-life-canvas');
canvas.height = (CELL_SIZE + 1) * height + 1;
canvas.width = (CELL_SIZE + 1) * width + 1;

const ctx = canvas.getContext('2d');

const renderLoop = () => {
  universe.tick();

  drawGrid();
  drawCells();

  requestAnimationFrame(renderLoop);
};
```

为了在单元格之间绘制网格,我们绘制 一组等间隔 的 水平线 和 一组等间距 的 垂直线. 这些线 纵横交错 形成网格.

```js
const drawGrid = () => {
  ctx.beginPath();
  ctx.lineWidth = 1 / window.devicePixelRatio;
  ctx.strokeStyle = GRID_COLOR;

  // Vertical lines.
  for (let i = 0; i <= width; i++) {
    ctx.moveTo(i * (CELL_SIZE + 1) + 1, 0);
    ctx.lineTo(i * (CELL_SIZE + 1) + 1, (CELL_SIZE + 1) * height + 1);
  }

  // Horizontal lines.
  for (let j = 0; j <= height; j++) {
    ctx.moveTo(0, j * (CELL_SIZE + 1) + 1);
    ctx.lineTo((CELL_SIZE + 1) * width + 1, j * (CELL_SIZE + 1) + 1);
  }

  ctx.stroke();
};
```

我们可以直接通过`memory`拿到 WebAssembly 的 线性内存, 而这个`memory`有原生 wasm 模块`wasm_game_of_life_bg`提供.为了绘制 细胞,我们从 `universe's cells` 拿到指针 ,构造一个覆盖细胞面积的`Uint8Array`缓冲区,迭代每个细胞,并分别根据 细胞是死还是活,绘制白色或黑色矩形. 通过使用 `idx` 和 覆盖,我们避免在每个`tick`上跨越边界复制单元格.

```js
// Import the WebAssembly memory at the top of the file.
import {memory} from './wasm_game_of_life_bg';

// ...

const getIndex = (row, column) => {
  return row * width + column;
};

const drawCells = () => {
  const cellsPtr = universe.cells(); // < universe's cells
  const cells = new Uint8Array(memory.buffer, cellsPtr, width * height);

  ctx.beginPath();

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

  ctx.stroke();
};
```

要开始渲染过程,我们将使用与 上部分相同的代码 来开始渲染循环的第一次迭代:

```js
requestAnimationFrame(renderLoop);
```

## 它工作了!

重建 WebAssembly 和 绑定 :

    npm run build-debug

确保您的开发服务器仍在运行. 如果不是,请再次启动:

    npm run serve

如果你刷新[http://localhost:8080/](http://localhost:8080/),你应该受到令人兴奋的展示!

[![Screenshot of the Game of Life implementation](./images/game-of-life/initial-game-of-life.png)](./images/game-of-life/initial-game-of-life.png)

您可以 checkout `chapter-one` branch 找到完整代码.

还有一个非常巧妙的算法,来实现生命游戏[hashlife](https://en.wikipedia.org/wiki/Hashlife). 它使用积极的记忆,实际上可以获得*指数级更快*计算后代的时间越长! 鉴于此,您可能想知道为什么我们在本教程中没有实现`hashlife`. 因为它超出了本文的范围,我们专注于 Rust 和 WebAssembly 集成,但我们强烈建议您自己去了解`hashlife`!

## 练习

- 使用单个太空飞船,初始化宇宙.

- 不是硬编码最初的宇宙,而是生成一个随机的,有五十五个细胞活着或死亡的机会.

  _提示: 使用`wasm_bindgen` import `Math.random` JavaScript 函数:_

  ```rust
  #[wasm_bindgen]
  extern {
      #[wasm_bindgen(js_namespace = Math)]
      fn random() -> f64;
  }
  ```

- 用 一个字节 表示 每个单元格 可以很容易地迭代单元格,但这是以浪费内存为代价的. 每个字节是 8 位,但我们只需要 一个位 来表示每个单元 是活还是死. 重构数据表示,以便每个单元,仅使用一个空格位.
