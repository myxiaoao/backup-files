# 队列

- [配置](#configuration)
- [基本用法](#basic-usage)
- [队列闭包](#queueing-closures)
- [运行队列监听器](#running-the-queue-listener)
- [压入队列](#push-queues)

<a name="configuration"></a>
## 配置

Laravel 队列组件为不同的队列服务提供了统一的接口。队列允许您推迟处理一个耗时的任务，比如发送电子邮件，直到一个稍后的时间，这样可以加快应用程序请求的速度。

队列配置文件保存在 `app/config/queue.php`。在这个文件中您会发现框架中包含的每种队列驱动的连接配置，包括 [Beanstalkd](http://kr.github.com/beanstalkd), [IronMQ](http://iron.io), [Amazon SQS](http://aws.amazon.com/sqs) 以及 synchronous (为本地应用) 驱动。 

以下是列出的队列驱动所需的依赖：

- Beanstalkd: `pda/pheanstalk`
- Amazon SQS: `aws/aws-sdk-php`
- IronMQ: `iron-io/iron_mq`

<a name="basic-usage"></a>
## 基本用法

使用 `Queue::push` 函数把一个新任务压入队列：

**把一个新任务压入队列**

	Queue::push('SendEmail', array('message' => $message));

传递给 `push` 函数的第一个参数是用来处理这个任务的类的名字。第二个参数是需要传递给处理器的一个数组。一个任务处理器应该这样定义：

**定义一个任务处理器**

	class SendEmail {

		public function fire($job, $data)
		{
			//
		}

	}

注意唯一必需的函数是 `fire`，它接受一个任务实例以及压入到队列时 `data` 的数组。

如果您想这个任务使用另一个名字而不是 `fire`，您可以指定在压入这个任务时指定：

**指定一个任务处理器函数**

	Queue::push('SendEmail@send', array('message' => $message));

一旦您处理了一个任务，它必须从队列中删除，这可以通过在任务实例上调用 `delete` 函数：

**删除一个已处理的任务**

	public function fire($job, $data)
	{
		// Process the job...

		$job->delete();
	}

如果您想从队列中遣返一个任务，可以使用 `release` 函数：

**从队列中遣返一个任务**

	public function fire($job, $data)
	{
		// Process the job...

		$job->release();
	}

您也可以指定任务被遣返前等待的秒数：

	$job->release(5);

如果在任务处理时发生了一个异常，将被自动遣返回队列。您可以使用 `attempts` 函数检查执行这个任务已经尝试的次数：

**检查已经尝试的次数**

	if ($job->attempts() > 3)
	{
		//
	}

您也可以获取任务的标识符：

**获取任务标识**

	$job->getJobId();

<a name="queueing-closures"></a>
## 队列闭包

您也可以把一个闭包压入队列。这是一个非常方便的把快捷、简单的任务压入队列的方法：

**把一个闭包压入队列**

	Queue::push(function($job) use ($id)
	{
		Account::delete($id);

		$job->delete();
	});

> **注意:** 当压入闭包到队列，`__DIR__` 以及 `__FILE__` 常量不应该被使用。

当使用 Iron.io [push queues](#push-queues)，您应该采取其他预防措施排队闭包。接收队列消息的后端应该检查一个令牌以验证这个请求来自 Iron.io。比如，您的压入队列后端应该像这样：`https://yourapp.com/queue/receive?token=SecretToken`。然后您可以在调度队列请求之前在应用程序中检查令牌的值。

<a name="running-the-queue-listener"></a>
## 运行队列监听器

Laravel 包含一个 Artisan 任务用于运行压入队列的新的任务。您可以通过使用 `queue:listen` 命令运行这个任务：

**开始队列监听**

	php artisan queue:listen

您可以指明监听器使用哪个队列连接：

	php artisan queue:listen connection

注意一旦这个任务开始，它将持续运行直到手动停止。您可以使用进程监控器比如 [Supervisor](http://supervisord.org/) 来确认队列监听器没有停止运行。

您可以设置每个任务允许运行的时间（以秒为单位）：


**指定任务超时参数**

	php artisan queue:listen --timeout=60

可以使用 `queue:work` 函数只处理队列中的第一个任务：

**处理队列中的第一个任务**

	php artisan queue:work

<a name="push-queues"></a>
## 压入队列

压入队列允许您利用强大的 Laravel 4 队列功能而不需要运行任何守护进程或后台监听器。目前压入队列只支持 [Iron.io](http://iron.io) 驱动。在开始之前，创建一个 Iron.io 帐号，并且添加 Iron 凭证到 `app/config/queue.php` 配置文件。

接下来，您可以使用 `queue:subscribe` Artisan 命令注册一个接收新的队列任务的 URL 后端：

**注册一个压入队列订阅者**

	php artisan queue:subscribe queue_name http://foo.com/queue/receive

现在，当您登录到 Iron 控制面板，您将看到新的压入队列，以及订阅的 URL。您可以按照您的意愿订阅尽可能多的 URLs到指定的队列。接下来，为 `queue/receive` 后端创建一个路由并且从 `Queue::marshal` 函数返回相应：

	Route::post('queue/receive', function()
	{
		return Queue::marshal();
	});

`marshal` 函数将用于触发正确的任务处理器类。为了在压入队列中触发任务，和常规的队列一样使用相同的 `Queue::push` 函数。
