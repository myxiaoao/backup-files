# Redis

- [简介](#introduction)
- [配置](#configuration)
- [使用](#usage)
- [流水线](#pipelining)

<a name="introduction"></a>
## 简介

[Redis](http://redis.io) 是一个开源的，高级的键值存储系统。它经常被涉及作为一个数据结构的服务器因为键能够包含 [字符串](http://redis.io/topics/data-types#strings)、[哈希](http://redis.io/topics/data-types#hashes)、[列表](http://redis.io/topics/data-types#lists)、[集合](http://redis.io/topics/data-types#sets) 以及 [有序集合](http://redis.io/topics/data-types#sorted-sets)。

> **注意:** 如果您通过 PECL 安装了 Redis 的 PHP 扩展，您需要在 `app/config/app.php` 文件中重命名 Redis 的别名。

<a name="configuration"></a>
## 配置

应用程序中 Redis 的配置被存放在 **app/config/database.php** 文件中。在这个文件里，您将看到一个 **redis** 数组包含一个应用程序中所用到的 Redis 服务器：

	'redis' => array(

		'cluster' => true,

		'default' => array('host' => '127.0.0.1', 'port' => 6379),

	),

默认的服务器配置足够开发的时候使用，但是您可以轻松的根据您的环境修改这个数据。请给每一个 Redis 服务器一个名字，并指明服务器所使用的主机地址和端口号。

`cluster` 选项将告诉 Laravel Redis 客户端在 Redis 节点中执行客户端分片，这允许您汇总节点并节省大量内存。但是，请注意客户端分片不处理故障转移，因此主要用于从一个主数据存储的缓存系统。

<a name="usage"></a>
## 使用

您可以通过调用 `Redis::connection` 函数获取一个 Redis 实例：

	$redis = Redis::connection();

这将返回一个默认的 Redis 服务器的实例。如果您没有使用服务器集群，您可以传递服务器的名字到 `connection` 函数获取一个在 Redis 配置文件中指定的服务器：

	$redis = Redis::connection('other');

一旦您得到一个 Redis 客户端的实例，我们可以对该实例调用任何 [Redis 命令](http://redis.io/commands)。Laravel 使用魔术函数传递命令到 Redis 服务器：

	$redis->set('name', 'Taylor');

	$name = $redis->get('name');

	$values = $redis->lrange('names', 5, 10);

注意命令的参数被简单的传递到魔术函数。当然，您不必使用魔术函数，您可以使用 `command` 函数传递命令到服务器：

	$values = $redis->command('lrange', array(5, 10));

当您通过默认的连接执行命令，只需要对 `Redis` 类使用静态的方法：

	Redis::set('name', 'Taylor');

	$name = Redis::get('name');

	$values = Redis::lrange('names', 5, 10);

> **注意:** Laravel 也包含 Redis [缓存](/docs/cache) 以及 [session](/docs/session) 的驱动.

<a name="pipelining"></a>
## 流水线

流水线可以在您需要在一次操作中发送许多命令到服务器的时候用到，请使用 `pipeline` 命令：

**传送许多命令至服务器**

	Redis::pipeline(function($pipe)
	{
		for ($i = 0; $i < 1000; $i++)
		{
			$pipe->set("key:$i", $i);
		}
	});