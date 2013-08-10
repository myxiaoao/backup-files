# 基础

- [配置](#configuration)
- [执行查询](#running-queries)
- [数据库事务](#database-transactions)
- [获取连接](#accessing-connections)
- [查询日志](#query-logging)

<a name="configuration"></a>
## 配置

Laravel 使得连接数据库和执行查询都非常的简单。数据库配置文件位于 `app/config/database.php`。在这个文件中您可以定义所有数据库连接，并且指定哪一个连接是被默认使用的。这个文件提供了所有支持的数据库系统的样例。

当前 Laravel 支持四种数据库系统：MySQL, Postgres, SQLite 以及 SQL Server。

<a name="running-queries"></a>
## 执行查询

一旦您已经配置好数据库连接，您可以使用 `DB` 类执行查询。

**执行一个Select查询**

	$results = DB::select('select * from users where id = ?', array(1));

`select` 方法通常返回一个查询结果的数组。

**执行一个插入表达式**

	DB::insert('insert into users (id, name) values (?, ?)', array(1, 'Dayle'));

**执行一个更新表达式**

	DB::update('update users set votes = 100 where name = ?', array('John'));

**执行一个删除表达式**

	DB::delete('delete from users');

> **注意:** `update` 和 `delete` 表达式返回该操作影响的行数。

**执行一个普通的表达式**

	DB::statement('drop table users');

您可以使用 `DB::listen` 函数监听查询事件：

**监听查询事件**

	DB::listen(function($sql, $bindings, $time)
	{
		//
	});

<a name="database-transactions"></a>
## 数据库事务

可以使用 `transaction` 函数在数据库事务中执行一系列操作：

	DB::transaction(function()
	{
		DB::table('users')->update(array('votes' => 1));

		DB::table('posts')->delete();
	});

<a name="accessing-connections"></a>
## 获取连接

当使用多个连接，您可以通过 `DB::connection` 函数获取它们：

	$users = DB::connection('foo')->select(...);

您也可以获取原始的，底层的 PDO 实例：

	$pdo = DB::connection()->getPdo();

有些时候，您可能需要重新连接给定的数据库：

	DB::reconnect('foo');

<a name="query-logging"></a>
## 查询日志

默认情况下，Laravel 在内存中保存一个包含当前请求所执行的所有查询的日志。然而，在一些情况下，比如插入很多行数据，这会导致应用程序消耗过多内存。可以使用 `disableQueryLog` 函数禁止查询日志：

	DB::connection()->disableQueryLog();