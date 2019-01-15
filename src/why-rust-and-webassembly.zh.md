# Why Rust and WebAssembly?

## Low-Level Control with High-Level Ergonomics

JavaScript Web应用程序很难获得并保持可靠的性能.JavaScript的动态类型系统和垃圾收集暂停没有帮助.如果您不小心徘徊在JIT的快乐路径上,看似很小的代码更改可能导致严重的性能回归.

Rust为程序员提供了低级控制和可靠的性能.它不受困扰JavaScript的非确定性垃圾收集暂停的影响.程序员可以控制间接,单态化和内存布局.

## Small `.wasm` Sizes

代码大小非常重要,因为`.wasm`必须通过网络下载.Rust缺少运行时,可以实现小型运行`.wasm`大小,因为没有像垃圾收集器那样包含额外的膨胀.您只需为实际使用的功能支付(代码大小).

## Do *Not* Rewrite Everything

不需要丢弃现有的代码库.您可以从将性能最敏感的JavaScript函数移植到Rust开始,立即获益.如果你愿意,你甚至可以在那里停下来.

## Plays Well With Others

Rust和WebAssembly与现有的JavaScript工具集成.它支持ECMAScript模块,您可以继续使用您喜欢的工具,如npm,Webpack和Greenkeeper.

## The Amenities You Expect

Rust拥有开发人员所期望的现代化设施,例如:

-   强大的包管理`cargo`,

-   富有表现力(和零成本)的抽象,

-   和一个热情的社区!😊
