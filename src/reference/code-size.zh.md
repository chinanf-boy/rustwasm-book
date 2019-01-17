# 收缩 `.wasm` 尺寸

本节将教您如何优化您的`.wasm`为小代码占用空间构建,以及如何识别更改 Rust 源的机会`.wasm`代码被发出.

## Why Care About Code Size?

服务时`.wasm`通过网络文件,它越小,客户端下载它的速度就越快.快点`.wasm`下载会导致更快的页面加载时间,从而使用户更快乐.

然而,重要的是要记住,虽然代码大小可能不是你感兴趣的最终所有指标,但更像是一些更模糊和难以衡量的东西,比如"第一次互动的时间".虽然代码大小在这个测量中起着很大的作用(如果你还没有所有的代码,也无法做任何事情!)它不是唯一的因素.

WebAssembly 通常为用户提供 gzip,因此您需要确保通过网络比较 gzip'd 大小的传输时间差异.还要记住,WebAssembly 二进制格式非常适合 gzip 压缩,通常可以减少 50%以上的大小.

此外,WebAssembly 的二进制格式经过优化,可以进行非常快速的解析和处理.浏览器现在拥有"基线编译器",它解析 WebAssembly 并以尽可能快的速度通过网络发出编译代码.这意味着[如果你正在使用`instantiateStreaming`][hacks]第二个 Web 请求完成后,WebAssembly 模块可能已准备就绪.另一方面,JavaScript 通常需要更长时间才能解析,但也可以通过 JIT 编译等方式加快速度.

最后,请记住,WebAssembly 也比 JavaScript 更优化执行速度.您需要确保测量 JavaScript 和 WebAssembly 之间的运行时比较,以考虑代码大小的重要性.

所有这一切要说,如果你的话,基本上不要沮丧`.wasm`文件大于预期!代码大小可能最终只是端到端故事中的众多因素之一.仅查看代码大小的 JavaScript 和 WebAssembly 之间的比较缺少树的林.

[hacks]: https://hacks.mozilla.org/2018/01/making-webassembly-even-faster-firefoxs-new-streaming-and-tiering-compiler/

## Optimizing Builds for Code Size

我们可以使用很多配置选项来获取`rustc`做小`.wasm`二进制文件.在某些情况下,我们的编译时间较长,较小`.wasm`大小.在其他情况下,我们以较小的代码大小交换 WebAssembly 的运行时速度.我们应该认识到每个选项的权衡,并且在我们交换代码大小,概况和度量的运行时速度以便做出关于交易是否值得的明智决策的情况下.

### 使用链接时间优化 (LTO) 进行编译

在`Cargo.toml`,添加`lto = true`在`[profile.release]`部分:

```toml
[profile.release]
lto = true
```

这为 LLVM 提供了更多内联和修剪功能的机会. 它不仅会成功`.wasm`更小,但它也会在运行时更快! 缺点是编译需要更长时间.

### 告诉 LLVM 优化大小而不是速度

默认情况下,调整 LLVM 的优化过程是提高速度,而不是大小. 我们可以通过修改目标, 来将目标更改为代码大小

`[profile.release]`部分:

```toml
[profile.release]
opt-level = 's'
```

或者,更进一步优化尺寸,以更大的速度成本:

```toml
[profile.release]
opt-level = 'z'
```

请注意,令人惊讶的是,`opt-level = "s"`,有时会导致 比 `opt-level = "z"` 更小. 总是要对比看看!

### 使用`wasm-opt`工具

该[Binaryen][]toolkit 是特定于 WebAssembly 的编译器工具的集合. 它比 LLVM 的 WebAssembly 后端 更进一步.

使用`wasm-opt`后,处理 LLVM 生成的`.wasm`二进制文件通常可以节省 15-20%的代码大小. 它通常还能帮运行时加速!

```bash
# Optimize for size.
wasm-opt -Os -o output.wasm input.wasm

# Optimize aggressively for size.
wasm-opt -Oz -o output.wasm input.wasm

# Optimize for speed.
wasm-opt -O -o output.wasm input.wasm

# Optimize aggressively for speed.
wasm-opt -O3 -o output.wasm input.wasm
```

[binaryen]: https://github.com/WebAssembly/binaryen

### Notes about Debug Information

ism 二进制大小的最大贡献者之一可以是调试信息和`names`wasm 二进制文件的一部分.该`wasm-pack`但是,工具默认删除 debuginfo.另外`wasm-opt`删除`names`默认情况下,除非`-g`也是指定的.

