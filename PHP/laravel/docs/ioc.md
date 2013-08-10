# IoC容器

- [简介](#introduction)
- [基本用法](#basic-usage)
- [自动分辨](#automatic-resolution)
- [实际使用](#practical-usage)
- [服务提供商](#service-providers)
- [容器事件](#container-events)

<a name="introduction"></a>
## 简介

Laravel 的反转控制容器是用来管理类依赖关系的强大的工具。依赖注入是一种删除硬编码类依赖的方法。相反，依赖关系将在运行时被注入，为依赖关系的实现能够轻易的改变提供更大的灵活性。

对构建一个大型的应用，理解 Laravel IoC 容器和理解 Laravel 核心代码本身一样是有必要的。

<a name="basic-usage"></a>
## 基本用法

有两种方式 IoC 容器能够解决依赖关系：通过闭包回调函数或以及自动分辨。我们首先将探讨闭包回调函数。首先，一个类型可以在容器内绑定：

**在容器内绑定一个类型**

	App::bind('foo', function($app)
	{
		return new FooBar;
	});

**在容器内解析一个类型**

	$value = App::make('foo');

当 `App::make` 函数被调用，闭包回调函数将被执行并返回结果。

有些时候，您可能希望在容器内绑定一些只被一次解析的内容，并且相同的实例应该在随后的调用中返回到容器：

**绑定一个共享类型的容器**

	App::singleton('foo', function()
	{
		return new FooBar;
	});

您可以使用 `instance` 函数绑定一个现有的对象实例到容器：

**绑定一个已有的实现到容器**

	$foo = new Foo;

	App::instance('foo', $foo);

<a name="automatic-resolution"></a>
## 自动分辨

IoC 容器足够强大，能在许多场景中哦解析类而不需要任何配置。比如：

**解析一个类**

	class FooBar {

		public function __construct(Baz $baz)
		{
			$this->baz = $baz;
		}

	}

	$fooBar = App::make('FooBar');

注意尽管我们并没有在容器中注册 FooBar 类，容器仍然可以解析这个类，甚至自动注入 `Baz` 的依赖关系。

当一个类型没有在容器中绑定，它将使用 PHP 的反射机制检查类并且读取构造函数的类型提示。利用这些信息，容器能够自动构建一个类的实例。

但是，在某些情况下，一个类可能依赖于一个接口的实现，而不是一个具体的类。在这中情况下，`App::bind` 函数必须被用来通知容器注入哪一个接口：

**绑定一个接口**

	App::bind('UserRepositoryInterface', 'DbUserRepository');

现在考虑下面的控制器；

	class UserController extends BaseController {

		public function __construct(UserRepositoryInterface $users)
		{
			$this->users = $users;
		}

	}

因为我们已经绑定 `UserRepositoryInterface` 到一个具体的类型，`DbUserRepository`将自动被注入到控制器当它被创建的时候。

<a name="practical-usage"></a>
## 实际使用

Laravel 提供了好几处场景使用 IoC 容器来为您的应用程序增加灵活性和可测试性。一个主要的例子是解析控制器。所有控制器通过 IoC 容器解析，意味着您能够在控制器构造函数中类型提示依赖关系，并且它们自动被注入。

**Type-Hinting 控制器依赖关系**

	class OrderController extends BaseController {

		public function __construct(OrderRepository $orders)
		{
			$this->orders = $orders;
		}

		public function getIndex()
		{
			$all = $this->orders->all();

			return View::make('orders', compact('all'));
		}

	}

在这个例子中，`OrderRepository` 类将自动注入到控制器。这意味着当 [单元测试](/docs/testing) 一个模拟，`OrderRepository` 可以被绑定到容器并且注入到控制器，允许无痛的数据库层交互。

[过滤器](/docs/routing#route-filters), [组件](/docs/responses#view-composers) 以及 [时间处理器](/docs/events#using-classes-as-listeners) 都可以被 IoC container 解析。当注册它们，给出需要被使用的名字：

**其他 IoC 使用的例子**

	Route::filter('foo', 'FooFilter');

	View::composer('foo', 'FooComposer');

	Event::listen('foo', 'FooHandler');

<a name="service-providers"></a>
## 服务提供商

服务提供商是一个很好的方式在一个地方组织相关的 IoC 的注册。把他们作为应用程序中引导组件的一种方式。在服务提供商中，您可以注册一个定制的认证驱动，使用 Ioc 容器注册应用程序的类库，甚至设置一个自定义的 Artisan 命令。

事实上，大多数 Laravel 的核心组件包含服务提供商。所有应用程序中已注册的服务提供商在 `app/config/app.php` 配置文件的 `providers` 数组中列出。

创建一个服务提供商，请继承 `Illuminate\Support\ServiceProvider` 类并定义一个 `register` 函数：

**定义一个服务提供商**

	use Illuminate\Support\ServiceProvider;

	class FooServiceProvider extends ServiceProvider {

		public function register()
		{
			$this->app->bind('foo', function()
			{
				return new Foo;
			});
		}

	}

注意在 `register` 函数中，应用程序的 IoC 容器通过 `$this->app` 属性是可用的。一旦您已经创建了一个提供者，并且准备在应用程序中注册，请添加它到 `app` 配置的 `providers` 数组。

您也可以在程序运行时通过 `App::register` 函数注册一个服务提供商：

**程序运行时注册一个服务提供商**

	App::register('FooServiceProvider');

<a name="container-events"></a>
## 容器事件

容器每次解析一个对象时将触发一个事件。您可以通过使用 `resolving` 函数监听这个事件：

**注册一个解析监听器**

	App::resolving(function($object)
	{
		//
	});

注意被解析的对象将被传递给回调函数。