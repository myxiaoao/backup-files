# 查询生成器

- [简介](#introduction)
- [查询](#selects)
- [连接](#joins)
- [高级Wheres](#advanced-wheres)
- [统计](#aggregates)
- [原始表达式](#raw-expressions)
- [插入](#inserts)
- [更新](#updates)
- [删除](#deletes)
- [联合](#unions)
- [缓存查询](#caching-queries)

<a name="introduction"></a>
## 简介

数据库查询生成器提供了一种方便、流畅的接口用于创建和执行数据库查询。它可用于在应用程序中执行大多数数据库操作，并且适用于所有支持的数据库系统。

> **注意:** Laravel 查询生成器使用 PDO 参数绑定来保护您的应用程序抵御 SQL 注入攻击。没有必要清理通过绑定传递的字符串。

<a name="selects"></a>
## 查询

**获取一个表中的所有行**

	$users = DB::table('users')->get();

	foreach ($users as $user)
	{
		var_dump($user->name);
	}

**获取一个表中的某一行**

	$user = DB::table('users')->where('name', 'John')->first();

	var_dump($user->name);

**获取某一行中的某一列**

	$name = DB::table('users')->where('name', 'John')->pluck('name');

**获取某一列的值**

	$roles = DB::table('roles')->lists('title');

这个方法将返回一个所有 title 的列表。您也可以对返回的数组指定一个指定的键：

	$roles = DB::table('roles')->lists('title', 'name');

**指定 Select 子句**

	$users = DB::table('users')->select('name', 'email')->get();

	$users = DB::table('users')->distinct()->get();

	$users = DB::table('users')->select('name as user_name')->get();

**对一个已存在的查询添加一个 Select 子句**

	$query = DB::table('users')->select('name');

	$users = $query->addSelect('age')->get();

**使用 Where 运算符**

	$users = DB::table('users')->where('votes', '>', 100)->get();

**Or 子句**

	$users = DB::table('users')
	                    ->where('votes', '>', 100)
	                    ->orWhere('name', 'John')
	                    ->get();

**使用 Where Between**

	$users = DB::table('users')
	                    ->whereBetween('votes', array(1, 100))->get();

**使用 Where In 和一个数组**

	$users = DB::table('users')
	                    ->whereIn('id', array(1, 2, 3))->get();

	$users = DB::table('users')
	                    ->whereNotIn('id', array(1, 2, 3))->get();

**使用 Where Null 查找没有赋值的记录**

	$users = DB::table('users')
	                    ->whereNull('updated_at')->get();

**Order By, Group By 和 Having**

	$users = DB::table('users')
	                    ->orderBy('name', 'desc')
	                    ->groupBy('count')
	                    ->having('count', '>', 100)
	                    ->get();

**Offset & Limit**

	$users = DB::table('users')->skip(10)->take(5)->get();

<a name="joins"></a>
## 连接

查询生成器也可用于编写连接表达式，请看下面的例子：

**基础连接表达式**

	DB::table('users')
	            ->join('contacts', 'users.id', '=', 'contacts.user_id')
	            ->join('orders', 'users.id', '=', 'orders.user_id')
	            ->select('users.id', 'contacts.phone', 'orders.price');

您可以指定更高级的连接子句：

	DB::table('users')
	        ->join('contacts', function($join)
	        {
	        	$join->on('users.id', '=', 'contacts.user_id')->orOn(...);
	        })
	        ->get();

<a name="advanced-wheres"></a>
## 高级Wheres

有时您可能需要创建跟高级的 where 子句，比如 "where exists" 或 嵌套参数组合。Laravel 查询生成器也能够处理这些：

**参数组合**

	DB::table('users')
	            ->where('name', '=', 'John')
	            ->orWhere(function($query)
	            {
	            	$query->where('votes', '>', 100)
	            	      ->where('title', '<>', 'Admin');
	            })
	            ->get();

上面的查询将产生下面的 SQL 语句:

	select * from users where name = 'John' or (votes > 100 and title <> 'Admin')

**存在表达式**

	DB::table('users')
	            ->whereExists(function($query)
	            {
	            	$query->select(DB::raw(1))
	            	      ->from('orders')
	            	      ->whereRaw('orders.user_id = users.id');
	            })
	            ->get();

上面的查询将产生下面的 SQL 语句:

	select * from users
	where exists (
		select 1 from orders where orders.user_id = users.id
	)

<a name="aggregates"></a>
## 统计

查询生成器也提供了一系列的统计函数，比如 `count`, `max`, `min`, `avg` 以及 `sum`函数。

**使用统计函数**

	$users = DB::table('users')->count();

	$price = DB::table('orders')->max('price');

	$price = DB::table('orders')->min('price');

	$price = DB::table('orders')->avg('price');

	$total = DB::table('users')->sum('votes');

<a name="raw-expressions"></a>
## 原始表达式

有时您可能需要使用在查询中使用一个原始表达式。这些表达式有可能在查询中被注入，所以不要留下任何可以 SQL 注入的地方！创建一个原始表达式，您可以使用 `DB::raw` 函数：

**使用原始表达式**

	$users = DB::table('users')
	                     ->select(DB::raw('count(*) as user_count, status'))
	                     ->where('status', '<>', 1)
	                     ->groupBy('status')
	                     ->get();

**递增或递减一个字段的值**

	DB::table('users')->increment('votes');

	DB::table('users')->increment('votes', 5);

	DB::table('users')->decrement('votes');

	DB::table('users')->decrement('votes', 5);

您也可以为更新指定其他的字段：

	DB::table('users')->increment('votes', 1, array('name' => 'John'));

<a name="inserts"></a>
## 插入

**向表中插入一条记录**

	DB::table('users')->insert(
		array('email' => 'john@example.com', 'votes' => 0)
	);

如果表中有一个自动递增的 ID 字段，使用 `insertGetId` 插入一条记录并获取 ID:

**向带有自动递增的 ID 字段的表中插入一条记录**

	$id = DB::table('users')->insertGetId(
		array('email' => 'john@example.com', 'votes' => 0)
	);

> **注意:** 当使用 PostgreSQL，insertGetId 函数需要自动递增的字段命名为 "id"。

**向表中一次性插入多条记录**

	DB::table('users')->insert(array(
		array('email' => 'taylor@example.com', 'votes' => 0),
		array('email' => 'dayle@example.com', 'votes' => 0),
	));

<a name="updates"></a>
## 更新

**更新表中的一条记录**

	DB::table('users')
	            ->where('id', 1)
	            ->update(array('votes' => 1));

<a name="deletes"></a>
## 删除

**删除表中的一条记录**

	DB::table('users')->where('votes', '<', 100)->delete();

**删除表中的所有记录**

	DB::table('users')->delete();

**清空表**

	DB::table('users')->truncate();

<a name="unions"></a>
## 联合

查询生成器也提供了一种快速的方法联合两个查询：

**执行一个查询联合**

	$first = DB::table('users')->whereNull('first_name');

	$users = DB::table('users')->whereNull('last_name')->union($first)->get();

也可以使用 `unionAll`，与 `union` 有同样的签名。

<a name="caching-queries"></a>
## 缓存查询

您可以很容易地使用 `remember` 函数缓存一个查询的结果：

**缓存一个查询的结果**

	$users = DB::table('users')->remember(10)->get();

在这个例子中，查询的结果将被缓存十分钟。因为查询结果被缓存，查询将不会在数据库中运行，结果将从应用程序指定的默认的缓存系统中加载。
