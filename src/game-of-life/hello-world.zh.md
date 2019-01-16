# 你好,世界!

本节将向您展示如何构建和运行您的第一个 Rust 和 WebAssembly 程序:一个 alert "Hello,World!"的网页。

在开始之前，确保你已经遵照了[安装说明](setup.zh.html)。

## 克隆项目模板

项目模板预先配置了默认值，因此您可以快速构建，集成和打包 Web 代码.

使用以下命令，克隆项目模板:

```text
cargo generate --git https://github.com/rustwasm/wasm-pack-template
```

这应该会提示您输入新项目的名称。我们会用**"wasm-game-of-life"**.

## 里面有什么

进入新的`wasm-game-of-life`项目，让我们来看看它的内容:

```text
wasm-game-of-life/
├── Cargo.toml
├── LICENSE_APACHE
├── LICENSE_MIT
├── README.md
└── src
    ├── lib.rs
    └── utils.rs
```

我们来详细介绍几个这样的文件.

### `wasm-game-of-life/Cargo.toml`

该`Cargo.toml`文件是`cargo`的 指定依赖项和元数据，Cargo 则是 Rust 的包管理器和构建工具。这个预先配置了一个`wasm-bindgen`依赖项，我们将在后面深入研究一些可选的依赖项，和正确初始化`crate-type`以生成`.wasm`库。

### `wasm-game-of-life/src/lib.rs`

该`src/lib.rs`文件 Rust 包的根，我们要编译成 WebAssembly 的。它用`wasm-bindgen`与 JavaScript 交互。它导入了`window.alert`的 JavaScript 函数，并导出为这个名为`greet`的 Rust 函数，用于警告(alert)一个问候(greet)消息。

```rust
extern crate cfg_if;
extern crate wasm_bindgen;

mod utils;

use cfg_if::cfg_if;
use wasm_bindgen::prelude::*;

cfg_if! {
    // 当 `wee_alloc` 特性启用, 使用 `wee_alloc` 作为全局分配器。
    if #[cfg(feature = "wee_alloc")] {
        extern crate wee_alloc;
        #[global_allocator]
        static ALLOC: wee_alloc::WeeAlloc = wee_alloc::WeeAlloc::INIT;
    }
}

#[wasm_bindgen]
extern {
    fn alert(s: &str);
}

#[wasm_bindgen]
pub fn greet() {
    alert("Hello, wasm-game-of-life!");
}
```

### `wasm-game-of-life/src/utils.rs`

该`src/utils.rs`模块提供了常用的实用函数，让 Rust 编译成 WebAssembly 来得更容易。我们将在本教程后面详细介绍其中一些实用函数，例如我们查看的内容[调试我们的 wasm 代码](debugging.zh.html),但我们现在可以先略过。

## 构建项目

我们用`wasm-pack`，是根据以下构建步骤:

- 确保我们有 Rust 1.30 或打上版本，通过`rustup`安装`wasm32-unknown-unknown`目标(target),
- 用`cargo`将我们的 Rust 的源 编译为 WebAssembly 的 `.wasm`二进制文件,
- 为 Rust 生成的 WebAssembly 使用`wasm-bindgen`，生成 JavaScript API.

要完成所有这些操作，请在项目目录中，运行此命令:

```
wasm-pack build
```

构建完成后，我们可以在`pkg`目录中，找到它的工件，它应该有这些内容:

```
pkg/
├── package.json
├── README.md
├── wasm_game_of_life_bg.wasm
├── wasm_game_of_life.d.ts
└── wasm_game_of_life.js
```

该`README.md`文件是从主项目复制的，但其他文件是新来的。

### `wasm-game-of-life/pkg/wasm_game_of_life_bg.wasm`

该`.wasm`文件 是 Rust 编译器从 Rust 源生成的 WebAssembly 二进制文件。其包含 wasm 编译版本(具有所有 Rust 函数和数据)。例如,它具有导出的"greet"函数.

### `wasm-game-of-life/pkg/wasm_game_of_life.js`

该`.js`文件是由`wasm-bindgen`生成的，并包含 JavaScript 胶水，其用于将 DOM 和 JavaScript 函数导入 Rust，并向 WebAssembly 函数公开一个很好的 API 到 JavaScript。例如，有一个 JavaScript 的`greet`函数，其包裹着从 WebAssembly 模块导出的`greet`函数。就现在而言，这胶水并没有做太多，但是当我们开始在 wasm 和 JavaScript 之间，来回传递更多有趣的值时，它将有助于将这些值传过边界。

```js
import * as wasm from './wasm_game_of_life_bg';

// ...

export function greet() {
  return wasm.greet();
}
```

### `wasm-game-of-life/pkg/wasm_game_of_life.d.ts`

该`.d.ts`文件包含给 JavaScript 胶水的[TypeScript][typescript]类型声明。如果您使用的是 TypeScript，可以检查对 WebAssembly 函数的调用类型，并且您的 IDE 可以提供自动完成和建议!如果您不使用 TypeScript，则可以安全地忽略此文件。

```typescript
export function greet(): void;
```

[typescript]: http://www.typescriptlang.org/

### `wasm-game-of-life/pkg/package.json`

[该`package.json`文件 包含有关生成的 JavaScript 和 WebAssembly 包的元数据。][package.json]npm 和 JavaScript 捆绑包使用它来确定包，包名，版本和一堆其他东西之间的依赖关系。它帮助我们与 JavaScript 工具集成，并允许我们将我们的包发布到 npm。

