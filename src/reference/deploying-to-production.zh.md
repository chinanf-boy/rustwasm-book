# 发布 Rust 和 WebAssembly 到 生产环境

> **⚡ 部署使用 Rust 和 WebAssembly 构建的 Web 应用程序几乎与部署任何其他 Web 应用程序完全相同!**

要在客户端上，使用 Rust 生成的 WebAssembly 部署的 Web 应用程序，请将构建的 Web 应用程序的文件复制到生产服务器的文件系统，并配置 HTTP 服务器以使其可访问。

## 确定好 你的 HTTP 服务器 使用`application/wasm` MIME Type

为了最快的页面加载，您将要使用[`WebAssembly.instantiateStreaming`函数][instantiatestreaming]，其通过网络传输管理 wasm 编译和实例化(或确保您的 捆绑器 能够使用该函数)。然而,`instantiateStreaming`要求 HTTP 响应具有`application/wasm` 的 [MIME 类型][]设置，否则会抛出错误。

- [如何为 Apache HTTP 服务器配置 MIME 类型][apache-mime]
- [如何为 NGINX HTTP 服务器配置 MIME 类型][nginx-mime]

[instantiatestreaming]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/WebAssembly/instantiateStreaming
[mime type]: https://developer.mozilla.org/en-US/docs/Web/HTTP/Basics_of_HTTP/MIME_types
[apache-mime]: https://httpd.apache.org/docs/2.4/mod/mod_mime.html#addtype
[nginx-mime]: https://nginx.org/en/docs/http/ngx_http_core_module.html#types

## 更多资源

- [Webpack 在生产环境中的最佳实践。][webpack-prod]：许多 Rust 和 WebAssembly 项目使用 Webpack ，去捆绑 Rust 生成的 WebAssembly，JavaScript，CSS 和 HTML。本指南提供了，部署到生产环境时，尽用 Webpack 的技巧。
- [Apache 文档.][apache]Apache 是 ​​ 一种流行的 HTTP 服务器,可用于生产.
- [NGINX 文档.][nginx]NGINX 是一种用于生产的流行 HTTP 服务器.

[webpack-prod]: https://webpack.js.org/guides/production/
[apache]: https://httpd.apache.org/docs/
[nginx]: https://docs.nginx.com/nginx/admin-guide/installing-nginx/installing-nginx-open-source/
