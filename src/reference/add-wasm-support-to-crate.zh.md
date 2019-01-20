# 怎么为 一个 通用箱 添加 WebAssembly 支持

本节适用于，希望支持 WebAssembly 的通用箱作者。

## 也许，你的 箱子 以及 支持 WebAssembly!

查看有关[什么样的事情能让一个通用箱 _不能_ 移植到 WebAssembly](./which-crates-work-with-wasm.zh.html)的信息。如果您的箱子没有任何这些东西，它可能已经支持 WebAssembly!

您可以随时通过运行`cargo build`，检查 WebAssembly 目标:

```
cargo build --target wasm32-unknown-unknown
```

如果该命令失败，那么您的 crate 现在不支持 WebAssembly。如果它没有失败，那么你的箱子*不一定*支持 WebAssembly。但你可以 100% 确定它(继续跟着做!)[为 wasm 添加测试 ，还有在 CI 上运行这些测试。](#maintaining-ongoing-support-for-webassembly)

## 添加 WebAssembly 支持

### 避免 直接执行 I/O

在 Web 上，I/O 始终是异步的，且时没有文件系统。从库中分出 I/O 因素吧，让用户以为自己在执行 I/O，但其实将输入 slice 传递到库中。

例如,重构这个:

```rust
use std::fs;
use std::path::Path;

pub fn parse_thing(path: &Path) -> Result<MyThing, MyError> {
    let contents = fs::read(path)?;
    // ...
}
```

进入这个:

```rust
pub fn parse_thing(contents: &[u8]) -> Result<MyThing, MyError> {
    // ...
}
```

### 添加 `wasm-bindgen` 作为依赖

如果您需要与外界进行互动(即，您不能让 库 的使用者为您推动这种互动)，那么您需要添加`wasm-bindgen`(还有`js-sys`和`web-sys`，若你需要它们)作为编译 WebAssembly 目标的依赖项:

```toml
[target.'cfg(target_arch = "wasm32")'.dependencies]
wasm-bindgen = "0.2"
js-sys = "0.3"
web-sys = "0.3"
```

### 避免 同步 I/O

如果你必须在库中执行 I/O，那它不能同步。Web 上只有异步 I/O。使用[`futures`箱](https://crates.io/crates/futures)和[`wasm-bindgen-futures`箱](https://rustwasm.github.io/wasm-bindgen/api/wasm_bindgen_futures/)管理异步 I/O。如果您的库函数为某些 future 类型`F`的通用函数，那么 future 是可以实现的，可能是通过在 Web 上`fetch`，或是通过操作系统提供的非阻塞 I/O。

```rust
pub fn do_stuff<F>(future: F) -> impl Future<Item = MyOtherThing>
where
    F: Future<Item = MyThing>,
{
    // ...
}
```

您还可以为 WebAssembly 和 Web 以及本机目标，定义一个 trait 并实现它:

```rust
trait ReadMyThing {
    type F: Future<Item = MyThing>;
    fn read(&self) -> Self::F;
}

#[cfg(target_arch = "wasm32")]
struct WebReadMyThing {
    // ...
}

#[cfg(target_arch = "wasm32")]
impl ReadMyThing for WebReadMyThing {
    // ...
}

#[cfg(not(target_arch = "wasm32"))]
struct NativeReadMyThing {
    // ...
}

#[cfg(not(target_arch = "wasm32"))]
impl ReadMyThing for NativeReadMyThing {
    // ...
}
```

### 避免 派生线程

Wasm 还不支持线程(但是[实验型工作还是在继续](https://rustwasm.github.io/2018/10/24/multithreading-rust-and-wasm.html))，所以试图在 wasm 中产生线程会引起恐慌。

您可以使用`#[cfg(..)]`启用代码路径，是不是线程和非线程，具体取决于目标是否为 WebAssembly:

```rust
#![cfg(target_arch = "wasm32")]
fn do_work() {
    // 只使用这个线程…
}

#![cfg(not(target_arch = "wasm32"))]
fn do_work() {
    use std::thread;

    // 将工作扩展到助手线程….
    thread::spawn(|| {
        // ...
    });
}
```

另一种选择是将从库中生成的线程分离出来，并允许用户带上“自己的线程”，类似于将文件 I/O 分离出来，并允许用户带上自己的 I/O。这有一个副作用，就是对想要拥有自己自定义线程池的应用程序要进行友好处理。

## Maintaining Ongoing Support for WebAssembly

> 维护对 WebAssembly 的持续支持

### 在 CI 上，构建成 `wasm32-unknown-unknown`

通过让 CI 脚本运行以下命令，确保为 WebAssembly 目标时编译不会失败:

```
rustup target add wasm32-unknown-unknown
cargo check --target wasm32-unknown-unknown
```

例如,您可以将此 Travis CI 的配置添加到您的`.travis.yml`:

```yaml
matrix:
  include:
    - language: rust
      rust: stable
      name: 'check wasm32 support'
      install: rustup target add wasm32-unknown-unknown
      script: cargo check --target wasm32-unknown-unknown
```

### 在 Node.js 和 Headless 浏览器 上测试

您可以在 Node.js 或无头浏览器中，使用`wasm-bindgen-test`和`wasm-pack test`子命令，运行 wasm 测试。您甚至可以将这些测试集成到 CI 中。

[学习 ，更多 测试 wasm 的资源](https://rustwasm.github.io/wasm-bindgen/wasm-bindgen-test/index.html)
