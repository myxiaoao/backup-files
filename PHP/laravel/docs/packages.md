# 包开发

- [简介](#introduction)
- [创建一个包](#creating-a-package)
- [包结构](#package-structure)
- [服务提供商](#service-providers)
- [包约定](#package-conventions)
- [开发流程](#development-workflow)
- [包路由](#package-routing)
- [包配置](#package-configuration)
- [包迁移](#package-migrations)
- [包资源](#package-assets)
- [发布包](#publishing-packages)

<a name="introduction"></a>
## 简介

包主要用于向 Laravel 中添加功能。包既可以是用于处理日期的 [Carbon](https://github.com/briannesbitt/Carbon)，也可以是一整套 像 [Behat](https://github.com/Behat/Behat) 这样的 BDD 测试框架。

当然，包有不同的类型。有一些包是独立的，意味着可以在任何框架中使用而不仅是 Laravel。Carbon 和 Behat 都是独立包的例子，他们都可以通过 `composer.json` 声明在 Laravel 中被使用。

另一方面，其他包仅针对 Laravel 使用。在早期的 Laravel 版本中，这种类型的包被称为 "bundles"。这些包可能包含路由、控制器、视图、配置以及迁移仅用于扩展一个 Laravel 应用程序。因为开发一个独立的包没有特别的流程，本指南将主要涵盖只针对 Laravel 的包的开发。

所有 Laravel 的包由 [Packagist](http://packagist.org) 和 [Composer](http://getcomposer.org) 分发，所以学习这些 PHP 包分发工具是必不可少的。

<a name="creating-a-package"></a>
## 创建一个包

最简单的方法来创建一个在 Laravel 中使用的包是 `workbench` Artisan 命令。首先，您需要在 `app/config/workbench.php` 文件中设置一些选项。在这个文件中，您会发现一个 `name` 以及 `email` 选项。这些值将被使用于为您的新包生成一个 `composer.json` 文件。一旦您已经提供了这些值，您就可以构建一个工作台包了！

**关于工作台 Artisan 命令**

	php artisan workbench vendor/package --resources

供应商的名字可以用来区分您的包和其他来自不同的作者但相同的名字的包。比如，我（Taylor Otwell）想创建一个名为 "Zapper" 的包，那么供应商的名字可以为 `Taylor` 同时包名可以为 `Zapper`。默认情况下，工作台将创建不可知的包，但是 `resources` 命令告诉工作台产生带有 Laravel 相关目录的包，比如 `migrations`, `views`, `config` 等。

一旦 `workbench`  命令已经被执行，您的包将在您安装的 Laravel 的 `workbench` 目录。接下来，您应该注册您的包所创建的 `ServiceProvider`。您可以注册供应商通过添加它到 `app/config/app.php` 文件中的 `providers` 数组。这将告诉 Laravel 在应用程序启动时加载您的包。服务供应商使用一个 `[Package]ServiceProvider` 命名惯例。所以使用上面那个例子，您应该添加 `Taylor\Zapper\ZapperServiceProvider` 到 `providers` 数组。

一旦供应商被注册，您就可以开发您的包了！但是，在开始之前，您可能希望浏览下面的章节对包结构和开发流程进行更深入的了解。

<a name="package-structure"></a>
## 包结构

当使用 `workbench` 命令，您的包将按照惯例以允许包与 Laravel 框架的其他部分协调工作的方式进行设置：

**基本包的目录结构**

	/src
		/Vendor
			/Package
				PackageServiceProvider.php
		/config
		/lang
		/migrations
		/views
	/tests
	/public

让我进一步探索这个结构。`src/Vendor/Package` 目录是所有包的类的主目录，包括 `ServiceProvider`。`config`, `lang`, `migrations` 以及 `views` 目录和您想的一样，包含相关的包的资源。包可以有任何这些资源，就像一个常规的应用程序一样。

<a name="service-providers"></a>
## 服务提供商

服务提供商是包的一个简单的引导类。默认情况下，它包含两个函数，`boot` 和 `register`。在这些函数中，您可以做这些事情，比如包含一个路由文件，注册 IoC 容器的绑定，附加事件或者任何您想做的事情。

`register` 函数当服务提供商被注册时立即调用，`boot` 函数只有在一个请求被路由之前调用。所以，如果在服务提供商中的行为依赖于另一个已注册的服务提供商，或者您重写另一个提供商的服务，您应该使用 `boot` 函数。

当使用 `workbench` 创建一个包，`boot` 命令将已经包含一个行为：

	$this->package('vendor/package');

这个函数允许 Laravel 知道怎样正确地为您的应用程序加载视图、配置文件以及其他资源。一般来说，您没有必要改变这行代码，因为它将按照工作台管理创建一个包。

<a name="package-conventions"></a>
## 包约定

当从一个包中使用资源，比如配置选项或者视图，一个双冒号语法通常会被使用：

**从包中加载一个视图**

	return View::make('package::view.name');

**检索包中的一个配置选项**

	return Config::get('package::group.option');

> **注意:** 如果您的包包含迁移，考虑为您的迁移加上包名的前缀避免潜在的与其他包的类名冲突。

<a name="development-workflow"></a>
## 开发流程
 
当开发一个包，在应用程序的上下文环境中开发是有用的，允许您轻松地查看和试验您的模板等。所以为了起步，安装一个全新的 Laravel 框架，然后使用 `workbench` 命令创建您的包结构。

在使用 `workbench` 命令创建您的包之后，您可以在 `workbench/[vendor]/[package]` 目录运行 `git init` 以及 `git push`。这将允许您方便在应用程序上下文环境中开发包而不必陷入使用 `composer update` 命令。

因为您的包在 `workbench` 目录，您可能像知道 Composer 怎样知道自动加载您包中的文件。当 `workbench` 目录存在的时候，Laravel 将智能地扫描包，当应用程序启动的时候加载它们的 Composer 自动加载文件。

如果你需要重新生成您的包的自动加载文件，您可以使用 `php artisan dump-autoload` 命令。这个命令将为您的项目以及如何您创建的工作台重新生成自动加载文件。

**运行 Artisan 自动加载命令**

	php artisan dump-autoload

<a name="package-routing"></a>
## 包路由

在之前版本的 Laravel 中，一个 `handles` 条款用于指明一个包可以响应哪些 URIs。但是，在 Laravel 4，一个包可以响应任何 URI。为您的包加载一个路由文件，简单地在服务提供商的 `boot` 函数中使用 `include` 它。

**从一个服务提供商中包含一个路由**

	public function boot()
	{
		$this->package('vendor/package');

		include __DIR__.'/../../routes.php';
	}

> **注意:** 如果您的包使用控制器，您需要确认在 `composer.json` 文件的自动加载段中正确地配置。

<a name="package-configuration"></a>
## 包配置

一些包可能包含配置文件。这些文件应该以常规应用程序配置文件的方式被定义。并且，在使用默认的 `$this->package` 方法在服务供应商中注册资源，可以使用通常的双冒号语法：

**访问包配置文件**

	Config::get('package::file.option');

然而，如果您的包仅包含一个单独的配置文件，您可以简单地命名这个文件为 `config.php`。当这样做，您可以直接访问这些选项，而不需要指定文件名。

**访问包的单文件配置**

	Config::get('package::option');

### 配置文件层叠

当其他的开发者安装了您的包，他们可能希望重写一些您配置中的选项。然而，如果他们改变包的源代码的值，他们需要在 Composer 升级这个包的时候再次重写一次。相反，`config:publish` Artisan 命令可以被使用：

**执行配置发布指令**

	php artisan config:publish vendor/package

当这条命令被执行，应用程序的配置文件将被拷贝到 `app/config/packages/vendor/package` 可以被开发者安全地修改。

> **注意:** 开发者也可以为您的包创建指定应用环境的配置文件，在 `app/config/packages/vendor/package/environment` 目录下。

<a name="package-migrations"></a>
## 包迁移

您可以轻松的为您的包创建和运行迁移。在工作台为您的包创建一个迁移，请使用 `--bench` 选项：

**为工作台包创建迁移**

	php artisan migrate:make create_users_table --bench="vendor/package"

**为工作台包运行迁移**

	php artisan migrate --bench="vendor/package"

为一个通过 Composer 安装到 `vendor` 目录的已完成的包运行迁移，您可以使用 `--package` 选项：

**为一个已安装的包运行迁移**

	php artisan migrate --package="vendor/package"

<a name="package-assets"></a>
## 包资源

一些包可能包含JavaScript、CSS、以及图片资源。但是我们无法通过 `vendor` 或 `workbench` 目录访问这些资源，所以我们需要一种方法移动这些资源文件转移到应用程序的 `public` 目录。`asset:publish` 命令将为您做这些工作：

**转移包资源到Public目录**

	php artisan asset:publish

	php artisan asset:publish vendor/package

如果包仍在 `workbench`，请使用 `--bench` 选项：

	php artisan asset:publish --bench="vendor/package"

这个命令将根据供应商和包名转移资源文件到 `public/packages` 目录。因此，一个名为 `userscape/kudos` 的包的资源文件将转移到  `public/packages/userscape/kudos` 目录下。使用这个资源发布惯例将允许您在包的视图中安全的使用资源路径。

<a name="publishing-packages"></a>
## 发布包

当您的包准备发布的时候，您应该提交包到 [Packagist](http://packagist.org) 仓库。如果这个包只适用于 Laravel，可以考虑在包的 `composer.json` 文件中添加一个 `laravel` 标记。

同时，对版本号做标记将是有礼貌且有帮助的，使开发者勊只在他们的 `composer.json` 文件中依赖稳定的版本。如果没有一个稳定的版本，可以考虑使用 `branch-alias` Composer 指令。

一旦您的包已发布，请在通过 `workbench` 创建的应用程序中继续发展它，在包发布之后进行持续的开发是一个很好的方式。

有一些组织选择选择为他们的开发者开放他们私有的代码仓库。如果您对此感兴趣，参考 Composer 团队提供的 [Satis](http://github.com/composer/satis) 项目的文档。
