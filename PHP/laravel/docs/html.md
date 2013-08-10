# 表单与HTML

- [创建表单](#opening-a-form)
- [CSRF保护](#csrf-protection)
- [表单模型绑定](#form-model-binding)
- [标签](#labels)
- [文本框、多行文本框、密码框以及隐藏域](#text)
- [复选框和单选按钮](#checkboxes-and-radio-buttons)
- [文件上传域](#file-input)
- [下拉列表](#drop-down-lists)
- [按钮](#buttons)
- [定制宏](#custom-macros)

<a name="opening-a-form"></a>
## 创建表单

**创建一个表单**

	{{ Form::open(array('url' => 'foo/bar')) }}
		//
	{{ Form::close() }}

默认情况下，表单使用 `POST` 方法，您可以很容易使用另一种方法：

	echo Form::open(array('url' => 'foo/bar', 'method' => 'put'))

> **注意:** 因为 HTML 表单只支持 `POST` 和 `GET` 方法， `PUT` 和 `DELETE` 方法将通过自动添加一个 `_method` 隐藏域到表单的方式进行模拟。

您也可以通过指向命名路由或控制器函数打开一个表单：

	echo Form::open(array('route' => 'route.name'))

	echo Form::open(array('action' => 'Controller@method'))

您同样可以传递路由参数：

	echo Form::open(array('route' => array('route.name', $user->id)))

	echo Form::open(array('action' => array('Controller@method', $user->id)))

如果您的表单需要允许文件上传，请在数组参数中添加一个 `files` 选项：

	echo Form::open(array('url' => 'foo/bar', 'files' => true))

<a name="csrf-protection"></a>
## CSRF保护

Laravel 提供了一个简单的办法保护您的应用抵御跨域攻击。首先，一个随机的令牌添加在用户的 Session 中。无需劳作，这将自动完成。CSRF 令牌将将自动以隐藏域添加到表单中。如果你希望自己为这个隐藏域产生 HTML 代码，可以使用 `token` 函数:

**在表单中手动添加一个 CSRF 令牌**

	echo Form::token();

**在一个路由上附加 CSRF 过滤器**

	Route::post('profile', array('before' => 'csrf', function()
	{
		//
	}));

<a name="form-model-binding"></a>
## 表单模型绑定

经常您希望基于一个模型的内容填充一个表单。可以使用 `Form::model` 实现这个功能： 

**打开一个模型表单**

	echo Form::model($user, array('route' => array('user.update', $user->id)))

现在当您生成一个表单元素，比如一个文本输入框，模型中与此相同名字的的值将被设置为文本框的值。比如，对于一个命名为 `email` 的文本框，用户模型的 `email` 属性的值将被设为它的值。而且，还有更多。如果在闪存中有符合输入名的值，将优先于模型中的值。所以优先级应该是这个样子：

1. 闪存中的值(旧输入)
2. 输入值
3. 模型中属性的值

这将允许我们快速构建表单，不仅能够绑定模型的值，还可以在验证出错的时候轻松地重新填充表单。

> **注意:** 当使用 `Form::model` 的时候，请确认已使用 `Form::close` 关闭您的表单！

<a name="labels"></a>
## 标签

**创建一个标签元素**

	echo Form::label('email', 'E-Mail Address');

**指定其他 HTML 属性**

	echo Form::label('email', 'E-Mail Address', array('class' => 'awesome'));

> **注意:** 创建一个标签元素后，您创建的任何与标签元素同名的表单元素将自动获取一个与名字相同的ID。

<a name="text"></a>
## 文本框、多行文本框、密码框以及隐藏域

**创建一个文本框**

	echo Form::text('username');

**指定默认值**

	echo Form::text('email', 'example@gmail.com');

> **注意:** *hidden* 和 *textarea* 方法拥有和 *text* 方法一样的形式。

**创建一个密码框**

	echo Form::password('password');
	
**创建其他输入框**

	echo Form::date($name, $value = null, $attributes = array());
	echo Form::email($name, $value = null, $attributes = array());
	echo Form::file($name, $attributes = array());
	echo Form::number($name, $value = null, $attributes = array());
	echo Form::search($name, $value = null, $attributes = array());
	echo Form::telephone($name, $value = null, $attributes = array());
	echo Form::url($name, $value = null, $attributes = array());
	
<a name="checkboxes-and-radio-buttons"></a>
## 复选框和单选按钮

**创建一个复选框或单选按钮**

	echo Form::checkbox('name', 'value');
	
	echo Form::radio('name', 'value');

**创建一个被选中的复选框或单选按钮**

	echo Form::checkbox('name', 'value', true);
	
	echo Form::radio('name', 'value', true);

<a name="file-input"></a>
## 文件上传域

**创建一个文件上传域**

	echo Form::file('image');

<a name="drop-down-lists"></a>
## 下拉列表

**创建一个下拉列表**

	echo Form::select('size', array('L' => 'Large', 'S' => 'Small'));

**创建一个有默认选中值的下拉列表**

	echo Form::select('size', array('L' => 'Large', 'S' => 'Small'), 'S');

**创建一个分组的下拉列表**

	echo Form::select('animal', array(
		'Cats' => array('leopard' => 'Leopard'),
		'Dogs' => array('spaniel' => 'Spaniel'),
	));

<a name="buttons"></a>
## 按钮

**创建一个提交按钮**

	echo Form::submit('Click Me!');

> **注意:** 需要创建一个按钮吗？尝试使用 *button* 方法，它拥有和 *submit* 一样的形式。

<a name="custom-macros"></a>
## 定制宏

自定义一个定制的表单元素的助手函数也被称作为 "macros" 是很简单的事情。这里将展示如何实现。首先，使用一个给定的名字以及一个闭包函数注册一个宏：

**注册一个表单宏**

	Form::macro('myField', function()
	{
		return '<input type="awesome">';
	});

现在您可以通过它的名字调用这个宏：

**调用一个定制的表单宏**

	echo Form::myField();
