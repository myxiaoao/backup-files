# 请求与输入

- [基本输入](#basic-input)
- [Cookies](#cookies)
- [旧输入](#old-input)
- [文件](#files)
- [请求信息](#request-information)

<a name="basic-input"></a>
## 基本输入

您可以通过几个简单的函数获取用户的输入。您不需要担心请求所使用的 HTTP 方法，因为所有方式的输入都可以通过相同的方式获取。

**获取一个输入变量**

	$name = Input::get('name');

**当输入变量不存在时获取一个默认值**

	$name = Input::get('name', 'Sally');

**检查一个输入变量是否存在**

	if (Input::has('name'))
	{
		//
	}

**获取请求的所有输入变量**

	$input = Input::all();

**获取请求的部分输入变量**

	$input = Input::only('username', 'password');

	$input = Input::except('credit_card');

一些 JavaScript 类库比如 Backbone 会以 JSON 格式向应用发送请求，您可以像往常一样通过 `Input::get` 获取数据。

<a name="cookies"></a>
## Cookies

所有由 Laravel 框架产生的 Cookie 都是被加密并根据一个认证码签名，这意味着如果客户改变了它，将变得不可用。

**获取一个 Cookie 变量**

	$value = Cookie::get('name');

**在响应中附加一个 Cookie**

	$response = Response::make('Hello World');

	$response->withCookie(Cookie::make('name', 'value', $minutes));

**创建一个永不过期的 Cookie**

	$cookie = Cookie::forever('name', 'value');

<a name="old-input"></a>
## 旧输入

您可能希望保持输入变量直到下一次请求。比如，您需要在验证表单错误后重新填充表单。

**闪存所有 Input 变量到 Session**

	Input::flash();

**闪存部分 Input 变量到 Session**

	Input::flashOnly('username', 'email');

	Input::flashExcept('password');

因为您经常希望在跳转到前一个页面的时候闪存 Input 变量，您可以在跳转的使用链式的方法：

	return Redirect::to('form')->withInput();

	return Redirect::to('form')->withInput(Input::except('password'));

> **注意:** 您可能希望使用 [Session](/docs/session) 类来根据请求闪存其他数据。

**获取旧输入**

	Input::old('username');

<a name="files"></a>
## 文件

**获取一个已上传的文件**

	$file = Input::file('photo');

**检查一个文件是否已上传**

	if (Input::hasFile('photo'))
	{
		//
	}

通过 `file` 函数返回的对象是一个 `Symfony\Component\HttpFoundation\File\UploadedFile` 类的实例，这个类扩展了 PHP 的 `SplFileInfo` 类，并且提供了众多与文件交互的方法。

**转移一个已上传的文件**

	Input::file('photo')->move($destinationPath);

	Input::file('photo')->move($destinationPath, $fileName);

**获取一个已上传文件的目录**

	$path = Input::file('photo')->getRealPath();

**获取一个已上传文件的大小**

	$size = Input::file('photo')->getSize();

**获取一个已上传文件的 MIME 类型**

	$mime = Input::file('photo')->getMimeType();

<a name="request-information"></a>
## 请求信息

`Request` 类扩展了 `Symfony\Component\HttpFoundation\Request` 类并提供了检查 HTTP 请求的众多函数，下面是一些常用代码：

**获取请求的 URI**

	$uri = Request::path();

**检查一个请求的路径是否匹配指定的模式**

	if (Request::is('admin/*'))
	{
		//
	}

**获取请求的 URL**

	$url = Request::url();

**获取请求的指定的 URI 段**

	$segment = Request::segment(1);

**获取请求的头部**

	$value = Request::header('Content-Type');

**从 $_SERVER 中获取变量**

	$value = Request::server('PATH_INFO');

**检查一个请求是否使用 AJAX**

	if (Request::ajax())
	{
		//
	}

**检查一个请求是否通过 HTTPS**

	if (Request::secure())
	{
		//
	}
