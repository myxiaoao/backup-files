# 路由

- [基本路由](#basic-routing)
- [路由参数](#route-parameters)
- [路由过滤器](#route-filters)
- [命名路由](#named-routes)
- [路由组](#route-groups)
- [子域名路由](#sub-domain-routing)
- [路由前缀](#route-prefixing)
- [路由模型绑定](#route-model-binding)
- [引发404错误](#throwing-404-errors)
- [路由至控制器](#routing-to-controllers)

<a name="basic-routing"></a>
## 基本路由

您的应用程序的绝大多数路由将在 `app/routes.php` 文件中定义。Laravel 中最简单的路由由一个 URI 和一个闭包调用组成。

**基本 GET 路由**

	Route::get('/', function()
	{
		return 'Hello World';
	});

**基本 POST 路由**

	Route::post('foo/bar', function()
	{
		return 'Hello World';
	});

**注册一个路由以响应所有 HTTP 方法**

	Route::any('foo', function()
	{
		return 'Hello World';
	});

**强制一个路由必须通过 HTTPS 访问**

	Route::get('foo', array('https', function()
	{
		return 'Must be over HTTPS';
	}));

经常您需要根据路由产生 URLs，您可以通过使用 `URL::to` 方法：

	$url = URL::to('foo');

<a name="route-parameters"></a>
## 路由参数

	Route::get('user/{id}', function($id)
	{
		return 'User '.$id;
	});

**可选的路由参数**

	Route::get('user/{name?}', function($name = null)
	{
		return $name;
	});

**带默认值的可选的路由参数**

	Route::get('user/{name?}', function($name = 'John')
	{
		return $name;
	});

**带正则表达式约束的路由**

	Route::get('user/{name}', function($name)
	{
		//
	})
	->where('name', '[A-Za-z]+');

	Route::get('user/{id}', function($id)
	{
		//
	})
	->where('id', '[0-9]+');

<a name="route-filters"></a>
## 路由过滤器

路由过滤器提供了一种限制访问指定路由的简单的方法，这在您需要为您的站点创建需要认证区域的时候非常有用。Laravel 框架中包含了一些路由过滤器，比如 `auth` 过滤器、`auth.basic` 过滤器、`guest` 过滤器、以及 `csrf` 过滤器。它们被存放在 `app/filters.php` 文件中。

**定义一个路由过滤器**

	Route::filter('old', function()
	{
		if (Input::get('age') < 200)
		{
			return Redirect::to('home');
		}
	});

如果一个响应从一个路由过滤器中返回，这个响应即被认为是这个请求的响应，路由将不被执行，任何关于这个路由的 `after` 过滤器也将被取消执行。

**为一个路由指定一个路由过滤器**

	Route::get('user', array('before' => 'old', function()
	{
		return 'You are over 200 years old!';
	}));

**为一个路由指定多个路由过滤器**

	Route::get('user', array('before' => 'auth|old', function()
	{
		return 'You are authenticated and over 200 years old!';
	}));

**指定路由过滤器参数**

	Route::filter('age', function($route, $request, $value)
	{
		//
	});

	Route::get('user', array('before' => 'age:200', function()
	{
		return 'Hello World';
	}));

当路由过滤器接收到作为第三个参数的响应 `$response`:

	Route::filter('log', function($route, $request, $response, $value)
	{
		//
	});

**基本路由过滤器的模式**

您可能希望根据 URI 为一组路由指定过滤器。

	Route::filter('admin', function()
	{
		//
	});

	Route::when('admin/*', 'admin');

在上面的例子中，`admin` 过滤器将应用带所有以 `admin/` 开头的路由。星号作为一个通配符，将适配到所有字符的组合。

您也可以通过指定 HTTP 方法约束模式过滤器：

	Route::when('admin/*', 'admin', array('post'));

**过滤器类**

对于高级的过滤器，您可以使用一个类代替闭包函数。因为过滤器类是位于应用程序之外的 [IoC 容器](/docs/ioc)，您能够在过滤器中使用依赖注入，更易于测试。

**定义一个过滤器类**

	class FooFilter {

		public function filter()
		{
			// Filter logic...
		}

	}

**注册一个基于类的过滤器**

	Route::filter('foo', 'FooFilter');

<a name="named-routes"></a>
## 命名路由

命名路由在更易于在生成跳转或 URLs 时指定路由。您可以像这样为路由指定一个名字：

	Route::get('user/profile', array('as' => 'profile', function()
	{
		//
	}));

您也可以为控制器的方法指定路由名字：

	Route::get('user/profile', array('as' => 'profile', 'uses' => 'UserController@showProfile'));

现在您在生成 URLs 或跳转的时候使用路由的名字：

	$url = URL::route('profile');

	$redirect = Redirect::route('profile');

您可以使用 `currentRouteName` 方法获取一个路由的名字：

	$name = Route::currentRouteName();

<a name="route-groups"></a>
## 路由组

有些时候您可能希望应用过滤器到一组路由。您不必要为每个路由指定过滤器，可以使用路由组：

	Route::group(array('before' => 'auth'), function()
	{
		Route::get('/', function()
		{
			// Has Auth Filter
		});

		Route::get('user/profile', function()
		{
			// Has Auth Filter
		});
	});

<a name="sub-domain-routing"></a>
## 子域名路由

Laravel 路由也能够处理通配符的子域名，并且从域名中获取通配符参数：

**注册子域名路由**

	Route::group(array('domain' => '{account}.myapp.com'), function()
	{

		Route::get('user/{id}', function($account, $id)
		{
			//
		});

	});
<a name="route-prefixing"></a>
## 路由前缀

一组路由可以通过在属性数组中使用 `prefix` 选项为路由组添加前缀：

**为路由组添加前缀**

	Route::group(array('prefix' => 'admin'), function()
	{

		Route::get('user', function()
		{
			//
		});

	});

<a name="route-model-binding"></a>
## 路由模型绑定

模型绑定提供了一个简单的方法向路由中注入模型。比如，不仅注入一个用户的 ID，您可以根据指定的 ID 注入整个用户模型实例。首先使用 `Route::model` 方法指定所需要的模型：


**为模型绑定一个变量**

	Route::model('user', 'User');

然后, 定义一个包含 `{user}` 参数的路由:

	Route::get('profile/{user}', function(User $user)
	{
		//
	});

因为我们已经绑定 `{user}` 参数到 `User` 模型，一个 `User` 实例将被注入到路由中。因此，比如一个 `profile/1` 的请求将注入一个 ID 为 1 的 `User` 实例。

> **注意:** 如果在数据库中没有找到这个模型实例，将引发404错误。

如果您希望指定您自己定义的没有找到的行为，您可以为 `model` 方法传递一个闭包作为第三个参数：

	Route::model('user', 'User', function()
	{
		throw new NotFoundException;
	});

有时您希望使用自己的方法处理路由参数，可以使用 `Route::bind` 方法：

	Route::bind('user', function($value, $route)
	{
		return User::where('name', $value)->first();
	});

<a name="throwing-404-errors"></a>
## 引发404错误

有两种方法在路由中手动触发一个404错误。首先，您可以使用 `App::abort` 方法：

	App::abort(404);

其次，您可以抛出一个 `Symfony\Component\HttpKernel\Exception\NotFoundHttpException` 的实例。

更多关于处理404异常和为这些错误使用使用自定义响应的信息可以在 [错误](/docs/errors#handling-404-errors) 章节中找到。

<a name="routing-to-controllers"></a>
## 路由至控制器

Laravel 不仅允许您路由至闭包，也可以路由至控制器类，甚至允许创建 [资源控制器](/docs/controllers#resource-controllers).

更多信息请访问 [控制器](/docs/controllers) 文档。
