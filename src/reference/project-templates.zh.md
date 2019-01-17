# 项目模版

Rust 和 WebAssembly 工作组负责管理和维护各种项目模板，以帮助您启动新项目并开始运行。

## `wasm-pack-template`

[这个模板][wasm-pack-template]是与[`wasm-pack`][wasm-pack]合作启动的， Rust 和 WebAssembly 项目.

使用`cargo generate`克隆此项目模板:

```
cargo install cargo-generate
cargo generate --git https://github.com/rustwasm/wasm-pack-template.git
```

## `create-wasm-app`

[这个模板][create-wasm-app]是 Rust 与[`wasm-pack`][wasm-pack]孵化的， (npm 包) JavaScript 项目.

使用`npm init`:

```
mkdir my-project
cd my-project/
npm init wasm-app
```

此模板通常与`wasm-pack-template`一起使用，这里的`wasm-pack-template`项目是由`npm link`在本地安装，并作为一个依赖拉进一个`create-wasm-app`项目。

## `rust-webpack-template`

[这个模板][rust-webpack-template]预先配置了所有样板文件，用于将 Rust 编译为 WebAssembly ，并将其直接挂钩到 Webpack 的 [`rust-loader`][rust-loader]构建管道流程。

使用`npm init`:

```
mkdir my-project
cd my-project/
npm init rust-webpack
```

[wasm-pack]: https://github.com/rustwasm/wasm-pack
[wasm-pack-template]: https://github.com/rustwasm/wasm-pack-template
[create-wasm-app]: https://github.com/rustwasm/create-wasm-app
[rust-webpack-template]: https://github.com/rustwasm/rust-webpack-template
[rust-loader]: https://github.com/wasm-tool/rust-loader/
