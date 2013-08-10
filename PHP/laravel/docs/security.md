# 安全

- [配置](#configuration)
- [存储密码](#storing-passwords)
- [认证用户](#authenticating-users)
- [手动登录用户](#manually)
- [路由保护](#protecting-routes)
- [HTTP基本认证](#http-basic-authentication)
- [密码提醒和重置](#password-reminders-and-reset)
- [加密](#encryption)

<a name="configuration"></a>
## 配置

Laravel旨在使得身份验证变得非常简单。事实上，几乎所有一切都已经为您配置好。验证配置文件位于 `app/config/auth.php`，其中包含几个详尽注释的选项可用于调整认证的行为。

默认情况下，Laravel包含一个 `User` 模型在 `app/models` 目录，可在默认的Eloquent认证驱动下使用。请记住当构建这个模型的结构时请确认密码域最少60个字符。

如果您的应用程序不打算使用Eloquent，您可以使用 `database` 认证驱动，该驱动使用Laravel查询生成器。

<a name="storing-passwords"></a>
## 存储密码

Laravel的 `Hash` 类提供了安全的Bcrypt散列:

**使用Bcrypt散列加密一个密码**

	$password = Hash::make('secret');

**根据一个哈希值验证一个密码**

	if (Hash::check('secret', $hashedPassword))
	{
		// The passwords match...
	}

**检查是否一个密码需要被再次加密**

	if (Hash::needsRehash($hashed))
	{
		$hashed = Hash::make('secret');
	}

<a name="authenticating-users"></a>
## 认证用户

使一个用户登录到您的应用程序，您可以使用 `Auth::attempt` 函数：

	if (Auth::attempt(array('email' => $email, 'password' => $password)))
	{
		return Redirect::intended('dashboard');
	}

注意 `email` 不是一个必需的选项，它只是用于示例。您应该使用数据库中任何代表用户名的字段的名字。`Redirect::intended` 函数将用户重定向到他们之前尝试访问的但被认证过滤器拦截的URL。一个备选的URI给这个函数用于防止目的地址是不可用的。

当 `attempt` 函数被调用，`auth.attempt` [事件](/docs/events)被触发。如果认证尝试是成功的，用户将登录，`auth.login` 事件也将被触发。

确定一个用户是否已经登录到您的应用程序，您可以使用 `check` 函数。

**确认一个用户是否被认证**

	if (Auth::check())
	{
		// The user is logged in...
	}

如果您希望在应用程序中提供“记住我”这样的功能，您可以传递 `true` 作为第二个参数到 `attempt` 函数，它将无限期保持用户认证（或者直到他们手动注销）：

**认证用户并“记住”他们**

	if (Auth::attempt(array('email' => $email, 'password' => $password), true))
	{
		// The user is being remembered...
	}

**注意:** 如果 `attempt` 函数返回 `true`，用户将被认为已登录到应用程序。

您也可以添加额外的条件来认证查询：

**带条件的用户认证**

    if (Auth::attempt(array('email' => $email, 'password' => $password, 'active' => 1)))
    {
        // The user is active, not suspended, and exists.
    }

一旦用户通过认证，您可以访问用户模型/记录：

**获取登录的用户**

	$email = Auth::user()->email;

可以使用 `loginUsingId` 函数根据他们的ID记录用户登录到应用程序：

	Auth::loginUsingId(1);

`validate` 函数允许您验证用户凭证而不实际上让他们登录到应用程序：

**验证用户凭证而不登录**

	if (Auth::validate($credentials))
	{
		//
	}

您也可以使用 `once` 函数为单次请求让用户登录到到应用程序。没有会话或Cookie会被使用。

**为单次请求让用户登录**

	if (Auth::once($credentials))
	{
		//
	}

**让用户从应用程序中注销**

	Auth::logout();

<a name="manually"></a>
## 手动登录用户

如果您需要一个已存在的用户实例登录到您的应用程序，您可以简单地对这个实例调用 `login` 函数：

	$user = User::find(1);

	Auth::login($user);

这相当于使用 `attempt` 函数使用户通过凭证进行登录。

<a name="protecting-routes"></a>
## 路由保护

路由过滤器可用来允许经过认证的用户访问指定的路由。Laravel 默认提供了 `auth` 过滤器，它定义在 `app/filters.php` 文件中。

**保护一个路由**

	Route::get('profile', array('before' => 'auth', function()
	{
		// Only authenticated users may enter...
	}));

### CSRF保护

Laravel 提供了一种简单的方法抵御跨站请求伪造来保护您的应用程序。

**在表单中插入CSRF令牌**

    <input type="hidden" name="_token" value="<?php echo csrf_token(); ?>">

**验证提交的CSRF令牌**

    Route::post('register', array('before' => 'csrf', function()
    {
        return 'You gave a valid CSRF token!';
    }));

<a name="http-basic-authentication"></a>
## HTTP基本认证

HTTP基本认证提供了一种快速的方法认证用户，而无需设置一个专门的登录页面。首先，在路由中附加 `auth.basic` 过滤器：

**使用HTTP基本认证保护路由**

	Route::get('profile', array('before' => 'auth.basic', function()
	{
		// Only authenticated users may enter...
	}));

默认情况下，`basic` 过滤器在认证中将使用用户记录的 `email` 字段。如果您希望使用另一个字段，您可以作为第一个参数传递给 `basic` 函数：

	return Auth::basic('username');

您也可以使用HTTP基本认证而无需在回话中设置一个用户身份标识 Cookie，这在 API 认证中特别有用。为此，定义一个返回 `onceBasic` 函数的过滤器：

**建立一个无状态的HTTP基本认证过滤器**

	Route::filter('basic.once', function()
	{
		return Auth::onceBasic();
	});

<a name="password-reminders-and-reset"></a>
## 密码提醒和重置

### 发送密码提醒

大多数应用程序提供了提供了一种重置已忘记密码的方法。Laravel 提供了简单的方法用来发送提醒以及密码重置，而不是强迫您在每个应用程序中重新实现。首先，请确认 `User` 模型实现了 `Illuminate\Auth\Reminders\RemindableInterface` 接口。当然，框架自带的 `User` 模型已经实现了此接口。

**实现RemindableInterface接口**

	class User extends Eloquent implements RemindableInterface {

		public function getReminderEmail()
		{
			return $this->email;
		}

	}

接下来，必选创建一张表用来保存密码重置令牌。可以使用 `auth:reminders` Artisan 命令对这张表生成一个迁移：

**生成Reminder表迁移**

	php artisan auth:reminders

	php artisan migrate

可以使用 `Password::remind` 函数发送一个密码提醒：

**发送一个密码提醒**

	Route::post('password/remind', function()
	{
		$credentials = array('email' => Input::get('email'));

		return Password::remind($credentials);
	});

注意传递到 `remind` 函数的参数与 `Auth::attempt` 函数类似。这个函数将检索用户并通过电子邮件发送一个密码重置的连接。电子邮件视图将被传入一个 `token` 变量用于构建重置密码表单的链接。`user` 对象也将传入到视图。

> **注意:** 您可以通过改变 `auth.reminder.email` 配置选项改变电子邮件消息的视图。当然一个默认的视图可以立即使用。

您可以通过一个闭包作为 `remind` 函数的第二个参数发送个用户修改消息实例：

	return Password::remind($credentials, function($message, $user)
	{
		$message->subject('Your Password Reminder');
	});

您可能已注意到我们从一个路由中直接返回 `remind` 函数的结果。默认情况下，`remind` 函数将返回当前 URI 的重定向。如果在重置密码时一个错误发生，一个 `error` 变量将闪存到会话，还有一个 `reason`，可用于从 `reminders` 语言文件中提取一个语言行。如果密码重置成功，一个 `success` 变量将闪存到会话。所以，您的密码重置表单视图可能类似这样：

	@if (Session::has('error'))
		{{ trans(Session::get('reason')) }}
	@elseif (Session::has('success'))
		An e-mail with the password reset has been sent.
	@endif

	<input type="text" name="email">
	<input type="submit" value="Send Reminder">

### 密码重置

一旦用户从提醒邮件中点击了重置密码的链接，他们将被引导到一个包含隐藏域 `token` 以及密码 `password` 和 密码确认 `password_confirmation`的表单。下面是一个用于密码重置表单的路由的示例：

	Route::get('password/reset/{token}', function($token)
	{
		return View::make('auth.reset')->with('token', $token);
	});

并且，一个密码重置表单看起来像这样：

	@if (Session::has('error'))
		{{ trans(Session::get('reason')) }}
	@endif

	<input type="hidden" name="token" value="{{ $token }}">
	<input type="text" name="email">
	<input type="password" name="password">
	<input type="password" name="password_confirmation">

再次注意，我们使用会话显示重置密码时框架检测到的任何错误。接下来，我们可以定义一个 `POST` 路由处理密码重置：

	Route::post('password/reset/{token}', function()
	{
		$credentials = array('email' => Input::get('email'));

		return Password::reset($credentials, function($user, $password)
		{
			$user->password = Hash::make($password);

			$user->save();

			return Redirect::to('home');
		});
	});

如果密码重置成功，`User` 实例和密码将被传递给您的闭包，允许您执行保存操作。然后，您可能从 `reset` 函数返回的闭包中返回一个重定向或任何其他类型的响应。注意 `reset` 函数对请求自动检查一个有效的 `token`、有效的身份以及匹配的密码。

同时，类似与 `remind` 函数，如果在重置密码时一个错误发生，`reset` 函数将返回一个重定向到当前的 URI，附带一个 `error` 以及 `reason`。

<a name="encryption"></a>
## 加密

Laravel 通过 PHP mcrypt 扩展使用强大的 AES-256 加密。

**加密**

	$encrypted = Crypt::encrypt('secret');

> **注意:** 请确认在 `app/config/app.php` 文件的 `key` 选项中设置一个32个字符长度的随机字符串。否则，加密的值将不是安全的。

**解密**

	$decrypted = Crypt::decrypt($encryptedValue);

您也可以设置加密所使用的算法和模式：

**设置算法和模式**

	Crypt::setMode('crt');

	Crypt::setCipher($cipher);
