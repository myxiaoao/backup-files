# Laravel 快速指南

- [安装](#installation)
- [路由](#routing)
- [创建一个视图](#creating-a-view)
- [创建一个迁移](#creating-a-migration)
- [Eloquent ORM](#eloquent-orm)
- [显示数据](#displaying-data)

<a name="installation"></a>
## 安装

为了安装 Laravel 框架，可以在终端输入以下命令：

	composer create-project laravel/laravel your-project-name

或者，也可以从 Github 下载 [代码库](https://github.com/laravel/laravel/archive/master.zip)。在 [安装 Composer](http://getcomposer.org) 之后，在项目文件的主目录下运行 `composer install` 命令，这个命令将下载和安装框架的依赖文件。

安装框架结束后，请浏览一下项目并熟悉目录结构。`app` 目录包含 `views`, `controllers` 以及 `models` 等文件夹。应用程序的大多数代码将位于这个目录下。您也许会浏览 `app/config` 目录，这里包含了应用程序的各配置选项。

<a name="routing"></a>
## 路由

作为开始，让我们创建第一个路由。在 Laravel 中创建路由最简单的办法是使用闭包函数。打开 `app/routes.php` 文件，在文件的底部添加以下路由：

	Route::get('users', function()
	{
		return 'Users!';
	});

现在，如果您在浏览器中输入 `/users`路由，您将会看 `Users!` 作为响应显示。很好！您已经创建了第一个路由。

路由也可以绑定到控制器类，比如：

	Route::get('users', 'UserController@getIndex');

这个路由告诉框架对 `/users` 路由的请求用该调用 `UserController` 类的 `getIndex` 函数。更多关于控制器路由的内容请访问 [控制器文档](/docs/controllers) 。

<a name="creating-a-view"></a>
## 创建一个视图

下面，我们将创建一个简单的试图显示用户数据。视图文件存放在 `app/views` 目录下并且包含应用的 HTML 。我们在这个目录下添加两个视图文件 `layout.blade.php` 以及 `users.blade.php`。首先创建 `layout.blade.php` 文件：

	<html>
		<body>
			<h1>Laravel Quickstart</h1>

			@yield('content')
		</body>
	</html>

接着, 创建 `users.blade.php` 视图:

	@extends('layout')

	@section('content')
		Users!
	@stop

您可能觉得里面的一些语法很奇怪。那是因为我们在使用 Laravel 的模板系统：Blade。Blade 是非常快的，因为只需要把模板中的一些正则表达式转换为纯PHP。Blade提供了强大的功能比如模板继承，
还有为一些典型的控制结构比如 `if` 和 `for` 提供的语法糖。更多信息请浏览 [Blade 文档](/docs/templates)。

现在我们已经创建了视图，让我们返回到 `/users` 路由，让它返回视图文件的内容：

	Route::get('users', function()
	{
		return View::make('users');
	});

很好！现在我们已经建立了一个简单的从布局扩展的视图，下面，我们将开始使用数据库。

<a name="creating-a-migration"></a>
## 创建一个迁移

为了创建一张表保存我们的数据，我们将使用 Laravel 的迁移系统。迁移系统允许我们用具有表现力的方式定义数据库的修改，并且很方便的与团队的其他成员共享。

首先，让我们配置数据库的连接。您可以 `app/config/database.php` 文件中配置所有数据库的连接。默认情况下，Laravel 使用 SQLite，并且 SQLite 数据库保存在 `app/database` 目录下，如果您愿意，您也可以改变 `driver` 选项为 `mysql` 并且在配置文件中设置 `mysql` 的连接认证参数。

下面，为了创建迁移，我们将使用 [Artisan 命令行](/docs/artisan)。切换到项目的根目录，在终端执行下面的命令：

	php artisan migrate:make create_users_table

下面，在 `app/database/migrations` 目录下找到生成的迁移文件。这个文件包含一个类，类中定义了两个函数：`up` 以及 `down`。在 `up` 方法中对数据库做您想要的改变，在 `down` 方法中撤消所做的改变。

让我们像下面这样定义一个迁移：

	public function up()
	{
		Schema::create('users', function($table)
		{
			$table->increments('id');
			$table->string('email')->unique();
			$table->string('name');
			$table->timestamps();
		});
	}

	public function down()
	{
		Schema::drop('users');
	}

下面，我们可以在终端使用 `migrate` 命令执行上面的迁移。在项目的根目录运行以下命令：

	php artisan migrate

如果您想回滚一个迁移，您可以关注 `migrate:rollback` 命令。现在我们已经有了数据库的表，让我们开始填充一些数据!

<a name="eloquent-orm"></a>
## Eloquent ORM

Laravel 拥有一个极好的ORM：Eloquent。如果您以前使用过 Ruby on Rails 框架，您会发现 Eloquent 与它很相似，因为它遵从数据库交互的 ActiveRecord ORM 模式。

首先，让我们顶一个模型，一个 Eloquent 模型可以用来查询一个相关的数据库的表，也可以代表表中的一行。不要担心，很快就会水落石出。模型文件一般保存在 `app/models` 目录。让我们像下面这样定义一个 `User.php` 模型：

	class User extends Eloquent {}

注意我们没有必要告诉 Eloquent 使用了哪一张表。Eloquent 有很多预订的惯例，其中之一就是模型名的复数作为模型的数据库中的表名。非常方便！

使用您喜爱的数据库管理员工具在 `users` 表中插入一些数据，我们将使用 Eloquent 获取这些数据并传递给视图。

现在我们像下面这样修改 `/users` 路由：

	Route::get('users', function()
	{
		$users = User::all();

		return View::make('users')->with('users', $users);
	});

让我们浏览一下这个路由。首先，`User` 模型的 `all` 方法将获取 `users` 表中的所有数据。然后，我们通过 `with` 函数将这些记录传递给视图。`with` 方法接受一个名字和一个值，用来向视图中传递可用的数据。

真了不起。下面我们将在视图中显示这些用户。

<a name="displaying-data"></a>
## 显示数据

现在我们已经在视图中传递了 `users` 变量。我们可以像这样显示它们：

	@extends('layout')

	@section('content')
		@foreach($users as $user)
			<p>{{ $user->name }}</p>
		@endforeach
	@stop

您可能想知道在哪里可以找到 `echo` 表达式。当使用 Blade 模板的时候，您可以通过用两个大括号包围数据的方式显示它。现在您能够访问 `/users` 路由，并且看到用户的名字显示在屏幕上。

这只是一个开端。在这篇指南中，您已经知晓了一些 Laravel 的基础知识，但是还有很多更精彩的内容需要学习，请继续阅读文档，并且更深入地了解 [Eloquent](/docs/eloquent) 以及 [Blade](/docs/templates)。或者，也许您对 [队列](/docs/queues) 和 [单元测试](/docs/testing) 更感兴趣，抑或您想通过 [IoC 容器](/docs/ioc) 重构，一切都取决于您。

