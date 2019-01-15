## 你好, 世界!

这个章节讲述的是如何构建与运行你的第一个 Rust 和 WebAssembly 程序: 可运行 `alert('Hello, World!')` 函数的网页


## 克隆项目模板

项目模板包含一个"hello world"程序. 它预先配置了默认的默认设置,因此您可以快速构建,集成和打包Web代码. 

克隆此教程代码存储库,输入其目录,然后 checkout `chapter-zero` branch: 

```text
git clone https://github.com/rustwasm/wasm_game_of_life.git
cd ./wasm_game_of_life
git checkout -b chapter-zero origin/chapter-zero
```

## 里面有什么

让我们来看看我们项目的内容: 

```text
.
├── bootstrap.js
├── Cargo.lock
├── Cargo.toml
├── index.html
├── index.js
├── package.json
├── package-lock.json
├── src
│   └── lib.rs
└── webpack.config.js
```

其中大多数是配置文件,但我们应该突出显示一些文件. 

### `index.html`

这是网页的根HTML文件. 它除了加载`bootstrap.js`之外没有其他作用,这是一个非常薄的`index.js`包装. 

```html
<html>
    <head>
        <meta content="text/html;charset=utf-8" http-equiv="Content-Type"/>
    </head>
    <body>
        <script src='./bootstrap.js'></script>
    </body>
</html>
```

### `index.js`

该`index.js`是我们网页的 JavaScript 的主要入口点. 它导入项目的WebAssembly模块,并调用模块`greet`函数. 

```js
import { greet } from "./wasm_game_of_life";

greet("Rust and WebAssembly");
```

### `src/lib.rs`

该`src/lib.rs`文件是我们正在编译到 WebAssembly的Rust包 的根. 它用`wasm_bindgen`与 JavaScript交互. 它导入了`window.alert` - JavaScript函数,并导出`greet`rust函数,函数需要一个`name`参数以便 传递给`alert`函数使用. 

```rust
#![feature(use_extern_macros)]

extern crate wasm_bindgen;

use wasm_bindgen::prelude::*;

#[wasm_bindgen]
extern {
    fn alert(s: &str);
}

#[wasm_bindgen]
pub fn greet(name: &str) {
    alert(&format!("Hello, {}!", name));
}
```

## 构建与启动服务器

首先,确保为此项目安装了JavaScript构建依赖项: 

```text
npm install
```

此命令只需运行一次,并将安装`webpack` - JavaScript bundler及其开发服务器. 注意:使用`webpack`对 Rust和WebAssembly 来说不是必需的,它只是我们为方便起见而选择的捆绑器和开发服务器. 

构建 Rust crate{📦} 为 WebAssembly 并生成`wasm_bindgen`胶水,运行此命令: 

```text
npm run build-debug
```

第一个构建可能需要一些时间,因为需要编译依赖项. 但不要担心: 后续构建,当依赖关系不需要重新编译时,将会快得多. 

命令会创建 Rust crates的"debug"版本: 未优化应用的构建,并包含符号以便在浏览器的开发人员工具中进行更好的调试. 您还可以创建一个"发布-release"版本,该版本有优化应用的过程: 

    npm run build-release

这是我们想要用来创建, 用于分析和部署到生产的`.wasm`二进制文件的命令. 

接下来,为开发服务器打开一个新终端. 在新终端中运行服务器让我们让它在后台运行,并且不会阻止我们在此期间运行其他命令. 在新终端中,运行以下命令: 

    npm run serve

浏览Web浏览器[http://localhost:8080/)](http://localhost:8080/)你应该收到一条 `alert` 信息: 

[![Screenshot of the "Hello, Rust and WebAssembly!" Web page alert](./images/game-of-life/setup.png)](./images/game-of-life/setup.png)

任何时候你做出改变,并希望它们反映到[http://localhost:8080/)](http://localhost:8080/),只是重新运行`npm run
build-debug`命令. 

## 练习

-   修改`index.js`用你的名字而不是"Rust和WebAssembly"来 运行 `alert`函数. 