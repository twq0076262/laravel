# Contracts

简介
为什么用 Contracts？
Contract 参考
如何使用 Contracts

## 简介

Laravel 的 Contracts 是一组定义了框架核心服务的接口（ interfaces ）。例如，`Queue` contract 定义了队列任务所需要的方法，而 `Mailer` contract 定义了发送 e-mail 需要的方法。

在 Laravel 框架里，每个 contract 都提供了一个对应的实现。例如， Laravel 提供了有多种驱动的 `Queue` 的实现，而根据 SwiftMailer 实现了 `Mailer`。

Laravel 所有的 contracts 都放在各自的 Github repository。除了提供了所有可用的 contracts 一个快速的参考，也可以单独作为一个低耦合的扩展包让其他扩展包开发者使用。


## 为什么用 Contracts？

你可能有很多关于 contracts 的问题。如为什么要使用接口？使用接口会不会变的更复杂？

让我们用下面的标题来解释为什么要使用接口：低耦合和简单性。

### 低耦合

首先，看一些强耦合的缓存实现代码。如下：

```
<?php namespace App\Orders;

class Repository {

    /**
     * The cache.
     */
    protected $cache;

    /**
     * Create a new repository instance.
     *
     * @param  \SomePackage\Cache\Memcached  $cache
     * @return void
     */
    public function __construct(\SomePackage\Cache\Memcached $cache)
    {
        $this->cache = $cache;
    }

    /**
     * Retrieve an Order by ID.
     *
     * @param  int  $id
     * @return Order
     */
    public function find($id)
    {
        if ($this->cache->has($id))
        {
            //
        }
    }

}
```

在上面的类里，代码跟缓存实现之间是强耦合。理由是它会依赖于扩展包库（ package vendor ）的特定缓存类。一旦这个扩展包的 API 更改了，我们的代码也要跟着改变。

同样的，如果想要将底层的缓存技术（比如 Memcached ）抽换成另一种（像 Redis ），又一次的我们必须修改这个 repository 类。我们的 repository 不应该知道这么多关于谁提供了数据，或是如何提供等等细节。

### 比起上面的做法，我们可以改用一个简单、和扩展包无关的接口来改进代码：

```
<?php namespace App\Orders;

use Illuminate\Contracts\Cache\Repository as Cache;

class Repository {

    /**
     * Create a new repository instance.
     *
     * @param  Cache  $cache
     * @return void
     */
    public function __construct(Cache $cache)
    {
        $this->cache = $cache;
    }

}
```

现在上面的代码没有跟任何扩展包耦合，甚至是 Laravel。既然 contracts 扩展包没有包含实现和任何依赖，你可以很简单的对任何 contract 进行实现，你可以很简单的写一个替换的实现，甚至是替换 contracts，让你可以替换缓存实现而不用修改任何用到缓存的代码。

### 简单性

当所有的 Laravel 服务都简洁的使用简单的接口定义，就能够很简单的决定一个服务需要提供的功能。**可以将 contracts 视为说明框架特色的简洁文档.**

除此之外，当你依赖简洁的接口，你的代码能够很简单的被了解和维护。比起搜索一个大型复杂的类里有哪些可用的方法，你有一个简单，干净的接口可以参考。


## Contract 参考

以下是大部分 Laravel Contracts 的参考，以及相对应的 "facade"

Contract	Laravel 4.x Facade
Illuminate\Contracts\Auth\Guard	Auth
Illuminate\Contracts\Auth\PasswordBroker	Password
Illuminate\Contracts\Bus\Dispatcher	Bus
Illuminate\Contracts\Cache\Repository	Cache
Illuminate\Contracts\Cache\Factory	Cache::driver()
Illuminate\Contracts\Config\Repository	Config
Illuminate\Contracts\Container\Container	App
Illuminate\Contracts\Cookie\Factory	Cookie
Illuminate\Contracts\Cookie\QueueingFactory	Cookie::queue()
Illuminate\Contracts\Encryption\Encrypter	Crypt
Illuminate\Contracts\Events\Dispatcher	Event
Illuminate\Contracts\Filesystem\Cloud	 
Illuminate\Contracts\Filesystem\Factory	File
Illuminate\Contracts\Filesystem\Filesystem	File
Illuminate\Contracts\Foundation\Application	App
Illuminate\Contracts\Hashing\Hasher	Hash
Illuminate\Contracts\Logging\Log	Log
Illuminate\Contracts\Mail\MailQueue	Mail::queue()
Illuminate\Contracts\Mail\Mailer	Mail
Illuminate\Contracts\Queue\Factory	Queue::driver()
Illuminate\Contracts\Queue\Queue	Queue
Illuminate\Contracts\Redis\Database	Redis
Illuminate\Contracts\Routing\Registrar	Route
Illuminate\Contracts\Routing\ResponseFactory	Response
Illuminate\Contracts\Routing\UrlGenerator	URL
Illuminate\Contracts\Support\Arrayable	 
Illuminate\Contracts\Support\Jsonable	 
Illuminate\Contracts\Support\Renderable	 
Illuminate\Contracts\Validation\Factory	Validator::make()
Illuminate\Contracts\Validation\Validator	 
Illuminate\Contracts\View\Factory	View::make()
Illuminate\Contracts\View\View	 

## 如何使用 Contracts

所以，要如何实现一个 contract？实际上非常的简单。很多 Laravel 的类都是经由 service container 解析，包含控制器，事件监听，过滤器，队列任务，甚至是闭包。所以，要实现一个 contract，你可以在类的构造器使用「类型提示」解析类。例如，看下面的事件处理程序：

```
<?php namespace App\Handlers\Events;

use App\User;
use App\Events\NewUserRegistered;
use Illuminate\Contracts\Redis\Database;

class CacheUserInformation {

    /**
     * Redis 数据库实现
     */
    protected $redis;

    /**
     * 建立新的事件处理实例
     *
     * @param  Database  $redis
     * @return void
     */
    public function __construct(Database $redis)
    {
        $this->redis = $redis;
    }

    /**
     * 处理事件
     *
     * @param  NewUserRegistered  $event
     * @return void
     */
    public function handle(NewUserRegistered $event)
    {
        //
    }

}
```
当事件监听被解析时，服务容器会经由类构造器参数的类型提示，注入适当的值。要知道怎么注册更多服务容器，参考这个文档。