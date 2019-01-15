# Deploying Rust and WebAssembly to Production

> **⚡部署使用Rust和WebAssembly构建的Web应用程序几乎与部署任何其他Web应用程序完全相同!**

要在客户端上部署使用Rust生成的WebAssembly的Web应用程序,请将构建的Web应用程序的文件复制到生产服务器的文件系统,并配置HTTP服务器以使其可访问.

## Ensure that Your HTTP Server Uses the `application/wasm` MIME Type

对于最快的页面加载,您将要使用[该`WebAssembly.instantiateStreaming`功能][instantiatestreaming]通过网络传输管理wasm编译和实例化(或确保您的bundler能够使用该功能).然而,`instantiateStreaming`要求HTTP响应具有`application/wasm` [MIME类型][]设置,否则会抛出错误.

-   [如何为Apache HTTP服务器配置MIME类型][apache-mime]
-   [如何为NGINX HTTP服务器配置MIME类型][nginx-mime]

[instantiatestreaming]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/WebAssembly/instantiateStreaming

[mime type]: https://developer.mozilla.org/en-US/docs/Web/HTTP/Basics_of_HTTP/MIME_types

[apache-mime]: https://httpd.apache.org/docs/2.4/mod/mod_mime.html#addtype

[nginx-mime]: https://nginx.org/en/docs/http/ngx_http_core_module.html#types

## More Resources

-   [Webpack在生产中的最佳实践.][webpack-prod]许多Rust和WebAssembly项目使用Webpack捆绑Rust生成的WebAssembly,JavaScript,CSS和HTML.本指南提供了在部署到生产环境时充分利用Webpack的技巧.
-   [Apache文档.][apache]Apache是​​一种流行的HTTP服务器,可用于生产.
-   [NGINX文档.][nginx]NGINX是一种用于生产的流行HTTP服务器.

[webpack-prod]: https://webpack.js.org/guides/production/

[apache]: https://httpd.apache.org/docs/

[nginx]: https://docs.nginx.com/nginx/admin-guide/installing-nginx/installing-nginx-open-source/
