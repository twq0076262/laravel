# Facades

## 介绍

Facades 提供一个静态接口给在应用程序的 [服务容器](container.md) 中可以取用的类。Laravel 附带许多 facades，甚至你可能已经在不知情的状况下使用过它们！Laravel 的「facades」作为在 IoC 容器里面的基础类的静态代理，提供的语法有简洁、易表达的优点，同时维持比传统的静态方法更高的可测试性和弹性。

有时，你或许会希望为应用程序和扩展包建立自己的 facades，所以让我们来探索这些类的概念、开发和用法。

**`注意： 在深入了解 facades 之前，强烈建议你先熟悉 Laravel IoC 容器.`**

## 解释

在 Laravel 应用程序的环境中，`facade` 是个提供从容器访问对象的类。Facade 类是让这个机制可以运作的原因。Laravel 的 facades 和你建立的任何自定义 facades，将会继承基本的 `Facade` 类。

你的 facade 类只需要去实现一个方法：`getFacadeAccessor`。`getFacadeAccessor` 方法的工作是定义要从容器解析什么。基本的 Facade 类利用 `__callStatic()` 魔术方法来从你的 `facade` 调用到解析出来的对象。

所以当你对 facade 调用，例如 `Cache::get`，Laravel 从 IoC 容器解析缓存管理类出来，并对该类调用 get 方法。用专业口吻来说，Laravel Facades 是使用 Laravel IoC 容器作为服务定位器的便捷语法。


## 实际用法

在下面的例子，对 Laravel 缓存系统进行调用。简单看过去这代码，有人可能会以为静态方法 `get` 是对 `Cache` 类调用。

```
$value = Cache::get('key');
```
然而，如果我们去看 `Illuminate\Support\Facades\Cache` 类，你将会看到它没有静态方法 `get`：

```
class Cache extends Facade {

    /**
     * 取得组件的注册名称
     *
     * @return string
     */
    protected static function getFacadeAccessor() { return 'cache'; }

}
```

Cache 类继承基本的 `Facade` 类并定义一个 getFacadeAccessor() 方法。记住，这个方法的工作是返回 IoC 绑定的名称。

当用户在 `Cache` 的 facade 上参考任何的静态方法，Laravel 会从 IoC 容器解析被绑定的 `cache` ，并对该对象执行被请求的方法 (在这个例子中， `get`)。

所以我们的 `Cache::get` 调用可以被重写成像这样：

```
$value = $app->make('cache')->get('key');
```

### 导入 Facades

记住，如果你在控制器有使用命名空间的情况下使用 facade，你会需要导入 facade 类进入命名空间。所有的 facades 存在于全局命名空间：

```
<?php namespace App\Http\Controllers;

use Cache;

class PhotosController extends Controller {

    /**
     * 取得所有的应用程序相片。
     *
     * @return Response
     */
    public function index()
    {
        $photos = Cache::get('photos');

        //
    }

}
```

## 建立 Facades

为你自己的应用程序或扩展包建立 facade 是很简单的。你只需要 3 个东西：

一个 IoC 绑定。
一个 facade 类。
一个 facade 别名配置。
让我们来看个例子。这里有一个定义为 `PaymentGateway\Payment` 的类。

```
namespace PaymentGateway;

class Payment {

    public function process()
    {
        //
    }

}
```
我们需要可以从 IoC 容器解析出这个类。所以，让我们来加上一个绑定到服务提供者：

```
App::bind('payment', function()
{
    return new \PaymentGateway\Payment;
});
```
注册这个绑定的好方式是建立新的 [服务提供者](container.md)  命名为 `PaymentServiceProvider`，并把这个绑定加到 `register` 方法。然后你可以配置 Laravel 从 `config/app.php` 配置文件加载你的服务提供者。

接下来，我们可以建立我们自己的 facade 类：

```
use Illuminate\Support\Facades\Facade;

class Payment extends Facade {

    protected static function getFacadeAccessor() { return 'payment'; }

}
```

最后，如果我们希望，可以在 `config/app.php` 配置文件为 facade 加个别名到 `aliases` 数组。现在我们可以在 `Payment` 类的实例上调用 `process` 方法。

```
Payment::process();
```
### 自动加载别名的附注

在 `aliases` 数组中的类在某些实例中不能使用，因为 PHP 将不会尝试去自动加载未定义的类型提示类。如果 `\ServiceWrapper\ApiTimeoutException` 命别名为 `ApiTimeoutException`，即便有异常被抛出，在 `\ServiceWrapper` 命名空间外面的 `catch(ApiTimeoutException $e)` 将永远捕捉不到异常。类似的问题在有类型提示的别名类一样会发生。唯一的替代方案就是放弃别名并用 `use` 在每一个文件的最上面引入你希望类型提示的类。


## 模拟 Facades

单元测试是为什么现在 facades 采用这样的工作方式的主要因素。事实上，可测试性甚至是 facades 存在的主要理由。想要获得更多信息，请查看文档的 [模拟 facades](facades.md) 章节。


## Facade 类参考

你将会在下面找到每一个 facade 和它的基础类。这是个可以从一个给定的 facade 根源快速地深入 API 文档的有用工具。可应用的 IoC 绑定 关键字也包含在里面。

