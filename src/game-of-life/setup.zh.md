# 安装

这个章节讲述的是: 怎样设置将 Rust 编译成 WebAssembly 和糅合到 JavaScript 的工具链

## Rust 工具链

您将需要标准的 Rust 工具链,包括`rustup`,`rustc`,和`cargo`

[按照以下说明安装 Rust 工具链. ][rust-install]

Rust 和 WebAssembly 的使用在 Rust 发布火车上稳定运行! 这意味着我们不需要任何实验性功能标志。但是,我们确实需要 Rust 1.30 或打上版本。

## `wasm-pack`

`wasm-pack`是您构建,测试和发布 Rust 生成的 WebAssembly 的一站式商店.

[得到`wasm-pack`这里!][wasm-pack-install]

## `cargo-generate`

[`cargo-generate`通过利用预先存在的 git 存储库作为模板，帮助您快速启动并运行新的 Rust 项目。][cargo-generate]

安装`cargo-generate`使用此命令:

```
cargo install cargo-generate
```

## `npm`

`npm`是 JavaScript 的包管理器. 我们将使用它来安装和运行 JavaScript 捆绑器 和 开发服务器. 在本教程结束时,我们将发布我们编译的`.wasm`到了`npm`注册表中.

[请按照以下说明进行安装`npm`. ][npm-install]

如果你已经拥有`npm`安装,使用此命令确保它是最新的:

```
npm install npm@latest -g
```

[rust-install]: https://www.rust-lang.org/tools/install
[npm-install]: https://www.npmjs.com/get-npm
[wasm-pack]: https://github.com/rustwasm/wasm-pack
[cargo-generate]: https://github.com/ashleygwilliams/cargo-generate
[wasm-pack-install]: https://rustwasm.github.io/wasm-pack/installer/
