# 缓存

- [配置](#configuration)
- [缓存用法](#cache-usage)
- [递增和递减](#increments-and-decrements)
- [缓存段](#cache-sections)
- [数据库缓存](#database-cache)

<a name="configuration"></a>
## 配置

Laravel 针对所有缓存系统提供了一个统一的 API。关于缓存的配置存放在 `app/config/cache.php` 文件中。在这个文件中，您可以指定在应用程序中默认使用哪一种缓存系统。Laravel 支持常见的缓存后端比如 [Memcached](http://memcached.org) 和 [Redis](http://redis.io) 等。

缓存配置文件也包含很多其他选项，在文件中都进行了详细的注释，所以请确认您仔细阅读了他们。默认情况下，Laravel 被配置为使用 `file` 文件缓存系统，在文件系统中保存串行化的数据。对于大型应用，推荐您使用基于内存的缓存系统比如 Memcached 以及 APC。

<a name="cache-usage"></a>
## 缓存用法

**在缓存中添加一个缓存项**

	Cache::put('key', 'value', $minutes);

**在缓存中保存一个缓存项如果这个缓存项不存在**

	Cache::add('key', 'value', $minutes);

**检查一个缓存项在缓存中是否存在**

	if (Cache::has('key'))
	{
		//
	}

**从缓存中获取一个缓存项**

	$value = Cache::get('key');

**从缓存中获取一个缓存项或它的默认值**

	$value = Cache::get('key', 'default');

	$value = Cache::get('key', function() { return 'default'; });

**在缓存中永久保存一个缓存项**

	Cache::forever('key', 'value');

有时您可能希望从缓存中获取一个缓存项，并且如果请求的缓存项不存在的时候保存一个默认值。您可以通过使用 `Cache::remember` 函数实现：

	$value = Cache::remember('users', $minutes, function()
	{
		return DB::table('users')->get();
	});

您也可以整合 `remember` 和 `forever` 函数：

	$value = Cache::rememberForever('users', function()
	{
		return DB::table('users')->get();
	});

注意所有保存在缓存的数据都已经经过串行化，所以您可以用来存储任何类型的数据。

**从缓存中删除一个缓存项**

	Cache::forget('key');

<a name="increments-and-decrements"></a>
## 递增 & 递减

除了文件缓存和数据库缓存之外的其他缓存系统都支持递增和递减的操作：

**递增一个变量**

	Cache::increment('key');

	Cache::increment('key', $amount);

**递减一个变量**

	Cache::decrement('key');

	Cache::decrement('key', $amount);

<a name="cache-sections"></a>
## 缓存段

> **注意:** 缓存段在文件缓存和数据库缓存中不被支持。

缓存段允许您在缓存中对相关的缓存项进行分组，并且刷新整个缓存段。为了获取一个缓存段，使用 `section` 函数：

**访问一个缓存段**

	Cache::section('people')->put('John', $john);

	Cache::section('people')->put('Anne', $anne);

您也可以从缓存段中获取缓存项，也可以使用其他缓存函数比如  `increment` 以及 `decrement`：

**从缓存段中获取缓存项**

	$anne = Cache::section('people')->get('Anne');

然后您可以刷新缓存段中的所有缓存项:

	Cache::section('people')->flush();

<a name="database-cache"></a>
## 数据库缓存

当使用 `database` 缓存驱动，您需要建立一张表保存缓存项。下面是一个对这张表的 `Schema` 声明的例子：

	Schema::create('cache', function($table)
	{
		$table->string('key')->unique();
		$table->text('value');
		$table->integer('expiration');
	});
