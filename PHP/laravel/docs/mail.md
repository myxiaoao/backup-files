# 邮件

- [配置](#configuration)
- [基本用法](#basic-usage)
- [嵌入内联附件](#embedding-inline-attachments)
- [邮件队列](#queueing-mail)
- [邮件与本地开发](#mail-and-local-development)

<a name="configuration"></a>
## 配置

Laravel 提供了一个基于流行的 [SwiftMailer](http://swiftmailer.org) 类库的简洁的接口。邮件的配置文件位于 `app/config/mail.php`，包含了一些选项允许您更改 SMTP 服务器地址、端口和凭证，且通过这个类为所有信息发送的表单提供了全局的邮件地址。您可以按照您的意愿使用任何 SMTP 服务器。如果您希望使用 PHP 的 `mail` 函数发送邮件，您可以在配置文件中更改 `driver` 为 `mail`，另外也可使用 `sendmail` 程序。

<a name="basic-usage"></a>
## 基本用法

`Mail::send` 函数用来发送一封电子邮件：

	Mail::send('emails.welcome', $data, function($message)
	{
		$message->to('foo@example.com', 'John Smith')->subject('Welcome!');
	});

传递给 `send` 函数的第一个参数是作为邮件内容的视图的名字。第二个参数是传递给视图的数据，第三个参数是一个闭包允许您指定电子邮件消息的更多选项。

> **注意:** 一个 `$message` 变量总会传递给邮件视图，允许内嵌附件。所以最好避免在视图中传递一个 `message` 变量。

除了使用 HTML 视图，您也可以指定使用一个纯文本的视图：

	Mail::send(array('html.view', 'text.view'), $data, $callback);

或者，您可以指定使用键名 `html` 或 `text` 指定文档的类型：

	Mail::send(array('text' => 'view'), $data, $callback);

您也可以在邮件消息中指定其他的选项，比如任何副本或附件：

	Mail::send('emails.welcome', $data, function($message)
	{
		$message->from('us@example.com', 'Laravel');

		$message->to('foo@example.com')->cc('bar@example.com');

		$message->attach($pathToFile);
	});

将文件附加到一个消息时，您也可以指定文件的 MIME 类型以及显示名称： 

	$message->attach($pathToFile, array('as' => $display, 'mime' => $mime));

> **注意:** 传递给 `Mail::send` 闭包函数的消息实例继承自 SwiftMailer 的消息类，允许您调用这个类的任何方法构建您的电子邮件消息。


<a name="embedding-inline-attachments"></a>
## 嵌入内联附件

在您的电子邮件中嵌入内嵌的图片通常是非常麻烦的，但是，Laravel 提供了一个便利的方式将图片附加到电子邮件中，并且获取相应的 CID。

**在邮件视图中嵌入一个图片**

	<body>
		Here is an image:

		<img src="<?php echo $message->embed($pathToFile); ?>">
	</body>

**在邮件视图中嵌入原始数据**

	<body>
		Here is an image from raw data:

		<img src="<?php echo $message->embedData($data, $name); ?>">
	</body>

注意 `$message` 变量总是通过 `Mail` 类传递到邮件视图。

<a name="queueing-mail"></a>
## 邮件队列

因为发送邮件会大大延长您应用程序的响应时间，许多开发者选择在后台使用队列发送电子邮件。Laravel 利用内置的 [统一队列接口](/docs/queues) 使得这个功能很容易实现。把一个邮件消息插入队列，只需要对 `Mail` 类使用 `queue` 函数：

**邮件消息排队**

	Mail::queue('emails.welcome', $data, function($message)
	{
		$message->to('foo@example.com', 'John Smith')->subject('Welcome!');
	});

您可以使用 `later` 函数指定您想延迟发送邮件消息的时间的秒数：

	Mail::later(5, 'emails.welcome', $data, function($message)
	{
		$message->to('foo@example.com', 'John Smith')->subject('Welcome!');
	});

如果您想指定一个指定的队列或管道来压入您的消息，您可以使用 `queueOn` 和 `laterOn` 实现：

	Mail::queueOn('queue-name', 'emails.welcome', $data, function($message)
	{
		$message->to('foo@example.com', 'John Smith')->subject('Welcome!');
	});

<a name="mail-and-local-development"></a>
## 邮件与本地开发

当开发一个应用程序，需要发送电子邮件时，经常需要在本地或开发环境中禁用发送邮件。要做到这一点，您可以调用 `Mail::pretend` 函数，或者设置 `app/config/mail.php` 配置文件中的 `pretend` 选项为 `true`。当邮件类在 `pretend` 模式时，消息将被写入到应用程序的日志文件中，而不是发送给收件人。

**开启 Pretend 邮件模式**

	Mail::pretend();