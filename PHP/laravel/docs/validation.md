# 验证

- [基础使用](#basic-usage)
- [附带错误消息](#working-with-error-messages)
- [错误消息 & 视图](#error-messages-and-views)
- [可用的验证规则](#available-validation-rules)
- [定制错误消息](#custom-error-messages)
- [定制验证规则](#custom-validation-rules)

<a name="basic-usage"></a>
## 基本使用

Laravel 自带一个简单、方便的 `Validation` 类用关于验证数据以及获取错误消息。

**基础验证例子**

	$validator = Validator::make(
		array('name' => 'Dayle'),
		array('name' => 'required|min:5')
	);

传递给 `make` 函数的第一个参数是待验证的数据，第二个参数是对该数据需要应用的验证规则。

多个验证规则可以通过 "|" 字符进行隔开，或者作为数组的一个单独的元素。

**通过数组指定验证规则**

	$validator = Validator::make(
		array('name' => 'Dayle'),
		array('name' => array('required', 'min:5'))
	);

一旦一个 `Validator` 实例被创建，可以使用 `fails` （或者 `passes`）函数执行这个验证。

	if ($validator->fails())
	{
		// The given data did not pass validation
	}

如果验证失败，您可以从验证器中获取错误消息。

	$messages = $validator->messages();

您也可以使用 `failed` 函数得到不带错误消息的没有通过验证的规则的数组。

	$failed = $validator->failed();

**文件验证**

`Validator` 类提供了一些验证规则用于验证文件，比如 `size`、`mimes`等。在验证文件的时候，您可以和其他验证一样传递给验证器。

<a name="working-with-error-messages"></a>
## 附带错误消息

在一个 `Validator` 实例上调用 `messages` 函数之后，将会得到一个 `MessageBag` 实例，该实例拥有很多处理错误消息的方便的函数。

**获取一个域的第一个错误消息**

	echo $messages->first('email');

**获取一个域的全部错误消息**

	foreach ($messages->get('email') as $message)
	{
		//
	}

**获取全部域的全部错误消息**

	foreach ($messages->all() as $message)
	{
		//
	}

**检查一个域是否存在消息**

	if ($messages->has('email'))
	{
		//
	}

**以某种格式获取一条错误消息**

	echo $messages->first('email', '<p>:message</p>');

> **注意:** 默认情况下，消息将使用与 Bootstrap 兼容的语法进行格式化。

**以某种格式获取所有错误消息**

	foreach ($messages->all('<li>:message</li>') as $message)
	{
		//
	}

<a name="error-messages-and-views"></a>
## 错误消息 & 视图

一旦您执行了验证，您需要一种简单的方法向视图反馈错误消息。这在 Lavavel 中能够方便的处理。以下面的路由作为例子：

	Route::get('register', function()
	{
		return View::make('user.register');
	});

	Route::post('register', function()
	{
		$rules = array(...);

		$validator = Validator::make(Input::all(), $rules);

		if ($validator->fails())
		{
			return Redirect::to('register')->withErrors($validator);
		}
	});

注意当验证失败，我们使用 `withErrors` 函数把 `Validator` 实例传递给 Redirect。这个函数将刷新 Session 中保存的错误消息，使得在下次请求中能够可用。

然而，注意我们没有必要明确的在 GET 路由中绑定错误消息到路由。这是因为 Laravel 总会检查 Session 中的错误，并自动绑定它们到视图如果它们是可用的。**所以，对于每个请求，一个 `$errors` 变量在所有视图中总是可用的**，允许您方便的认为 `$errors` 总是被定义并可以安全使用的。`$errors` 变量将是一个 `MessageBag` 类的实例。

所以，在跳转之后，您可以在视图中使用自动绑定的 `$errors` 变量：

	<?php echo $errors->first('email'); ?>

<a name="available-validation-rules"></a>
## 可用的验证规则

下面是一个所有可用的验证规则的列表以及它们的功能：

- [Accepted](#rule-accepted)
- [Active URL](#rule-active-url)
- [After (Date)](#rule-after)
- [Alpha](#rule-alpha)
- [Alpha Dash](#rule-alpha-dash)
- [Alpha Numeric](#rule-alpha-num)
- [Before (Date)](#rule-before)
- [Between](#rule-between)
- [Confirmed](#rule-confirmed)
- [Date](#rule-date)
- [Date Format](#rule-date-format)
- [Different](#rule-different)
- [E-Mail](#rule-email)
- [Exists (Database)](#rule-exists)
- [Image (File)](#rule-image)
- [In](#rule-in)
- [Integer](#rule-integer)
- [IP Address](#rule-ip)
- [Max](#rule-max)
- [MIME Types](#rule-mimes)
- [Min](#rule-min)
- [Not In](#rule-not-in)
- [Numeric](#rule-numeric)
- [Regular Expression](#rule-regex)
- [Required](#rule-required)
- [Required If](#rule-required-if)
- [Required With](#rule-required-with)
- [Required Without](#rule-required-without)
- [Same](#rule-same)
- [Size](#rule-size)
- [Unique (Database)](#rule-unique)
- [URL](#rule-url)

<a name="rule-accepted"></a>
#### accepted

验证此规则的值必须是 _yes_、 _on_ 或者是 _1_。这在验证是否同意"服务条款"的时候非常有用。

<a name="rule-active-url"></a>
#### active_url

验证此规则的值必须是一个合法的 URL，根据 PHP 函数 `checkdnsrr`。

<a name="rule-after"></a>
#### after:_date_

验证此规则的值必须在给定日期之后，日期将通过 PHP 函数 `strtotime` 传递。

<a name="rule-alpha"></a>
#### alpha

验证此规则的值必须全部由字母字符构成。

<a name="rule-alpha-dash"></a>
#### alpha_dash

验证此规则的值必须全部由字母、数字、中划线或下划线字符构成。

<a name="rule-alpha-num"></a>
#### alpha_num

验证此规则的值必须全部由字母和数字构成。

<a name="rule-before"></a>
#### before:_date_

验证此规则的值必须在给定日期之前，日期将通过 PHP 函数 `strtotime` 传递。

<a name="rule-between"></a>
#### between:_min_,_max_

验证此规则的值必须在给定的 _min_ 和 _max_ 之间。字符串、数字以及文件都将使用大小规则进行比较。

<a name="rule-confirmed"></a>
#### confirmed

验证此规则的值必须和 `foo_confirmation` 的值相同。比如，需要验证此规则的域是 `password`，那么在输入中必须有一个与之相同的 `password_confirmation` 域。

<a name="rule-date"></a>
#### date

验证此规则的值必须是一个合法的日期，根据 PHP 函数 `strtotime`。

<a name="rule-date-format"></a>
#### date_format:_format_

验证此规则的值必须符合给定的 _format_ 的格式，根据 PHP 函数 `date_parse_from_format`。

<a name="rule-different"></a>
#### different:_field_

验证此规则的值必须与指定的 _field_ 域的值不同。

<a name="rule-email"></a>
#### email

验证此规则的值必须是一个合法的电子邮件地址。

<a name="rule-exists"></a>
#### exists:_table_,_column_

验证此规则的值必须在指定的数据库的表中存在。

**Exists 规则的基础使用**

	'state' => 'exists:states'

**指定列名**

	'state' => 'exists:states,abbreviation'

您也可以指定更多的条件，将以 "where" 的形式添加到查询。

	'email' => 'exists:staff,email,account_id,1'

<a name="rule-image"></a>
#### image

验证此规则的值必须是一个图片 (jpeg, png, bmp 或者 gif)。

<a name="rule-in"></a>
#### in:_foo_,_bar_,...

验证此规则的值必须在给定的列表中存在。

<a name="rule-integer"></a>
#### integer

验证此规则的值必须是一个整数。

<a name="rule-ip"></a>
#### ip

验证此规则的值必须是一个合法的 IP 地址。

<a name="rule-max"></a>
#### max:_value_

验证此规则的值必须小于最大值 _value_。字符串、数字以及文件都将使用大小规则进行比较。

<a name="rule-mimes"></a>
#### mimes:_foo_,_bar_,...

验证此规则的文件的 MIME 类型必须在给定的列表中。

**MIME 规则的基础使用**

	'photo' => 'mimes:jpeg,bmp,png'

<a name="rule-min"></a>
#### min:_value_

验证此规则的值必须大于最小值 _value_。字符串、数字以及文件都将使用大小规则进行比较。

<a name="rule-not-in"></a>
#### not_in:_foo_,_bar_,...

验证此规则的值必须在给定的列表中不存在。

<a name="rule-numeric"></a>
#### numeric

验证此规则的值必须是一个数字。

<a name="rule-regex"></a>
#### regex:_pattern_

验证此规则的值必须符合给定的正则表达式。

**注意:** 当使用 `regex` 模式的时候，有必要使用数组指定规则，而不是管道分隔符，特别是正则表达式中包含一个管道字符的时候。

<a name="rule-required"></a>
#### required

验证此规则的值必须在输入数据中存在。

<a name="rule-required-if"></a>
#### required_if:_field_,_value_

当指定的域为某个值的时候，验证此规则的值必须存在。

<a name="rule-required-with"></a>
#### required_with:_foo_,_bar_,...

_仅当_指定的域存在的时候，验证此规则的值必须存在。

<a name="rule-required-without"></a>
#### required_without:_foo_,_bar_,...

_仅当_指定的域不存在的时候，验证此规则的值必须存在。

<a name="rule-same"></a>
#### same:_field_

验证此规则的值必须与给定域的值相同。

<a name="rule-size"></a>
#### size:_value_

验证此规则的值的大小必须与给定的 _value_ 相同。对于字符串，_value_ 代表字符的个数；对于数字，_value_ 代表它的整数值，对于文件，_value_ 代表文件以KB为单位的大小。

<a name="rule-unique"></a>
#### unique:_table_,_column_,_except_,_idColumn_

验证此规则的值必须在给定的数据库的表中唯一。如果 `column` 没有被指定，将使用该域的名字。

**Unique 规则的基础使用**

	'email' => 'unique:users'

**指定列名**

	'email' => 'unique:users,email_address'

**强制忽略一个给定的 ID**

	'email' => 'unique:users,email_address,10'

<a name="rule-url"></a>
#### url

验证此规则的值必须是一个合法的 URL。

<a name="custom-error-messages"></a>
## 定制错误消息

如果有需要，您可以使用定制的错误消息代替默认的消息。这里有好几种定制错误消息的方法。

**传递定制消息到验证器**

	$messages = array(
		'required' => 'The :attribute field is required.',
	);

	$validator = Validator::make($input, $rules, $messages);

*注意:* `:attribute` 占位符将被实际的进行验证的域的名字代替，您也可以在错误消息中使用其他占位符。

**其他验证占位符**

	$messages = array(
		'same'    => 'The :attribute and :other must match.',
		'size'    => 'The :attribute must be exactly :size.',
		'between' => 'The :attribute must be between :min - :max.',
		'in'      => 'The :attribute must be one of the following types: :values',
	);

有些时候，您可能希望只对一个指定的域指定定制的错误消息：

**对一个指定的域指定定制的错误消息**

	$messages = array(
		'email.required' => 'We need to know your e-mail address!',
	);

在一些情况下，您可能希望在一个语言文件中指定错误消息而不是直接传递给 `Validator`。为了实现这个目的，请在 `app/lang/xx/validation.php` 文件中添加您的定制消息到 `custom` 数组。

**在语言文件中指定错误消息**

	'custom' => array(
		'email' => array(
			'required' => 'We need to know your e-mail address!',
		),
	),

<a name="custom-validation-rules"></a>
## 定制验证规则

Laravel 提供了一系列的有用的验证规则；但是，您可能希望添加自己的验证规则。其中一种方法是使用 `Validator::extend` 函数注册定制的验证规则：

**注册一个定制的验证规则**

	Validator::extend('foo', function($attribute, $value, $parameters)
	{
		return $value == 'foo';
	});

> **注意:** 传递给 `extend` 函数的规则的名字必须符合 "snake cased" 命名规则。

定制的验证器接受三个参数：待验证属性的名字、待验证属性的值以及传递给这个规则的参数。

您也可以传递一个类的函数到 `extend` 函数，而不是使用闭包：

	Validator::extend('foo', 'FooValidator@validate');

注意您需要为您的定制规则定义错误消息。您既可以使用一个行内的定制消息数组，也可以在验证语言文件中进行添加。
 
您也可以扩展 `Validator` 类本身，而不是使用闭包回调扩展验证器。为了实现这个目的，添加一个继承自 `Illuminate\Validation\Validator` 的验证器类。您可以添加在类中添加以 `validate` 开头的验证函数：

**扩展验证器类**

	<?php

	class CustomValidator extends Illuminate\Validation\Validator {

		public function validateFoo($attribute, $value, $parameters)
		{
			return $value == 'foo';
		}

	}

下面，您需要注册定制的验证器扩展：

**您需要注册定制的验证器扩展**

	Validator::resolver(function($translator, $data, $rules, $messages)
	{
		return new CustomValidator($translator, $data, $rules, $messages);
	});

当创建一个定制的验证规则，您有时需要为错误消息定义一个定制的占位符。为了实现它，您可以像上面那样创建一个定制的验证器，并且在验证器中添加一个 `replaceXXX` 函数：

	protected function replaceFoo($message, $attribute, $rule, $parameters)
	{
		return str_replace(':foo', $parameters[0], $message);
	}