这意味着如果您按照上述步骤操作,则默认情况下不应该使用 debuginfo 或 wasm 二进制文件中的 names 部分.但是,如果您手动将其保留在 wasm 二进制文件中的此调试信息,请务必注意这一点!

## 大小分析

如果调整构建配置以优化代码大小后,不会导致足够小`.wasm`二进制,是时候进行一些分析,以查看剩余代码大小的来源.

> ⚡ 就像我们如何让时间分析指导我们的加速工作一样,我们希望让大小分析指导我们的代码大小缩小工作量. 不这样做,你可能会浪费自己的时间!

### 该`twiggy`代码大小分析器

[`twiggy`是一个代码大小分析器][twiggy]支持 WebAssembly 作为输入. 它分析二进制的调用图来回答如下问题:

- 为什么这个函数首先包含在二进制文件中?

- 这个函数*保留大小*多少? 即如果删除它, 以及删除后所有死代码的函数,将节省多少空间?

<style>
/* For whatever reason, the defautl mdbook fonts fonts break with the
   following box-drawing characters, hence the manual style. */
pre, code {
  font-family: "SFMono-Regular",Consolas,"Liberation Mono",Menlo,Courier,monospace;
}
</style>

```text
$ twiggy top -n 20 wasm_game_of_life_bg.wasm
 Shallow Bytes │ Shallow % │ Item
───────────────┼───────────┼────────────────────────────────────────────────────────────────────────────────────────
          9158 ┊    19.65% ┊ "function names" subsection
          3251 ┊     6.98% ┊ dlmalloc::dlmalloc::Dlmalloc::malloc::h632d10c184fef6e8
          2510 ┊     5.39% ┊ <str as core::fmt::Debug>::fmt::he0d87479d1c208ea
          1737 ┊     3.73% ┊ data[0]
          1574 ┊     3.38% ┊ data[3]
          1524 ┊     3.27% ┊ core::fmt::Formatter::pad::h6825605b326ea2c5
          1413 ┊     3.03% ┊ std::panicking::rust_panic_with_hook::h1d3660f2e339513d
          1200 ┊     2.57% ┊ core::fmt::Formatter::pad_integral::h06996c5859a57ced
          1131 ┊     2.43% ┊ core::str::slice_error_fail::h6da90c14857ae01b
          1051 ┊     2.26% ┊ core::fmt::write::h03ff8c7a2f3a9605
           931 ┊     2.00% ┊ data[4]
           864 ┊     1.85% ┊ dlmalloc::dlmalloc::Dlmalloc::free::h27b781e3b06bdb05
           841 ┊     1.80% ┊ <char as core::fmt::Debug>::fmt::h07742d9f4a8c56f2
           813 ┊     1.74% ┊ __rust_realloc
           708 ┊     1.52% ┊ core::slice::memchr::memchr::h6243a1b2885fdb85
           678 ┊     1.45% ┊ <core::fmt::builders::PadAdapter<'a> as core::fmt::Write>::write_str::h96b72fb7457d3062
           631 ┊     1.35% ┊ universe_tick
           631 ┊     1.35% ┊ dlmalloc::dlmalloc::Dlmalloc::dispose_chunk::hae6c5c8634e575b8
           514 ┊     1.10% ┊ std::panicking::default_hook::{{closure}}::hfae0c204085471d5
           503 ┊     1.08% ┊ <&'a T as core::fmt::Debug>::fmt::hba207e4f7abaece6
```

[twiggy]: https://github.com/rustwasm/twiggy

### 手动检查 LLVM-IR

LLVM-IR 是 LLVM 生成 WebAssembly 之前编译器工具链中的最终中间表示. 因此,它与最终发出的 WebAssembly 非常相似. 更多 LLVM-IR 通常意味着更多`.wasm`大小,如果一个函数占 LLVM-IR 的 25%,那么它通常会占 25%`.wasm`. 虽然这些数字一般只保留. LLVM-IR 具有关键信息,而这些信息并不存在`.wasm`中 (因为 WebAssembly 缺少像 DWAR F 这样的调试格式) : 哪些子程序被内联到 给定的函数中.

您可以使用此方法生成 LLVM-IR:

    cargo rustc --release -- --emit llvm-ir

然后,你可以使用`find`找到`.ll`包含 LLVM-IR 的文件:

    find target/release -type f -name '*.ll'

#### 参考

