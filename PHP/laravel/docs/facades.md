# 外观

- [简介](#introduction)
- [解释](#explanation)
- [实际应用](#practical-usage)
- [创建外观](#creating-facades)
- [模拟外观](#mocking-facades)

<a name="introduction"></a>
## 简介

Facades 为应用程序的 [IoC容器](/docs/ioc) 的类提供了一个静态的接口。Laravel 自带很多外观，也许您已经使用它们但并没有意识到。

有时候，您可能希望为您的应用程序和包创建您自己的外观，让我们探讨它们的概念、开发以及使用。

> **注意:** 强烈建议您非常熟悉 Laravel 的 [IoC容器](/docs/ioc) 之后再深入探讨外观。

<a name="explanation"></a>
## 解释

在 Laravel 应用程序的上下文环境中，外观是一个为来自容器的一个对象提供访问的类。实际工作是 `Facade` 类。Laravel 的外观和您创建的外观都继承自 `Facade` 类。

您的外观类只需要实现一个函数：`getFacadeAccessor`。正是这个 `getFacadeAccessor` 函数的功能定义了从容器中怎样解析。`Facade` 基础类使用 `__callStatic()` 魔方函数来推迟来自您的外观到解析对象的调用。

<a name="practical-usage"></a>
## 实际应用

在下面的例子中，调用了 Laravel 的缓存系统。通过浏览这段代码，人们可能认为静态类 `get` 被 `Cache` 类调用。

	$value = Cache::get('key');

但是，如果我们看看 `Illuminate\Support\Facades\Cache` 类，您会发现并没有静态方法 `get`：

	class Cache extends Facade {

		/**
		 * Get the registered name of the component.
		 *
		 * @return string
		 */
		protected static function getFacadeAccessor() { return 'cache'; }

	}

Cache 类继承自基类 `Facade` 并且定义了一个 `getFacadeAccessor()` 函数。记住，这个函数的工作就是返回 IoC 绑定的名字。

当一个用户在 `Cache` 外观中引用任何静态函数，Laravel 将从 IoC 容器解析 `cache` 绑定并针对该对象运行请求的函数（在这个例子中是 `get`）。

所以，我们的 `Cache::get` 调用将被像这样重写:

	$value = $app->make('cache')->get('key');

<a name="creating-facades"></a>
## 创建外观

为您的应用程序或包创建一个外观是简单的，您只需要三件事：

- 一个 IoC 绑定
- 一个外观类
- 一个外观别名的配置

让我们看一个例子，在这里，我们定义了一个 `PaymentGateway\Payment` 类：

	namespace PaymentGateway;

	class Payment {

		public function process()
		{
			//
		}

	}

我们需要能够从 IoC 容器中解析这个类。所以，让我们添加一个绑定：

	App::bind('payment', function()
	{
		return new \PaymentGateway\Payment;
	});

一个很好的地方注册这个绑定是创建一个新的名为 `PaymentServiceProvider` 的 [服务提供者](/docs/ioc#service-providers)，并且使用 `register` 添加这个绑定。然后您可以配置 Laravel 从 `app/config/app.php` 文件中加载您的服务提供商。

接下来，我们可以创建自己的外观类：

	use Illuminate\Support\Facades\Facade;

	class Payment extends Facade {

		protected static function getFacadeAccessor() { return 'payment'; }

	}

最后，如果我们愿意，我们可以为我们的外观在 `app/config/app.php` 配置文件中添加一个别名到 `aliases`  数组。现在，我们可以对 `Payment` 类的实例调用 `process` 函数。

	Payment::process();

<a name="mocking-facades"></a>
## 模拟外观

单元测试是一个重要的方面关于为什么外观能够以它们的方式工作。事实上，可测试性是外观存在的一个主要的原因。跟多信息，请参阅 [模拟外观](/docs/testing#mocking-facades) 部分的文档。
