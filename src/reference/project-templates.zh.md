# Project Templates

Rust和WebAssembly工作组负责管理和维护各种项目模板,以帮助您启动新项目并开始运行.

## `wasm-pack-template`

[这个模板][wasm-pack-template]用于启动要与之一起使用的Rust和WebAssembly项目[`wasm-pack`][wasm-pack].

使用`cargo generate`克隆此项目模板:

```
cargo install cargo-generate
cargo generate --git https://github.com/rustwasm/wasm-pack-template.git
```

## `create-wasm-app`

[这个模板][create-wasm-app]适用于使用从Rust创建的npm包的JavaScript项目[`wasm-pack`][wasm-pack].

使用它`npm init`:

```
mkdir my-project
cd my-project/
npm init wasm-app
```

此模板通常与其一起使用`wasm-pack-template`,哪里`wasm-pack-template`项目在本地安装`npm link`,并作为一个依赖拉入`create-wasm-app`项目.

## `rust-webpack-template`

[这个模板][rust-webpack-template]预先配置了所有样板文件,用于将Rust编译为WebAssembly并将其直接挂钩到Webpack的Webpack构建管道中[`rust-loader`][rust-loader].

使用它`npm init`:

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
