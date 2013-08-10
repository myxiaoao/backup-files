# Session

- [配置](#configuration)
- [Session 用法](#session-usage)
- [闪存数据](#flash-data)
- [数据库 Sessions](#database-sessions)

<a name="configuration"></a>
## 配置

因为 HTTP 驱动的应用是无状态的，Session 提供了一种在请求中保存用户信息的方式。Laravel 自带很多可用的 Session 后端，并提供了整洁统一的 API。常见的后端比如 [Memcached](http://memcached.org)、[Redis](http://redis.io) 以及数据库可以立即进行使用。

Session 的配置存放在 `app/config/session.php` 文件。请确认浏览了该文件中已经详尽注释的选项。在默认情况下，Laravel 被配置为使用 `native` 原生的 Session 驱动，这在大多应用中都能够很好的工作。

<a name="session-usage"></a>
## Session 用法

**在 Session 中存储一个变量**

	Session::put('key', 'value');

**从 Session 中获取一个变量**

	$value = Session::get('key');

**从 Session 中获取一个变量或返回一个默认值**

	$value = Session::get('key', 'default');

	$value = Session::get('key', function() { return 'default'; });

**检查一个变量是否在 Session 中存在**

	if (Session::has('users'))
	{
		//
	}

**从 Session 中删除一个变量**

	Session::forget('key');

**从 Session 中删除所有变量**

	Session::flush();

**重新生成 Session ID**

	Session::regenerate();

<a name="flash-data"></a>
## 闪存数据

有些时候您可能希望只为下一次请求在 Session 中存储一些数据。您可以通过使用 `Session::flash` 函数实现：

	Session::flash('key', 'value');

**为下一次请求再次存储当前闪存的数据**

	Session::reflash();

**为下一次请求再次存储当前闪存数据的部分数据**

	Session::keep(array('username', 'email'));

<a name="database-sessions"></a>
## 数据库 Sessions

当使用 `database` Session 驱动，您需要建立一张表保存 Session 数据。下面是这张表的模式声明的一个例子：

	Schema::create('sessions', function($table)
	{
		$table->string('id')->unique();
		$table->text('payload');
		$table->integer('last_activity');
	});

当然，您也可以使用Artisan 命令 `session:table` 为您生成这个迁移:

	php artisan session:table

	composer dump-autoload

	php artisan migrate