- [LLVM 语言参考手册](https://llvm.org/docs/LangRef.html)

## 更具侵入性的工具和技术

调整构建配置,以缩小`.wasm`二进制文件非常适合. 但是,当您需要加倍努力时,您准备使用更具侵入性的技术,例如重写源代码以避免膨胀. 接下来是一系列可以应用于获取较小代码的自适应技巧.

### 避免使用字符串格式

`format!`,`to_string`等等...可以带来很多代码臃肿. 如果可能,仅在调试模式下进行字符串格式化,在发布模式下使用静态字符串.

### 避免恐慌

这说起来容易做起来难,但工具就像`twiggy`并手动检查 LLVM-IR 可以帮助您找出哪些功能令人恐慌.

恐慌并不总是表现为`panic!()`宏调用. 它们隐含地来自许多结构,例如:

- 对超出范围索引的切片进行索引: `my_slice[i]`

- 如果除数为零,则 分得数 会惊慌失措: `dividend / divisor`

- 打开一个`Option`或者`Result`: `opt.unwrap()`或者`res.unwrap()`

前两个可以体现为第三个. 索引可以用`my_slice.get(i)`操作. 分得数 可以`checked_div`调用. 现在我们只有一个案例可以应对.

打开一个`Option`或者`Result`没有恐慌有两种风格: 安全和不安全.

安全的方法是`abort`代替恐慌,当得出一个`None`或一个`Error`:

```rust
#[inline]
pub fn unwrap_abort<T>(o: Option<T>) -> T {
    use std::process;
    match o {
        Some(t) => t,
        None => process::abort(),
    }
}
```

最终,无论如何在`wasm32-unknown-unknown`恐慌都会转化为 aborts ,所以这给你相同的行为,但没有代码膨胀.

或者,[`unreachable`箱][unreachable]提供不安全的[`unchecked_unwrap`扩展方法][unchecked_unwrap]对于`Option`和`Result`, 它告诉 Rust 编译器*承担*那个`Option`是`Some`或者`Result`是`Ok`. 它是未定义如果该假设不成立,会发生什么. 你真的只能当你 110%确定时,使用这种不安全的方法,而且编译器只是不够聪明看到它. 即使你沿着这条路走下去,你也应该有一个仍然进行检查的调试构建配置,并且只在发布版本中使用未经检查的操作.

[unreachable]: https://crates.io/crates/unreachable
[unchecked_unwrap]: https://docs.rs/unreachable/1.0.0/unreachable/trait.UncheckedOptionExt.html#tymethod.unchecked_unwrap

### 避免分配或切换到`wee_alloc`

Rust 对 WebAssembly 的默认分配器,是`dlmalloc`的一部分. 它的重量大约在 10 千字节左右. 如果你可以完全避免动态分配,那么你应该能够减少这十个千字节.

完全避免动态分配可能非常困难. 但是从热代码路径中删除分配通常要容易得多 (并且通常也有助于使这些热代码路径更快) . 在这些情况下,[用,替换默认的全局分配器`wee_alloc`][wee_alloc]应该节省你最多 (但不是全部) 的十千字节. `wee_alloc`是一个为您需要*一些*类型情况时设计,但不需要特别快的分配器,并将愉快地交换分配速度较小的代码大小的分配器.

[wee_alloc]: https://github.com/rustwasm/wee_alloc

### 使用特征对象而不是通用类型参数

当您创建使用类型参数的泛型函数时,如下所示:

```rust
fn whatever<T: MyTrait>(t: T) { ... }
```

然后`rustc`和 LLVM 将为每个创建一个新的函数副本 - `T`类型函数. 这为基于特定的`T`每个副本都在使用的编译器的优化提供了许多机会,但这些副本在代码大小方面快速增加.

如果您使用特征对象而不是类型参数,如下所示:

```rust
fn whatever(t: Box<MyTrait>) { ... }
// or
fn whatever(t: &MyTrait) { ... }
// etc...
```

然后使用通过虚拟调用的动态调度,并且仅在该函数中发出单个版本的函数`.wasm`. 缺点是失去了编译器优化机会,以及间接动态调度函数调用的额外成本.

### 使用`wasm-snip`工具

[`wasm-snip`用一个替换 WebAssembly 函数的主体的`unreachable`指令. ][snip]这是一个相当沉重,钝的锤子,如果你足够眯眼,那些看起来像指甲的函数.

也许您知道某些函数永远不会在运行时调用,但编译器无法在编译时证明这一点? 剪断它! 然后,跑`wasm-opt`带着`--dce`,以及`snipping 函数`传递调用的所有函数 (也可能永远不会在运行时调用) 也将被删除.

这个工具对于消除恐慌特别有用,因为恐慌最终会转化为陷阱.

[snip]: https://github.com/fitzgen/wasm-snip
