# 配置

- [简介](#introduction)
- [环境配置](#environment-configuration)
- [维护模式](#maintenance-mode)

<a name="introduction"></a>
## 简介

Laravel 框架的所有配置文件都存放在 `app/config` 目录下。每个文件的每个选项都被详细的注释，所以您能够很轻松地浏览这些变量。

有些时候您可能需要在程序运行时设置配置变量，您可以使用 `Config` 类类进行读取和设置：

**获取一个配置值**

	Config::get('app.timezone');

您可能希望在配置选项不存在时指定一个默认值：

	$timezone = Config::get('app.timezone', 'UTC');

注意点式语法可以用来获取在多个文件的配置值。您也可能希望在程序运行时设置配置值：

**设置一个配置值**

	Config::set('database.default', 'sqlite');

在程序运行时设置的配置值只在本次请求中有效，不会对以后的请求造成影响。

<a name="environment-configuration"></a>
## 环境配置

通常在不同的环境中设置不同的配置值是非常有用的。比如，您可能想在本地环境中使用不同于生产环境的缓存驱动。这些通过基于环境的配置能够很容易实现。

在 `config` 目录下建立一个符合环境的名字，比如 `local`。接着，创建这个环境需要修改的配置文件。比如，为 `local` 环境重写缓存驱动，在 `app/config/local` 中创建包含以下内容的 `cache.php`：

	<?php

	return array(

		'driver' => 'file',

	);

> **注意:** 请不要使用 'testing' 作为环境的名字。这个名字在单元测试中保留使用。

请注意，您不需要为新环境指定基础配置中的每一个配置变量，只需要设置那些您需要重写的配置变量。其他配置变量将从那些基础配置文件中继承。

下面，您需要通知框架如何检测当前的环境。默认的环境通常是 `production`。您可以通过修改根目录下的 `bootstrap/start.php` 文件设置其他环境。在这个文件中，您将会发现一个 `$app->detectEnvironment` 函数调用，传给这个函数的参数数组被用于检测当前的环境。您可以根据需要向数组中添加其他环境和机器名。

    <?php

    $env = $app->detectEnvironment(array(

        'local' => array('your-machine-name'),

    ));

您也可以向 `detectEnvironment` 函数传递一个闭包函数，这将允许您实现自己的环境监测方法：

	$env = $app->detectEnvironment(function()
	{
		return $_SERVER['MY_LARAVEL_ENV'];
	});

您可以通过 `environment` 方法获取当前的应用程序环境：

**获取当前的应用程序环境**

	$environment = App::environment();

<a name="maintenance-mode"></a>
## 维护模式

当您的应用在维护模式时，对于应用中所有路由的请求将显示一个定制的视图。这方便您更新应用时禁用应用程序。调用的 `App::down` 方法位于 `app/start/global.php` 文件，这个函数的响应将在维护模式时发送给用户。

开启维护模式，请执行 Artisan 的 `down` 命令：

	php artisan down

关闭维护模式，使用 `up` 命令：

	php artisan up

为了在维护模式时显示一个定制的视图，您可以在应用的 `app/start/global.php` 文件中加入以下代码：

	App::down(function()
	{
		return Response::view('maintenance', array(), 503);
	});
