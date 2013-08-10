# 本地化

- [简介](#introduction)
- [语言文件](#language-files)
- [基础使用](#basic-usage)
- [复数](#pluralization)

<a name="introduction"></a>
## 简介

Laravel 的 `Lang` 类提供了一种便利的方式在不同的语言中获取字符串，允许您在应用程序中支持多国语言。

<a name="language-files"></a>
## 语言文件

语言字符串保存在 `app/lang` 目录下的文件中。在这个目录中，应该为应用程序需要支持的每种语言创建一个子目录。

	/app
		/lang
			/en
				messages.php
			/es
				messages.php

语言文件返回一个有键值的字符串数组，比如：

**语言文件例子**

	<?php

	return array(
		'welcome' => 'Welcome to our application'
	);

您应用程序默认的语言保存在 `app/config/app.php` 配置文件中。您可以在任何时候通过 `App::setLocale` 改变当前使用的语言：

**程序运行时改变默认语言**

	App::setLocale('es');

<a name="basic-usage"></a>
## 基础使用

**从语言文件中获取行**

	echo Lang::get('messages.welcome');

传递给 `get` 函数的字符串的第一个段是语言文件的名字，第二个是需要获取的行的名字。

> **注意**: 如果一个语言行不存在，`get` 函数将返回键值。

**在行内进行替换**

您可以定义在语言行中定义占位符：

	'welcome' => 'Welcome, :name',

然后，为 `Lang::get` 函数的第二个参数传递替换值：

	echo Lang::get('messages.welcome', array('name' => 'Dayle'));

**检查语言文件中是否包含某一行**

	if (Lang::has('messages.welcome'))
	{
		//
	}

<a name="pluralization"></a>
## 复数

复数是一个复杂的议题，因为不同的语言对复数有很多复杂的规则。您可以简单地通过语言文件进行管理。通过使用一个管道字符，您可以把一个字符串的单数和复数形式隔开：

	'apples' => 'There is one apple|There are many apples',

接下来您可以使用 `Lang::choice` 方法获取这一行：

	echo Lang::choice('messages.apples', 10);

因为 Laravel 翻译器基于 Symfony 翻译组件，您可以简单地创建更多明确的复数的规则：

	'apples' => '{0} There are none|[1,19] There are some|[20,Inf] There are many',
