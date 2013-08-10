# 参与贡献

- [简介](#introduction)
- [推送请求](#pull-requests)
- [编码规范](#coding-guidelines)

<a name="introduction"></a>
## 简介

Laravel 是免费、开源的软件，这意味着任何人都可以对它的发展和进步做出贡献。Laravel 源码目前托管在 [Github](http://github.com)，它提供了 Fork 项目以及合并您贡献的代码的简易的方法。

<a name="pull-requests"></a>
## 推送请求

关于新特性和漏洞的推送请求的流程是不一样的。在为一个新特性发送请求前，您首先需要创建一个带有 `[Proposal]` 标题的 issue，这个建议应该描述这个新特性以及实现这个新特性的想法。这个建议将被浏览并决定是否采纳。一旦这个请求被通过，一个推送请求将被创建以实现这个新特性。不符合以上条款的推送请求将被立即关闭。

关于漏洞的推送请求可以不需要创建建议 issue。如果您相信您知道在 Github 列出的漏洞的解决办法，请留言告诉我们关于修复此漏洞的细节。

### 特性请求

如果您想在 Laravel 中加入一个新特性，您可以在 Github `[Request]` 中添加一个 issue，这个 特性请求将被核心开发者关注到。

<a name="coding-guidelines"></a>
## 编码规范

Laravel 遵循 [PSR-0](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-0.md) 和 [PSR-1](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-1-basic-coding-standard.md) 编码规范. 除了这些标准，下面是一些您也需要遵循的规范：

- 命名空间的声明应该和 `<?php` 在同一行。
- 类定义 `{` 应该和类名在同一行。
- 函数和控制结构 `{` 应该在下一行。
- 接口名应该有 `Interface` 后缀 （比如 `FooInterface`）。