| Facade         | Class                                           | IoC Binding      |
| -------------- | ----------------------------------------------- | ---------------- |
| App            | [IlluminateFoundationApplication](http://laravel.com/api/5.0/Illuminate/Foundation/Application.html)           | `app`            |
| Artisan        | [IlluminateConsoleApplication](http://laravel.com/api/5.0/Illuminate/Console/Application.html)              | `artisan`        |
| Auth           | [IlluminateAuthAuthManager](http://laravel.com/api/5.0/Illuminate/Auth/AuthManager.html)                 | `auth`           |
| Auth (实例)      | [IlluminateAuthGuard](http://laravel.com/api/5.0/Illuminate/Auth/Guard.html)                       |                  |
| Blade          | [IlluminateViewCompilersBladeCompiler](http://laravel.com/api/5.0/Illuminate/View/Compilers/BladeCompiler.html)      | `blade.compiler` |
| Bus            | [IlluminateContractsBusDispatcher](http://laravel.com/api/5.0/Illuminate/Contracts/Bus/Dispatcher.html)          |                  |
| Cache          | [IlluminateCacheRepository](http://laravel.com/api/5.0/Illuminate/Cache/Repository.html)                 | `cache`          |
| Config         | [IlluminateConfigRepository](http://laravel.com/api/5.0/Illuminate/Config/Repository.html)                | `config`         |
| Cookie         | [IlluminateCookieCookieJar](http://laravel.com/api/5.0/Illuminate/Cookie/CookieJar.html)                 | `cookie`         |
| Crypt          | [IlluminateEncryptionEncrypter](http://laravel.com/api/5.0/Illuminate/Encryption/Encrypter.html)             | `encrypter`      |
| DB             | [IlluminateDatabaseDatabaseManager](http://laravel.com/api/5.0/Illuminate/Database/DatabaseManager.html)         | `db`             |
| DB (实例)        | [IlluminateDatabaseConnection](http://laravel.com/api/5.0/Illuminate/Database/Connection.html)              |                  |
| Event          | [IlluminateEventsDispatcher](http://laravel.com/api/5.0/Illuminate/Events/Dispatcher.html)                | `events`         |
| File           | [IlluminateFilesystemFilesystem](http://laravel.com/api/5.0/Illuminate/Filesystem/Filesystem.html)            | `files`          |
| Hash           | [IlluminateContractsHashingHasher](http://laravel.com/api/5.0/Illuminate/Contracts/Hashing/Hasher.html)          | `hash`           |
| Input          | [IlluminateHttpRequest](http://laravel.com/api/5.0/Illuminate/Http/Request.html)                     | `request`        |
| Lang           | [IlluminateTranslationTranslator](http://laravel.com/api/5.0/Illuminate/Translation/Translator.html)           | `translator`     |
| Log            | [IlluminateLogWriter](http://laravel.com/api/5.0/Illuminate/Log/Writer.html)                       | `log`            |
| Mail           | [IlluminateMailMailer](http://laravel.com/api/5.0/Illuminate/Mail/Mailer.html)                      | `mailer`         |
| Password       | [IlluminateAuthPasswordsPasswordBroker](http://laravel.com/api/5.0/Illuminate/Auth/Passwords/PasswordBroker.html)     | `auth.password`  |
| Queue          | [IlluminateQueueQueueManager](http://laravel.com/api/5.0/Illuminate/Queue/QueueManager.html)               | `queue`          |
| Queue (实例)     | [IlluminateQueueQueueInterface](http://laravel.com/api/5.0/Illuminate/Queue/QueueInterface.html)             |                  |
| Queue (基础类)    | [IlluminateQueueQueue](http://laravel.com/api/5.0/Illuminate/Queue/Queue.html)                      |                  |
| Redirect       | [IlluminateRoutingRedirector](http://laravel.com/api/5.0/Illuminate/Routing/Redirector.html)               | `redirect`       |
| Redis          | [IlluminateRedisDatabase](http://laravel.com/api/5.0/Illuminate/Redis/Database.html)                   | `redis`          |
| Request        | [IlluminateHttpRequest](http://laravel.com/api/5.0/Illuminate/Http/Request.html)                     | `request`        |
| Response       | [IlluminateContractsRoutingResponseFactory](http://laravel.com/api/5.0/Illuminate/Contracts/Routing/ResponseFactory.html) |                  |
| Route          | [IlluminateRoutingRouter](http://laravel.com/api/5.0/Illuminate/Routing/Router.html)                   | `router`         |
| Schema         | [IlluminateDatabaseSchemaBlueprint](http://laravel.com/api/5.0/Illuminate/Database/Schema/Blueprint.html)         |                  |
| Session        | [IlluminateSessionSessionManager](http://laravel.com/api/5.0/Illuminate/Session/SessionManager.html)           | `session`        |
| Session (实例)   | [IlluminateSessionStore](http://laravel.com/api/5.0/Illuminate/Session/Store.html)                    |                  |
| Storage        | [IlluminateContractsFilesystemFactory](http://laravel.com/api/5.0/Illuminate/Contracts/Filesystem/Factory.html)      | `filesystem`     |
| URL            | [IlluminateRoutingUrlGenerator](http://laravel.com/api/5.0/Illuminate/Routing/UrlGenerator.html)            | `url`            |
| Validator      | [IlluminateValidationFactory](http://laravel.com/api/5.0/Illuminate/Validation/Factory.html)               | `validator`      |
| Validator (实例) | [IlluminateValidationValidator](http://laravel.com/api/5.0/Illuminate/View/Factory.html)             |                  |
| View           | [IlluminateViewFactory](http://laravel.com/api/5.0/Illuminate/View/View.html)                     | `view`           |
| View (实例)      | [IlluminateViewView](http://laravel.com/api/5.0/Illuminate/View/View.html)                       |                  |
