# How to Add WebAssembly Support to a General-Purpose Crate

本节适用于希望支持WebAssembly的通用包装箱作者.

## Maybe Your Crate Already Supports WebAssembly!

查看有关的信息[what kinds of things can make a general-purpose
crate *not* portable for WebAssembly](./which-crates-work-with-wasm.html).如果您的箱子没有任何这些东西,它可能已经支持WebAssembly!

您可以随时通过运行进行检查`cargo build`对于WebAssembly目标:

```
cargo build --target wasm32-unknown-unknown
```

如果该命令失败,那么您的crate现在不支持WebAssembly.如果它没有失败,那么你的箱子*威力*支持WebAssembly.你可以100%确定它(并继续这样做!)[adding tests for wasm and
running those tests in CI.](#maintaining-ongoing-support-for-webassembly)

## Adding Support for WebAssembly

### Avoid Performing I/O Directly

在Web上,I / O始终是异步的,并且没有文件系统.从库中分解因子I / O,让用户执行I / O,然后将输入切片传递到库中.

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

### Add `wasm-bindgen` as a Dependency

如果您需要与外界进行互动(即您不能让图书馆的消费者为您推动这种互动),那么您需要添加`wasm-bindgen`(和`js-sys`和`web-sys`如果你需要它们)作为编译目标WebAssembly的依赖项:

```toml
[target.'cfg(target_arch = "wasm32")'.dependencies]
wasm-bindgen = "0.2"
js-sys = "0.3"
web-sys = "0.3"
```

### Avoid Synchronous I/O

如果必须在库中执行I / O,则它不能同步.Web上只有异步I / O.使用[the `futures`
crate](https://crates.io/crates/futures)和[the `wasm-bindgen-futures`
crate](https://rustwasm.github.io/wasm-bindgen/api/wasm_bindgen_futures/)管理异步I / O.如果您的库函数在某些未来类型中是通用的`F`那么未来可以通过实现`fetch`在Web上或通过操作系统提供的非阻塞I / O.

```rust
pub fn do_stuff<F>(future: F) -> impl Future<Item = MyOtherThing>
where
    F: Future<Item = MyThing>,
{
    // ...
}
```

您还可以为WebAssembly和Web以及本机目标定义特征并实现它:

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

### Avoid Spawning Threads

Wasm还不支持线程(但是[experimental work is
ongoing](https://rustwasm.github.io/2018/10/24/multithreading-rust-and-wasm.html)),所以试图在wasm中产生线程会引起恐慌.

您可以使用`#[cfg(..)]`■启用线程和非线程代码路径,具体取决于目标是否为WebAssembly:

```rust
#![cfg(target_arch = "wasm32")]
fn do_work() {
    // Do work with only this thread...
}

#![cfg(not(target_arch = "wasm32"))]
fn do_work() {
    use std::thread;

    // Spread work to helper threads....
    thread::spawn(|| {
        // ...
    });
}
```

另一个选择是从库中分解线程产生,并允许用户"自带线程"类似于分解文件I / O并允许用户自带I / O.这具有与想要拥有自己的自定义线程池的应用程序一起玩的副作用.

## Maintaining Ongoing Support for WebAssembly

### Building for `wasm32-unknown-unknown` in CI

通过让CI脚本运行以下命令,确保在定位WebAssembly时编译不会失败:

```
rustup target add wasm32-unknown-unknown
cargo check --target wasm32-unknown-unknown
```

例如,您可以将此添加到您的`.travis.yml`Travis CI的配置:

```yaml
matrix:
  include:
    - language: rust
      rust: stable
      name: "check wasm32 support"
      install: rustup target add wasm32-unknown-unknown
      script: cargo check --target wasm32-unknown-unknown
```

### Testing in Node.js and Headless Browsers

您可以使用`wasm-bindgen-test`和`wasm-pack test`子命令在Node.js或无头浏览器中运行wasm测试.您甚至可以将这些测试集成到CI中.

[Learn more about testing wasm
here.](https://rustwasm.github.io/wasm-bindgen/wasm-bindgen-test/index.html)
