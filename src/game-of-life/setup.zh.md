
# 安装

这个章节讲述的是: 怎样设置将Rust编译成WebAssembly和糅合到JavaScript的工具链

### Rust工具链

您将需要标准的 Rust工具链,包括`rustup`,`rustc`,和`cargo`

[按照以下说明安装Rust工具链. ][rust-install]

[rust-install]: https://www.rust-lang.org/en-US/install.html

## `wasm32-unknown-unknown`目标

一旦安装了 Rust工具链 ,您就可以将 Rust程序 编译为 WebAssembly,而不是机器的本机代码. 您可以通过添加`wasm32-unknown-unknown`来启用此功能,使用以下命令进行目标: 

    rustup update
    rustup install nightly
    rustup target add wasm32-unknown-unknown --toolchain nightly

## `npm`

`npm`是 JavaScript的包管理器. 我们将使用它来安装和运行 JavaScript捆绑器 和 开发服务器. 在本教程结束时,我们将发布我们编译的`.wasm`到了`npm`注册表中. 

[请按照以下说明进行安装`npm`. ][npm-install]

[npm-install]: https://www.npmjs.com/get-npm

## `wasm-bindgen`

[`wasm-bindgen`][wb]为 Rust和WebAssembly 生成与 JavaScript 的双向绑定. 

安装`wasm-bindgen`使用此命令: 

    cargo +nightly install wasm-bindgen-cli

[wb]: https://github.com/rustwasm/wasm-bindgen