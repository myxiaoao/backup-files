# 控制器

- [基本控制器](#basic-controllers)
- [控制器过滤器](#controller-filters)
- [RESTful控制器](#restful-controllers)
- [资源控制器](#resource-controllers)
- [处理未定义的方法](#handling-missing-methods)

<a name="basic-controllers"></a>
## 基本控制器

除了在 `routes.php` 文件中定义您的路由逻辑，您也可能希望使用控制器组织这些逻辑。控制器可以根据相关的路由逻辑对路由进行分组，并且能够享用框架的高级功能比如自动 [依赖注入](/docs/ioc)。

控制器通常存放在 `app/controllers` 目录，并且这个目录默认在 `composer.json` 文件的 `classmap` 选项中注册。

下面是一个基本控制器类的例子：

	class UserController extends BaseController {

		/**
		 * Show the profile for the given user.
		 */
		public function showProfile($id)
		{
			$user = User::find($id);

			return View::make('user.profile', array('user' => $user));
		}

	}

所有的控制器应该继承 `BaseController` 类。`BaseController` 类也存放在 `app/controllers` 目录下，可以被用来存放共享的控制器逻辑。`BaseController` 类继承自框架的 `Controller` 类，现在我们可以像这样为控制器添加路由：

	Route::get('user/{id}', 'UserController@showProfile');

如果您使用 PHP 命名空间嵌套或组织您的控制器，在定义路由的时候可以使用控制类的全称：

	Route::get('foo', 'Namespace\FooController@method');

您也可以控制器的路由指定名字：

	Route::get('foo', array('uses' => 'FooController@method',
											'as' => 'name'));

使用 `URL::action` 函数从控制器函数生成 URL：

	$url = URL::action('FooController@method');

使用 `currentRouteAction` 函数获取控制器函数的名字：

	$action = Route::currentRouteAction();

<a name="controller-filters"></a>
## 控制器过滤器

[过滤器](/docs/routing#route-filters) 可以用来像常规路由一样指定控制器路由：

	Route::get('profile', array('before' => 'auth',
				'uses' => 'UserController@showProfile'));

但是，您也可以在控制器中指定过滤器：

	class UserController extends BaseController {

		/**
		 * Instantiate a new UserController instance.
		 */
		public function __construct()
		{
			$this->beforeFilter('auth');

			$this->beforeFilter('csrf', array('on' => 'post'));

			$this->afterFilter('log', array('only' =>
								array('fooAction', 'barAction')));
		}

	}

您也可以使用闭包在控制器中指定过滤器：

	class UserController extends BaseController {

		/**
		 * Instantiate a new UserController instance.
		 */
		public function __construct()
		{
			$this->beforeFilter(function()
			{
				//
			});
		}

	}

<a name="restful-controllers"></a>
## RESTful控制器

Laravel 允许您按照简单的，REST 命名惯例轻松的定义一条路由来处理控制器的所有函数。首先，使用 `Route::controller` 定义路由：

**定义一个 RESTful 控制器**

	Route::controller('users', 'UserController');

`controller` 函数接受两个参数。第一个参数是控制器处理的基准 URI。下面，在控制器中添加函数，需要添加 HTTP 访问方法的前缀：

	class UserController extends BaseController {

		public function getIndex()
		{
			//
		}

		public function postProfile()
		{
			//
		}

	}

`index` 函数将作为控制器处理根 URI 访问的响应，在这个例子中是 `users`。

如果您的控制器函数包含多个单词，您可以通过在 URI 中使用下划线的语法来访问这个函数。比如，下面这个在 `UserController` 的控制器函数将会作为 `users/admin-profile` URI 的响应：

	public function getAdminProfile() {}

<a name="resource-controllers"></a>
## 资源控制器

资源控制器使得基于资源构建 RESTful 控制器变得简单。比如，您可能希望创建一个控制器管理应用程序中的照片。在 Artisan 命令行中使用 `controller:make` 命令以及 `Route::resource 函数，我们可以很快的创建这样一个控制器。

通过命令行创建控制器，请执行以下命令：

	php artisan controller:make PhotoController

现在注册一个基于资源的路由到控制器：

	Route::resource('photo', 'PhotoController');

这个简单的路由声明创建了许多路由来处理对照片资源的 RESTful 访问。同样的，生成的控制器已经拥有了对各个访问处理的函数。

**资源控制器处理的访问**

Verb      | Path                  | Action       | Route Name
----------|-----------------------|--------------|---------------------
GET       | /resource             | index        | resource.index
GET       | /resource/create      | create       | resource.create
POST      | /resource             | store        | resource.store
GET       | /resource/{id}        | show         | resource.show
GET       | /resource/{id}/edit   | edit         | resource.edit
PUT/PATCH | /resource/{id}        | update       | resource.update
DELETE    | /resource/{id}        | destroy      | resource.destroy

有些时候，您只需要处理一部分资源访问：

	php artisan controller:make PhotoController --only=index,show

	php artisan controller:make PhotoController --except=index

并且，您也可以在路由中指明一部分资源访问的处理：

	Route::resource('photo', 'PhotoController',
					array('only' => array('index', 'show')));

<a name="handling-missing-methods"></a>
## 处理未定义的方法

在控制中没有找到匹配函数的时候，一个全捕捉函数可以被定义用来调用。这个函数应该被命名为 `missingMethod` 并且接受请求的参数数组做为它的唯一参数：

**定义一个全捕捉函数**

	public function missingMethod($parameters)
	{
		//
	}