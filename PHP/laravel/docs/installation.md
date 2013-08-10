# 安装

- [安装Composer](#install-composer)
- [安装Laravel](#install-laravel)
- [服务器要求](#server-requirements)
- [配置](#configuration)
- [友好的URL](#pretty-urls)

<a name="install-composer"></a>
## 安装Composer

Laravel 使用 [Composer](http://getcomposer.org) 管理包依赖关系。 首先，请下载 `composer.phar`。 当您有了这个 PHAR 打包文件，您可以把它放在本地的项目目录下，或者移至 `usr/local/bin` 目录使得可以全局调用。在 Windows 环境，您可以使用 Composer [Windows 安装文件](https://getcomposer.org/Composer-Setup.exe) 安装。

<a name="install-laravel"></a>
## 安装Laravel

### 通过 Composer Create-Project

您可以通过运行 Composer 的 `create-project` 命令安装 Laravel：

	composer create-project laravel/laravel

### 通过下载

确认 Composer 已安装，下载 Laravel 框架的 [最新版本](https://github.com/laravel/laravel/archive/master.zip) 并解压到服务器的指定目录。然后，在 Laravel 应用的根目录运行 `php composer.phar install` （或者 `composer install`）安装框架的依赖。这个流程需要服务器上安装了 Git 以完成安装。

如果您想更新 Laravel 框架，可以使用 `php composer.phar update` 命令。

<a name="server-requirements"></a>
## 服务器要求

Laravel 框架仅需要以下服务器要求：

- PHP >= 5.3.7
- MCrypt PHP Extension

<a name="configuration"></a>
## 配置

Laravel 几乎是一个需要零配置的框架，您可以立即开始开发程序！也许您想浏览 `app/config/app.php` 文件和它的文档。它包含了一些比如 `timezone` 以及 `locale` 等您想根据应用修改的配置选项。

> **注意:** 一个您需要确认的配置选项是在 `app/config/app.php` 文件中的 `key` 选项。这个选项应该设置为一个32个字符的随机字符串。这个值将被用于加密，只有当这个值正确的设置，加密的值才是安全的。您可以通过以下命令快速设置这个值： `php artisan key:generate`。

<a name="permissions"></a>
### 权限
Laravel 要求一组权限被设置 - `app/storage` 目录下的文件夹必须设置为 Web 服务器可写。

<a name="paths"></a>
### 目录

框架的一些目录的路径是可设置的。改变这些路径，请检查 `bootstrap/paths.php` 文件。

> **注意:** Laravel 被设计为保护您的应用程序以及本地存储，因此，只在 public 目录下存放必须的文件。推荐您设置 public 目录为您站点的 documentRoot （也常称为web root），或者把public 目录下的内容放在站点的 root 目录并把 Laravel 的其他文件放在 web root 以外的目录。  

<a name="pretty-urls"></a>
## 友好的URL

框架使用 `public/.htaccess` 文件允许 URLs 中去掉 `index.php`。如果您使用 Apache 服务器，请确认开启了 `mod_rewrite` 模块。

如果 Laravel 自带的 `.htaccess` 文件在您的 Apache 服务器上没有作用，请尝试以下的方法：

	Options +FollowSymLinks
	RewriteEngine On

	RewriteCond %{REQUEST_FILENAME} !-f
	RewriteRule ^ index.php [L]
