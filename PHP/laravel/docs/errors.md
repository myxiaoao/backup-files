# 错误与日志

- [错误信息](#error-detail)
- [错误处理](#handling-errors)
- [HTTP异常](#http-exceptions)
- [处理404错误](#handling-404-errors)
- [日志](#logging)

<a name="error-detail"></a>
## 错误信息

默认情况下，错误信息在应用程序中是开启的，这意味着当一个错误发生的时候，您会看到到错误代码跟踪和错误信息显示在出错页上。您可以通过设置 `app/config/app.php` 文件中的 `debug` 选项为 `false` 来关闭错误信息。**强烈推荐您在生产环境下关闭错误信息。**

<a name="handling-errors"></a>
## 错误处理

默认情况下，`app/start/global.php` 文件包含一个针对所有异常的错误处理器：

	App::error(function(Exception $exception)
	{
		Log::error($exception);
	});

这是一个最基础的错误处理器。但是您可以根据需求指定更多的处理器。错误处理器根据它们能够处理的异常进行调用。比如，您可以创建一个只处理 `RuntimeException` 异常的错误处理器：

	App::error(function(RuntimeException $exception)
	{
		// Handle the exception...
	});

如果一个异常处理器返回一个响应，响应将被发送至浏览器，其他异常处理器将不会被调用：

	App::error(function(InvalidUserException $exception)
	{
		Log::error($exception);

		return 'Sorry! Something is wrong with this account!';
	});

为了监听 PHP 的 fatal 错误，您可以使用 `App::fatal` 函数：

	App::fatal(function($exception)
	{
		//
	});

<a name="http-exceptions"></a>
## HTTP异常

关于 HTTP 异常，指的是在客户端请求时发生的错误。这有可能是一个页面没有找到 (404)，一个没有通过认证的错误 (401) 甚至是一个500错误。为了返回这样一个响应，可以使用如下方法：

	App::abort(404, 'Page not found');

第一个参数是 HTTP 状态码，紧接着是一个您希望显示的定制的消息。

为了引发一个没有通过认证的401异常，使用如下方法：

	App::abort(401, 'You are not authorized.');

这些异常可以在请求生命周期的任何阶段被处理。

<a name="handling-404-errors"></a>
## 处理404错误

您可以注册一个异常处理器处理您应用程序中所有的 "404 Not Found" 错误，允许您返回定制的404错误页面：

	App::missing(function($exception)
	{
		return Response::view('errors.missing', array(), 404);
	});

<a name="logging"></a>
## 日志

Laravel 的日志工具提供了强大的 [Monolog](http://github.com/seldaek/monolog) 的简单的封装。默认情况下，Laravel 配置成为您的应用产生每天的日志，这些文件被存放在 `app/storage/logs` 目录下。您可以像下面这样向日志中写入信息：

	Log::info('This is some useful information.');

	Log::warning('Something could be going wrong.');

	Log::error('Something is really going wrong.');

日志工具根据在 [RFC 5424](http://tools.ietf.org/html/rfc5424) 定义了7个日志等级，分别是： **debug**, **info**, **notice**, **warning**, **error**, **critical** 以及 **alert**。

Monolog 有许多其他的处理器您可以用来产生日志，如果您愿意，您可以获取 Laravel 使用的底层的 Monolog 实例：

	$monolog = Log::getMonolog();

您也可以注册一个事件抓取传递给日志工具的所有消息：

**注册一个日志监听器**

	Log::listen(function($level, $message, $context)
	{
		//
	});
