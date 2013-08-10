# Artisan命令行

- [简介](#introduction)
- [用法](#usage)

<a name="introduction"></a>
## 简介

Artisan 是 Laravel 框架的内置的命令行接口。它为开发应用程序提供了一系列的有用的命令。它基于强大的 Symfony 命令行组件。

<a name="usage"></a>
## 用法

使用 `list` 命令显示所有可用的 Artisan 命令：

**显示所有可用的命令**

	php artisan list

每一条命令包含一个帮助窗口显示该命令可用的参数和选项。为了显示帮助窗口，请在命令的名字前加上 `help`：

**显示一条命令的帮助窗口**

	php artisan help migrate

您可以通过 `--env` 选项指定一条命令运行时应当使用的应用环境：

**指定应用环境**

	php artisan migrate --env=local

您也可以通过 `--version` 选项显示当前安装的 Laravel 的版本：

**显示您当前Laravel的版本**

	php artisan --version
