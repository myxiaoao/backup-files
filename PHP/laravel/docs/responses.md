# 视图与响应

- [基本响应](#basic-responses)
- [跳转](#redirects)
- [视图](#views)
- [视图组件](#view-composers)
- [特殊的响应](#special-responses)

<a name="basic-responses"></a>
## 基本响应

**从路由中返回字符串**

	Route::get('/', function()
	{
		return 'Hello World';
	});

**创建定制的响应**

一个 `Response` 实例继承自 `Symfony\Component\HttpFoundation\Response` 类，提供了很多构建 HTTP 响应的函数。

	$response = Response::make($contents, $statusCode);

	$response->header('Content-Type', $value);

	return $response;

**在响应中添加 Cookie**

	$cookie = Cookie::make('name', 'value');

	return Response::make($content)->withCookie($cookie);

<a name="redirects"></a>
## 跳转

**返回一个跳转**

	return Redirect::to('user/login');

**返回一个带闪存数据的跳转**
	
	return Redirect::to('user/login')->with('message', 'Login Failed');

**返回一个命名路由的跳转**

	return Redirect::route('login');

**返回一个带参数的命名路由的跳转**

	return Redirect::route('profile', array(1));

**返回一个带命名参数的命名路由的跳转**

	return Redirect::route('profile', array('user' => 1));

**返回一个控制器函数的跳转**

	return Redirect::action('HomeController@index');

**返回一个带参数的控制器函数的跳转**

	return Redirect::action('UserController@profile', array(1));

***返回一个带命名参数的控制器函数的跳转**

	return Redirect::action('UserController@profile', array('user' => 1));

<a name="views"></a>
## 视图

视图通常包含您应用程序的 HTML 代码以及提供一种便利的方式把控制器从事务逻辑和显示逻辑中分离开来。视图文件存放在 `app/views` 目录。

一个简单的视图可以像下面这样：

	<!-- View stored in app/views/greeting.php -->

	<html>
		<body>
			<h1>Hello, <?php echo $name; ?></h1>
		</body>
	</html>

这个视图可以像这样返回至显示器：

	Route::get('/', function()
	{
		return View::make('greeting', array('name' => 'Taylor'));
	});

传给 `View::make` 的第二个参数是一个数据的数组，用于传递视图中所需要的变量。

**传递数据至视图**

	$view = View::make('greeting', $data);

	$view = View::make('greeting')->with('name', 'Steve');

在上面的例子中，变量 `$name` 将在视图中可以访问，并且包含 `Steve` 值。

您也可以向所有视图共享一个变量的值：

	View::share('name', 'Steve');

**传递一个子视图到视图**

有些时候，您可能希望传递一个视图到另一个视图。比如，有一个存放在 `app/views/child/view.php` 的子视图，我们可以像这样转递它至另一个视图：

	$view = View::make('greeting')->nest('child', 'child.view');

	$view = View::make('greeting')->nest('child', 'child.view', $data);

子视图将在父视图中呈现：

	<html>
		<body>
			<h1>Hello!</h1>
			<?php echo $child; ?>
		</body>
	</html>

<a name="view-composers"></a>
## 视图组件

视图组件是当一个视图建立的时候的回调函数或者类方法。如果您希望在视图建立的时候想绑定数据至一个指定的视图，一个视图组件可以被组织在一个单独的文件，视图组件可以实现 "视图模型" 或 "视图解析" 这样的功能。

**定义一个视图组件**

	View::composer('profile', function($view)
	{
		$view->with('count', User::count());
	});

现在，每次 `profile` 视图建立的时候，`count` 数据将被附加到该视图。

您也可以一次性附加一个视图组件到很多视图：

    View::composer(array('profile','dashboard'), function($view)
    {
        $view->with('count', User::count());
    });

如果您更希望使用基于类的视图组件，将受益于 [IoC 容器](/docs/ioc)，您可以这样做：

	View::composer('profile', 'ProfileComposer');

一个视图组件类可以像下面这样定义：

	class ProfileComposer {

		public function compose($view)
		{
			$view->with('count', User::count());
		}

	}

注意没有惯例指明组件类的存放位置。您可以在任何地方存放它们，只要它们能够在 `composer.json` 中使用自动加载功能加载它们。

<a name="special-responses"></a>
## 特殊的响应

**创建一个 JSON 响应**

	return Response::json(array('name' => 'Steve', 'state' => 'CA'));

**创建一个 JSONP 响应**

	return Response::json(array('name' => 'Steve', 'state' => 'CA'))->setCallback(Input::get('callback'));

**创建一个文件下载的响应**

	return Response::download($pathToFile);

	return Response::download($pathToFile, $name, $headers);
