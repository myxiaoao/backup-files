# 事件

- [基础使用](#basic-usage)
- [通配符监听器](#wildcard-listeners)
- [使用类作为监听器](#using-classes-as-listeners)
- [事件队列](#queued-events)
- [事件订阅者](#event-subscribers)

<a name="basic-usage"></a>
## 基础使用

Laravel 的 `Event` 类提供了一个观察者的实现，允许您在应用程序中订阅和监听事件。


**监听一个事件**

	Event::listen('user.login', function($user)
	{
		$user->last_login = new DateTime;

		$user->save();
	});

**触发一个时间**

	$event = Event::fire('user.login', array($user));

您也可以在订阅事件的时候指定优先级。高优先级的监听器会被首先运行，同等优先级的监听器将会按照订阅的顺序执行。

**带优先级的事件订阅**

	Event::listen('user.login', 'LoginHandler', 10);

	Event::listen('user.login', 'OtherHandler', 5);

有些时候您可能希望停止事件触发其他监听器，这可以通过在监听器中返回 `false` 来实现：

**停止事件冒泡**

	Event::listen('user.login', function($event)
	{
		// Handle the event...

		return false;
	});

<a name="wildcard-listeners"></a>
## 通配符监听器

当注册事件监听器的时候，您可以使用星号指定通配符监听器：

**注册通配符事件监听器**

	Event::listen('foo.*', function($param, $event)
	{
		// Handle the event...
	});

这个监听器将处理以 `foo.` 开头的所有事件。注意事件的全名作为最后一个参数传递给监听器。

<a name="using-classes-as-listeners"></a>
## 使用类作为监听器

在某些情况下，您可能希望使用一个类来处理一个时间而不是一个闭包。监听器类将利用 [Laravel IoC 容器](/docs/ioc) 在监听器中提供强大的依赖注入。

**注册一个监听器类**

	Event::listen('user.login', 'LoginHandler');

默认情况下，`LoginHandler` 类的 `handle` 函数将被调用：

**定义一个监听器类**

	class LoginHandler {

		public function handle($data)
		{
			//
		}

	}

如果您不想使用默认的 `handle` 函数，您可以指定需要被订阅的函数。

**指定需要订阅的函数**

	Event::listen('user.login', 'LoginHandler@onLogin');

<a name="queued-events"></a>
## 事件队列

使用 `queue` 和 `flush` 方法，您可以把事件放入队列等待触发，而不是立即触发：

**注册一个排队事件**

	Event::queue('foo', array($user));

**注册一个事件清理器**

	Event::flusher('foo', function($user)
	{
		//
	});

最后，您可以使用 `flush` 函数清理所有时间队列。

	Event::flush('foo');

<a name="event-subscribers"></a>
## 事件订阅者

事件订阅者是可以在自身类中订阅多个时间的类。订阅者类必须定义一个 `subscribe` 函数，用以传递一个事件分发实例：

**定义一个事件订阅者**

	class UserEventHandler {

		/**
		 * Handle user login events.
		 */
		public function onUserLogin($event)
		{
			//
		}

		/**
		 * Handle user logout events.
		 */
		public function onUserLogout($event)
		{
			//
		}

		/**
		 * Register the listeners for the subscriber.
		 *
		 * @param  Illuminate\Events\Dispatcher  $events
		 * @return array
		 */
		public function subscribe($events)
		{
			$events->listen('user.login', 'UserEventHandler@onUserLogin');

			$events->listen('user.logout', 'UserEventHandler@onUserLogout');
		}

	}

一旦订阅者被定义，可以通过 `Event` 类注册。

**注册一个事件订阅者**

	$subscriber = new UserEventHandler;

	Event::subscribe($subscriber);
