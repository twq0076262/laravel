# 事件

* 基本用法
* 事件处理队列
* 事件订阅者

## 基本用法

Laravel 的 event 功能提供一个简单的观察者实现，允许你在应用程序里订阅与监听事件。事件类通常被保存在 `app/Events` 目录下，而它们的处理程序则被保存在 `app/Handlers/Events` 目录下。

你可以使用 Artisan 命令行工具产生一个新的事件类：

```
    php artisan make:event PodcastWasPurchased
```

#### 订阅事件

Laravel 里的 `EventServiceProvider` 提供了一个方便的地方注册所有的事件处理程序。`listen` 属性包含一个所有的事件 (键) 和相对应的处理程序 (值) 的 数组。当然，你可以依应用程序的需求添加任何数量的事件到这个数组。举个例子，让我们来加上 `PodcastWasPurchased` 事件：

```
    /**
     * 应用程序的事件处理程序对照。
     *
     * @var array
     */
    protected $listen = [
        'AppEventsPodcastWasPurchased' => [
            'AppHandlersEventsEmailPurchaseConfirmation',
        ],
    ];
```

使用 Artisan 命令行命令 `handler:event`，来产生一个事件的处理程序：

```
    php artisan handler:event EmailPurchaseConfirmation --event=PodcastWasPurchased
```

当然，在每次你需要一个处理程序或是事件时，手动地执行 `make:event` 和 `handler:event` 命令很麻烦。作为替代，简单地添加处理程序跟事件到你的 `EventServiceProvider` 并使用 `event:generate` 命令。这个命令将会产生任何在你的 `EventServiceProvider` 列出的事件跟处理程序：

```
    php artisan event:generate
```
#### 触发事件

现在我们准备好使用 `Event` facade 触发我们的事件：

```
    $response = Event::fire(new PodcastWasPurchased($podcast));
```

`fire` 方法返回一个响应的数组，让你可以用来控制你的应用程序接下来要有什么反应。

你也可以使用 `event` 辅助方法来触发事件：

```
    event(new PodcastWasPurchased($podcast));
```

#### 监听器闭包

你甚至可以不需对事件建立对应的处理类。举个例子，在你的 `EventServiceProvider` 的 `boot` 方法里，你可以做下面这件事：

```
    Event::listen('AppEventsPodcastWasPurchased', function($event)
    {
        // 处理事件...
    });
```

#### 停止继续传递事件

有时候你会希望停止继续传递事件到其他监听器。你可以通过从处理程序返回 `false` 来做到这件事：

```
    Event::listen('AppEventsPodcastWasPurchased', function($event)
    {
        // 处理事件...

        return false;
    });
```

## 事件处理队列

需要把事件处理程序放到 [队列][49] 吗？这不能变得再更简单了。当你产生处理程序，简单地使用 `\--queued` 旗标：

```
    php artisan handler:event SendPurchaseConfirmation --event=PodcastWasPurchased --queued
```

这将会产生一个实现了 `IlluminateContractsQueueShouldBeQueued` 接口的处理程序类。这样就可以了！现在当这个处理程序因为事件发生被调用，它将会被事件配送器自动地排进队列。

当处理程序被队列执行，如果没有异常被丢出，在执行后该队列中的任务将会自动被删除。你也可以手动取用队列中的任务的 `delete` 和 `release` 方法。队列处理程序默认会引入的 `IlluminateQueueInteractsWithQueue` trait，让你可以取用这些方法：

```
    public function handle(PodcastWasPurchased $event)
    {
        if (true)
        {
            $this->release(30);
        }
    }
```

如果你想要把一个已存在的处理程序转换成一个队列的处理程序，简单地手动添加 `ShouldBeQueued` 接口到类。

## 事件订阅者

#### 定义事件订阅者

事件订阅者是个可以从类自身里面订阅多个事件的类。订阅者应该定义 `subscribe` 方法，事件配送器实体将会被传递到这个方法：

```
    class UserEventHandler {

        /**
         * 处理用户登录事件。
         */
        public function onUserLogin($event)
        {
            //
        }

        /**
         * 处理用户注销事件。
         */
        public function onUserLogout($event)
        {
            //
        }

        /**
         * 注册监听器给订阅者。
         *
         * @param  IlluminateEventsDispatcher  $events
         * @return array
         */
        public function subscribe($events)
        {
            $events->listen('AppEventsUserLoggedIn', 'UserEventHandler@onUserLogin');

            $events->listen('AppEventsUserLoggedOut', 'UserEventHandler@onUserLogout');
        }

    }
```

#### 注册事件订阅者

当定义了订阅者后，可以使用 `Event` 类注册。

```
    $subscriber = new UserEventHandler;

    Event::subscribe($subscriber);
```


你也可以使用 [服务容器][26] 自动解析订阅者。简单地传递订阅者的名字给 `subscribe` 方法就可以做到：

```
Event::subscribe('UserEventHandler');
```