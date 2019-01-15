# rustwasm/book [![explain]][source] [![translate-svg]][translate-list]

<!-- [![size-img]][size] -->

[explain]: http://llever.com/explain.svg
[source]: https://github.com/chinanf-boy/Source-Explain
[translate-svg]: http://llever.com/translate.svg
[translate-list]: https://github.com/chinanf-boy/chinese-translate-list
[size-img]: https://packagephobia.now.sh/badge?p=Name
[size]: https://packagephobia.now.sh/result?p=Name

「 此 `repo` 包含有关使用 `Rust` 到 `wasm` ,常见工作流, 如何入门等的文档.

它可以作为 如何整洁做事 的指南. 」

[中文](./readme.md) | [english](https://github.com/rustwasm/book)

---

## 校对 🀄️

<!-- doc-templite START generated -->
<!-- repo = 'rustwasm/book' -->
<!-- commit = '7cdec718cd95bbeca0164935e74d72a690a552f1' -->
<!-- time = '2018-12-21' -->

| 翻译的原文 | 与日期        | 最新更新 | 更多                       |
| ---------- | ------------- | -------- | -------------------------- |
| [commit]   | ⏰ 2018-12-21 | ![last]  | [中文翻译][translate-list] |

[last]: https://img.shields.io/github/last-commit/rustwasm/book.svg
[commit]: https://github.com/rustwasm/book/tree/7cdec718cd95bbeca0164935e74d72a690a552f1

<!-- doc-templite END generated -->

- [x] [README.md](README.md)
- [ ] [src/setup.md](src/setup.md)
- [ ] [src/SUMMARY.md](src/SUMMARY.md)
- [ ] [src/introduction.md](src/introduction.md)
- [ ] [src/why-rust-and-webassembly.md](src/why-rust-and-webassembly.md)
- [ ] [src/background-and-concepts.md](src/background-and-concepts.md)
- [ ] [src/game-of-life/deploying-to-production.md](src/game-of-life/deploying-to-production.md)
- [ ] [src/game-of-life/hello-world.md](src/game-of-life/hello-world.md)
- [ ] [src/game-of-life/code-size.md](src/game-of-life/code-size.md)
- [ ] [src/game-of-life/implementing.md](src/game-of-life/implementing.md)
- [ ] [src/game-of-life/setup.md](src/game-of-life/setup.md)
- [ ] [src/game-of-life/time-profiling.md](src/game-of-life/time-profiling.md)
- [ ] [src/game-of-life/introduction.md](src/game-of-life/introduction.md)
- [ ] [src/game-of-life/testing.md](src/game-of-life/testing.md)
- [ ] [src/game-of-life/interactivity.md](src/game-of-life/interactivity.md)
- [ ] [src/game-of-life/rules.md](src/game-of-life/rules.md)
- [ ] [src/game-of-life/publishing-to-npm.md](src/game-of-life/publishing-to-npm.md)
- [ ] [src/game-of-life/debugging.md](src/game-of-life/debugging.md)
- [ ] [src/what-is-webassembly.md](src/what-is-webassembly.md)
- [ ] [src/reference/deploying-to-production.md](src/reference/deploying-to-production.md)
- [ ] [src/reference/add-wasm-support-to-crate.md](src/reference/add-wasm-support-to-crate.md)
- [ ] [src/reference/code-size.md](src/reference/code-size.md)
- [ ] [src/reference/js-ffi.md](src/reference/js-ffi.md)
- [ ] [src/reference/time-profiling.md](src/reference/time-profiling.md)
- [ ] [src/reference/index.md](src/reference/index.md)
- [ ] [src/reference/which-crates-work-with-wasm.md](src/reference/which-crates-work-with-wasm.md)
- [ ] [src/reference/crates.md](src/reference/crates.md)
- [ ] [src/reference/project-templates.md](src/reference/project-templates.md)
- [ ] [src/reference/debugging.md](src/reference/debugging.md)
- [ ] [src/reference/tools.md](src/reference/tools.md)

### 贡献

欢迎 👏 勘误/校对/更新贡献 😊 [具体贡献请看](https://github.com/chinanf-boy/chinese-translate-list#贡献)

## 生活

[help me live , live need money 💰](https://github.com/chinanf-boy/live-need-money)

---

## The Rust and WebAssembly Book

如果您想立即开始学习如何一起使用 Rust 和 WebAssembly，您可以阅读本书 [online `en`](https://rustwasm.github.io/book/game-of-life/introduction.html) | [中文网站](http://llever.com/rustwasm-book/)

[打开用于改进 Rust 和 WebAssembly 书籍 的 Issue.][book-issues]

[book-issues]: https://github.com/rustwasm/book/issues

## 建立这本书

这本书是用来制作的[`mdbook`][mdbook]. 要安装它,你需要`cargo`安装. 如果您没有安装任何 Rust 工具,则第一安装[`rustup`][rustup]. 按照网站上的说明进行设置.

完成后,只需执行以下操作:

```bash
$ cargo install mdbook
```

确保`cargo install`目录在你的`$PATH`, 这样你就可以运行二进制文件了.

现在只需从此目录运行此命令:

```bash
$ mdbook build
```

这将把书和输出文件构建到一个名为`book`的目录中. 到`index.html`文件以在浏览器中查看它.

```bash
$ mdbook serve
```

这将在您进行更改时自动生成文件,这样您就可以轻松查看它们而无需每次调用`build`.

这些文件都是用`Markdown`编写的,所以如果你不想生成书来阅读它们,那么你可以从中读取它们`src`目录.

[mdbook]: https://github.com/rust-lang-nursery/mdBook
[rustup]: https://github.com/rust-lang-nursery/rustup.rs/
