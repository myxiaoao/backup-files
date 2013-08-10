# 请求的生命周期

- [概要](#overview)
- [启动文件](#start-files)
- [应用事件](#application-events)

<a name="overview"></a>
## 概要

在 Laravel 中，请求的生命周期是非常简单的。一个请求进入到您的应用程序并且被分发到合适的路由或控制器。然后路由的响应被发送到浏览器并显示在屏幕上。有些时候您可能希望在路由被真正调用之前或之后进行一些操作。有好几种方法可以做这份工作，其中的两个是启动文件和应用事件。

<a name="start-files"></a>
## 启动文件

您的应用程序的启动文件被存放在 `app/start` 目录。默认有三个被包含进您的应用程序，分别是 `global.php`, `local.php` 以及 `artisan.php`，关于 `artisan.php` 的文档，请参考 [Artisan 命令行](/docs/commands#registering-commands)。

`global.php` 这个文件的开头默认将包括一些基本的步骤，比如 [Logger](/docs/errors) 的注册，还有包含 `app/filters.php` 文件等。但是您可以按照您的意图在文件中加入任何东西。它将在每次请求中被自动包含到应用程序中，而不论在什么应用环境中。`local.php` 这个文件就不一样了，它只在 `local` 应用环境中被调用。关于应用环境的更多信息，请参考 [配置](/docs/configuration) 文档。

当然，如果您有除了 `local` 以外的应用环境，您也可以为之创建启动文件。当应用在这个环境中运行时，这个启动文件将被自动包含进去。

<a name="application-events"></a>
## 应用事件

您也可能希望通过注册 `before`, `after`, `close`, `finish` 以及 `shutdown` 应用事件在请求之前或之后进行处理：

**注册应用事件**

	App::before(function()
	{
		//
	});

	App::after(function($request, $response)
	{
		//
	});

这些事件的监听器将在您的应用程序的每个请求的 `before` 以及 `after` 事件触发时执行。
