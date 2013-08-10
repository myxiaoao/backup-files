# 分页

- [配置](#configuration)
- [用法](#usage)
- [分页链接附加](#appending-to-pagination-links)

<a name="configuration"></a>
## 配置

在其他框架中，分页可能是非常痛苦的。Laravel 使它变得很轻松。在 `app/config/view.php` 文件中有一个配置选项。`pagination` 选项指明哪个视图用来生成分页链接。默认情况，Laravel 包含两个视图。

`pagination::slider` 视图将显示一个智能的基于当前页码的范围的链接，另一个 `pagination::simple` 将简单显示 "previous" 和 "next" 按钮。**这两个都可以立即适用于 Twitter Bootstrap。**

<a name="usage"></a>
## 用法

有好几种方法产生分页，最简单的的是在一个查询构建器或 Eloquent 模型中使用 `paginate` 函数：

**数据库查询结果分页**

	$users = DB::table('users')->paginate(15);

您也可以对 [Eloquent](/docs/eloquent) 模型进行分页；

**Eloquent 模型分页**

	$users = User::where('votes', '>', 100)->paginate(15);

传递给 `paginate` 函数的参数是您希望每页显示项目的个数。一旦您获取了结果，您可以在视图中显示它们，并使用 `links` 函数产生分页链接：

	<div class="container">
		<?php foreach ($users as $user): ?>
			<?php echo $user->name; ?>
		<?php endforeach; ?>
	</div>

	<?php echo $users->links(); ?>

以上便是产生一个分页的全部过程！注意我们没有必要告诉框架当前的页面，Laravel 将会帮我们自动检测。

您也可以通过以下函数获取其他分页信息：

- `getCurrentPage`
- `getLastPage`
- `getPerPage`
- `getTotal`
- `getFrom`
- `getTo`

有些时候，您可能希望通过传递一个选项数组手动创建一个分页实例。您可以使用 `Paginator::make` 函数实现这个功能：

**手动创建一个分页**

	$paginator = Paginator::make($items, $totalItems, $perPage);

<a name="appending-to-pagination-links"></a>
## 分页链接附加

您可以在分页类上使用 `appends` 函数在分页链接中附加查询字符串：

	<?php echo $users->appends(array('sort' => 'votes'))->links(); ?>

这将产生如下所示的 URL：

	http://example.com/something?page=2&sort=votes