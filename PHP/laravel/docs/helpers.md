# 助手函数

- [数组](#arrays)
- [路径](#paths)
- [字符串](#strings)
- [URLs](#urls)
- [杂项](#miscellaneous)

<a name="arrays"></a>
## 数组

### array_add

`array_add` 函数添加一个键/值对到数组如果给定的键在数组中不存在。

	$array = array('foo' => 'bar');

	$array = array_add($array, 'key', 'value');

### array_divide

The `array_divide` 函数返回两个数组，一个包含这个数组中所有的键，另一个包含这个数组中所有的值。

	$array = array('foo' => 'bar');

	list($keys, $values) = array_divide($array);

### array_dot

`array_dot` 函数使用点符号表示层次把一个多维数组转为一个一维数组。

	$array = array('foo' => array('bar' => 'baz'));

	$array = array_dot($array);

	// array('foo.bar' => 'baz');

### array_except

`array_except` 函数从数组中删除给定的键/值对。

	$array = array_except($array, array('keys', 'to', 'remove'));

### array_fetch

`array_fetch` 函数返回一个扁平的数组包含所选的嵌套元素。

	$array = array(array('name' => 'Taylor'), array('name' => 'Dayle'));

	var_dump(array_fetch($array, 'name'));

	// array('Taylor', 'Dayle');

### array_first

`array_first` 函数根据给定的布尔测试返回数组的第一个元素。

	$array = array(100, 200, 300);

	$value = array_first($array, function($key, $value)
	{
		return $value >= 150;
	});

一个默认值可以通过第三个参数被传递：

	$value = array_first($array, $callback, $default);

### array_flatten

`array_flatten` 函数将扁平一个多维数组到一个一维数组。

	$array = array('name' => 'Joe', 'languages' => array('PHP', 'Ruby'));

	$array = array_flatten($array);

	// array('Joe', 'PHP', 'Ruby');

### array_forget

`array_forget` 函数将从使用点符号从嵌套的数组删除给定的键/值对。

	$array = array('names' => array('joe' => array('programmer')));

	$array = array_forget($array, 'names.joe');

### array_get

`array_get` 函数将使用点符号从嵌套的数组获取值。

	$array = array('names' => array('joe' => array('programmer')));

	$value = array_get($array, 'names.joe');

### array_only

`array_only` 函数将从数组中返回指定的键/值对。

	$array = array('name' => 'Joe', 'age' => 27, 'votes' => 1);

	$array = array_only($array, array('name', 'votes'));

### array_pluck

`array_pluck` 函数将从数组的键/值对中导出一个列表。

	$array = array(array('name' => 'Taylor'), array('name' => 'Dayle'));

	$array = array_pluck($array, 'name');

	// array('Taylor', 'Dayle');

### array_pull

`array_pull` 函数将从数组中返回给定的键/值对，并删除它。

	$array = array('name' => 'Taylor', 'age' => 27);

	$name = array_pull($array, 'name');

### array_set

`array_set` 函数使用点符号在深层嵌套的数组中设置一个值。

	$array = array('names' => array('programmer' => 'Joe'));

	array_set($array, 'names.editor', 'Taylor');

### array_sort

`array_sort` 函数通过给定的闭包函数对数组排序。

	$array = array(
		array('name' => 'Jill'),
		array('name' => 'Barry'),
	);

	$array = array_values(array_sort($array, function($value)
	{
		return $value['name'];
	}));

### head

返回数组的第一个元素。在 PHP 5.3.x 的链式方法中有用。

	$first = head($this->returnsArray('foo'));

### last

返回数组的最后一个元素。在链式方法中有用。

	$last = last($this->returnsArray('foo'));

<a name="paths"></a>
## 路径

### app_path

返回 `application` 目录的完整路径。

### base_path

返回应用安装主目录的完整路径。

### public_path

返回 `public` 目录的完整路径。

### storage_path

返回 `application/storage` 目录的完整路径。

<a name="strings"></a>
## 字符串

### camel_case

使用 `camelCase` 方法转换一个给定字符串。

	$camel = camel_case('foo_bar');

	// fooBar

### class_basename

获取给定类的类名，除去任何名字空间。

	$class = class_basename('Foo\Bar\Baz');

	// Baz

### e

对给定字符串运行 `htmlentites`，支持UTF-8。

	$entities = e('<html>foo</html>');

### ends_with

检查某字符串是否以给定的字符串结尾。

	$value = ends_with('This is my name', 'name');

### snake_case

使用 `snake_case` 方法转换一个给定的字符串。

	$snake = snake_case('fooBar');

	// foo_bar

### starts_with

检查某字符串是否以给定的字符串开头。

	$value = starts_with('This is my name', 'This');

### str_contains

检查某字符串是否包含给定的字符串。

	$value = str_contains('This is my name', 'my');

### str_finish

添加一个字符串实例到某字符串，删除任何这个字符串已存在的实例。

	$string = str_finish('this/string', '/');

	// this/string/

### str_is

确定给定的字符串是否匹配给定的模式表达式。可使用星号作为通配符。

	$value = str_is('foo*', 'foobar');

### str_plural

将字符串转换成它的复数形式（只支持英文）。

	$plural = str_plural('car');

### str_random

生成一个给定长度的随机字符串。

	$string = str_random(40);

### str_singular

将字符串转换成它的单数形式（只支持英文）。

	$singular = str_singular('cars');

### studly_case

使用 `StudlyCase` 方法转换一个给定的字符串。

	$value = studly_case('foo_bar');

	// FooBar

### trans

翻译一个语言行，作为 `Lang::get` 的快捷方式。

	$value = trans('validation.required'):

### trans_choice

使用反射翻译一个语言行，作为 `Lang::choice` 的快捷方式。

	$value = trans_choice('foo.bar', $count);

<a name="urls"></a>
## URLs

### action

对给定的控制器动作生成 URL。

	$url = action('HomeController@getIndex', $params);

### asset

对一个资源生成 URL。

	$url = asset('img/photo.jpg');

### link_to

对一个 HTML 链接生成 URL。

	echo link_to('foo/bar', $title, $attributes = array(), $secure = null);

### link_to_asset

对给定的资源生成一个 HTML 链接。

	echo link_to_asset('foo/bar.zip', $title, $attributes = array(), $secure = null);

### link_to_route

对给定的路由生成一个 HTML 链接。

	echo link_to_route('route.name', $title, $parameters = array(), $attributes = array());

### link_to_action

对给定的控制器动作生成一个 HTML 链接。

	echo link_to_action('HomeController@getIndex', $title, $parameters = array(), $attributes = array());

### secure_asset

使用 HTTPS 对给定的资源生成一个 HTML 链接。

	echo secure_asset('foo/bar.zip', $title, $attributes = array());

### secure_url

使用 HTTPS 对给定的路径生成完整的 URL。

	echo secure_url('foo/bar', $parameters = array());

### url

根据给定的路径生成完整的 URL。

	echo url('foo/bar', $parameters = array(), $secure = null);

<a name="miscellaneous"></a>
## 杂项

### csrf_token

获取当前 CSRF 令牌的值。

	$token = csrf_token();

### dd

打印指定变量的值并且停止运行脚本。

	dd($value);

### value

如果指定的值是一个闭包，返回闭包函数所返回的值，否则直接返回这个值。

	$value = value(function() { return 'bar'; });

### with

返回指定的对象。对 PHP 5.3.x 中的链式函数调用很有用。

	$value = with(new Foo)->doWork();
