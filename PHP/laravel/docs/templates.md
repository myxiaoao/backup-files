# 模板

- [控制器布局](#controller-layouts)
- [Blade 模板](#blade-templating)
- [其他 Blade 控制结构](#other-blade-control-structures)

<a name="controller-layouts"></a>
## 控制器布局

一种在 Laravel 使用模板的方法是通过控制器布局。通过在控制器中指定 `layout` 属性，指定的视图将为您创建并被作为函数的响应。

**在控制器中定义一个布局**

	class UserController extends BaseController {

		/**
		 * The layout that should be used for responses.
		 */
		protected $layout = 'layouts.master';

		/**
		 * Show the user profile.
		 */
		public function showProfile()
		{
			$this->layout->content = View::make('user.profile');
		}

	}

<a name="blade-templating"></a>
## Blade 模板

Blade 是 Laravel 提供的一个简单但很强大的模板引擎。不像控制器布局，Blade 由_模板继承_和_切片_驱动。所有 Blade 模板应该使用 `.blade.php` 文件后缀。
**定义一个 Blade 布局**

	<!-- Stored in app/views/layouts/master.blade.php -->

	<html>
		<body>
			@section('sidebar')
				This is the master sidebar.
			@show

			<div class="container">
				@yield('content')
			</div>
		</body>
	</html>

**使用一个 Blade 布局**

	@extends('layouts.master')

	@section('sidebar')
		@parent

		<p>This is appended to the master sidebar.</p>
	@stop

	@section('content')
		<p>This is my body content.</p>
	@stop

注意 `extend` 一个 Blade 布局的视图将覆盖布局中的切片。布局中的内容可以在子视图的切片中使用 `@parent` 指令被包含进来，允许您添加布局中的内容比如侧边栏或者底部。

<a name="other-blade-control-structures"></a>
## 其他 Blade 控制结构

**显示数据**

	Hello, {{ $name }}.

	The current UNIX timestamp is {{ time() }}.


为了对输出内容进行转义，您可以使用三花括号语法：

	Hello, {{{ $name }}}.

**If 结构**

	@if (count($records) === 1)
		I have one record!
	@elseif (count($records) > 1)
		I have multiple records!
	@else
		I don't have any records!
	@endif

	@unless (Auth::check())
		You are not signed in.
	@endunless

**循环**

	@for ($i = 0; $i < 10; $i++)
		The current value is {{ $i }}
	@endfor

	@foreach ($users as $user)
		<p>This is user {{ $user->id }}</p>
	@endforeach

	@while (true)
		<p>I'm looping forever.</p>
	@endwhile

**包含子视图**

	@include('view.name')

**显示语言行**

	@lang('language.line')

	@choice('language.line', 1);

**注释**

	{{-- This comment will not be in the rendered HTML --}}
