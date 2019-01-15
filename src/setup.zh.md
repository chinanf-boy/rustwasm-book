# 来我们开始安装吧

如果你想使用`Rust for wasm`，那么你需要一个能够做到这一点的环境! 如果您尚未安装，则需要安装[rustup][rustup] (官方工具) ，以便安装和管理 Rust 编译器的不同版本。 按照站点上的说明将其安装到您的计算机上。 目前，在与 wasm 合作时，你需要最新「nightly」的 Rust:

```bash
$ rustup default nightly
```

一旦安装完毕，你就需要得到`wasm32-unknown-unknown`工具链。

```bash
$ rustup target add wasm32-unknown-unknown --toolchain nightly
```

接下来，如果你有兴趣制作小型的 wasm 二进制文件，你需要安装[wasm-gc][wasm-gc]，用于制作较小的二进制文件， 并在编译器工具链中解决 bug 的工具:

```bash
$ cargo install wasm-gc
```

最后，如果你是*真*有兴趣制作你想要安装的小型 wasm 二进制文件， `wasm-opt`来自[binaryen 工具包][binaryen]符合你。

[rustup]: https://www.rustup.rs/
[binaryen]: https://github.com/WebAssembly/binaryen
[wasm-gc]: https://github.com/alexcrichton/wasm-gc