```json
{
  "name": "wasm-game-of-life",
  "collaborators": ["Your Name <your.email@example.com>"],
  "description": null,
  "version": "0.1.0",
  "license": null,
  "repository": null,
  "files": ["wasm_game_of_life_bg.wasm", "wasm_game_of_life.d.ts"],
  "main": "wasm_game_of_life.js",
  "types": "wasm_game_of_life.d.ts"
}
```

[package.json]: https://docs.npmjs.com/files/package.json

## 将其放入网页

拿我们的`wasm-game-of-life`去打包，并在网页中使用它，我们使用[该`create-wasm-app`JavaScript 项目模板][create-wasm-app].

[create-wasm-app]: https://github.com/rustwasm/create-wasm-app

在`wasm-game-of-life`目录中，运行此命令:

```
npm init wasm-app www
```

这就是我们新的`wasm-game-of-life/www`子目录，所包含的:

```
wasm-game-of-life/www/
├── bootstrap.js
├── index.html
├── index.js
├── LICENSE-APACHE
├── LICENSE-MIT
├── package.json
├── README.md
└── webpack.config.js
```

再一次,让我们仔细看看其中的一些文件。

<!-- here -->

### `wasm-game-of-life/www/package.json`

这个`package.json`预配置了`webpack`和`webpack-dev-server`依赖项,以及`hello-wasm-pack`依赖，这是已发布到 npm 的`wasm-pack-template`包初始版本。

### `wasm-game-of-life/www/webpack.config.js`

此文件配置 webpack 及其本地开发服务器。它是预配置的，你根本不需要调整它，就可以使用 webpack 及其本地开发服务器。

### `wasm-game-of-life/www/index.html`

这是网页的根 HTML 文件。它除了加载`bootstrap.js`之外，没有其他作用，就是装着`index.js`的薄膜。

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8" />
    <title>Hello wasm-pack!</title>
  </head>
  <body>
    <script src="./bootstrap.js"></script>
  </body>
</html>
```

### `wasm-game-of-life/www/index.js`

该`index.js`是我们网页的 JavaScript 的主要入口点。它导入了`hello-wasm-pack`npm 包，包含默认`wasm-pack-template`的 WebAssembly 编译 和 JavaScript 胶水，然后调用`hello-wasm-pack`的`greet`函数。

```js
import * as wasm from 'hello-wasm-pack';

wasm.greet();
```

### 安装依赖项

首先，通过运行在`wasm-game-of-life/www`子目录内`npm install`，确保安装本地开发服务器及其依赖项:

```text
npm install
```

此命令只需运行一次，将安装`webpack`JavaScript 捆绑器 及其开发服务器。

> 注意`webpack`，并不是 Rust 和 WebAssembly 的必需品,它只是我们为方便起见，而选择的捆绑器和开发服务器。Parcel 和 Rollup 也可以支持将 WebAssembly 导入为 ECMAScript 模块.

### 使用我们`www`中的本地`wasm-game-of-life`包

不是使用来自 npm 的`hello-wasm-pack`包，而是想使用我们的本地`wasm-game-of-life`包。这将使我们能够逐步开发我们的生命游戏程序。

首先，在`wasm-game-of-life/pkg`目录的里面，运行`npm link`，以便本地包可以被其他本地包依赖，而不需要将它们发布到 npm:

```bash
npm link
```

> 🐞 `npm link`后，你是否收到`EACCESS`或运行时的权限错误了吗?[请查看，如何使用`npm`来防止权限错误.](https://docs.npmjs.com/getting-started/fixing-npm-permissions)

第二，要使用来自`www`的，已`npm link`版本的`wasm-game-of-life`，通过在`wasm-game-of-life/www`中运行此命令:

```
npm link wasm-game-of-life
```

最后，修改`wasm-game-of-life/www/index.js`，去导入`wasm-game-of-life`而不是`hello-wasm-pack`包:

```js
import * as wasm from 'wasm-game-of-life';

wasm.greet();
```

我们的网页，现已准备好在本地供给!

## 本地供给

接下来，为开发服务器打开一个新终端。在新终端中运行服务器，我们让它在后台运行，并且不会阻止我们在此期间运行其他命令。在新终端中，从`wasm-game-of-life/www`目录内部运行此命令:

```
npm run start
```

浏览 Web 浏览器[http://localhost:8080/](http://localhost:8080/)你应该收到一条警告(alert)信息:

[![Screenshot of the "Hello， wasm-game-of-life!" Web page alert](../images/game-of-life/hello-world.png)](../images/game-of-life/hello-world.png)

任何时候你做出改变，并希望它们反映在[http://localhost:8080/](http://localhost:8080/),只是在`wasm-game-of-life`目录内，重新运行`wasm-pack build`命令。

## 演示

- 修改`wasm-game-of-life/src/lib.rs`中的`greet`函数，让其拿一个`name: &str`，这是可以自定义警报消息的参数，并在`wasm-game-of-life/www/index.js`内部，将您的名称传递给`greet`函数，让新版本发挥作用。用`wasm-pack build`，重建`.wasm`二进制，然后刷新[http://localhost:8080/](http://localhost:8080/)在您的 Web 浏览器中，您应该看到自定义的问候语!

  <details>
    <summary>Answer</summary>

  新版本`wasm-game-of-life/src/lib.rs`的`greet`函数:

  ```rust
  #[wasm_bindgen]
  pub fn greet(name: &str) {
      alert(&format!("Hello, {}!", name));
  }
  ```

  在`wasm-game-of-life/www/index.js`中新的`greet`调用:

  ```js
  wasm.greet('Your Name');
  ```

  </details>
