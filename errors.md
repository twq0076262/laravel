# 错误与日志

## 配置

应用程序的日志功能配置在 `IlluminateFoundationBootstrapConfigureLogging` 启动类中。这个类使用 `config/app.php` 配置文件的 `log` 配置选项。

日志工具默认使用每天的日志文件；然而，你可以依照需求自定义这个行为。因为 Laravel 使用流行的 [Monolog](https://github.com/Seldaek/monolog) 日志函数库，你可以利用很多 Monolog 提供的处理进程。

例如，如果你想要使用单一日志文件，而不是每天一个日志文件，你可以对 `config/app.php` 配置文件做下面的变更：

```
    'log' => 'single'
```

Laravel 提供立即可用的 `single` 、 `daily` 和 `syslog` 日志模式。然而，你可以通过覆写 `ConfigureLogging` 启动类，依照需求自由地自定义应用程序的日志。

### 错误细节

`config/app.php` 配置文件的 `app.debug` 配置选项控制应用程序透过浏览器显示错误细节。配置选项默认参照 `.env` 文件的 `APP_DEBUG` 环境变量。

进行本地开发时，你应该配置 `APP_DEBUG` 环境变量为 `true` 。 **在上线环境，这个值应该永远为 `false` 。**

## 错误处理

所有的异常都由 `AppExceptionsHandler` 类处理。这个类包含两个方法： `report` 和 `render` 。

`report` 方法用来记录异常或把异常传递到外部服务，例如： [BugSnag](https://github.com/Seldaek/monolog) 。默认情况下， `report` 方法只基本实现简单地传递异常到父类并于父类记录异常。然而，你可以依你所需自由地记录异常。如果你需要使用不同的方法来报告不同类型的异常，你可以使用 PHP 的 `instanceof` 比较运算符：

```
    /**
     * 报告或记录异常。
     *
     * 这是一个发送异常到 Sentry、Bugsnag 等服务的好地方。
     *
     * @param  Exception  $e
     * @return void
     */
    public function report(Exception $e)
    {
        if ($e instanceof CustomException)
        {
            //
        }

        return parent::report($e);
    }
```

`render` 方法负责把异常转换成应该被传递回浏览器的 HTTP 响应。默认情况下，异常会被传递到基础类并帮你产生响应。然而，你可以自由的检查异常类型或返回自定义的响应。

异常处理进程的 `dontReport` 属性是个数组，包含应该不要被纪录的异常类型。由 404 错误导致的异常默认不会被写到日志文件。你可以依照需求添加其他类型的异常到这个数组。

## HTTP 异常

有一些异常是描述来自服务器的 HTTP 错误码。例如，这可能是个「找不到页面」错误 (404)、「未授权错误」(401)，或甚至是工程师导致的 500 错误。使用下面的方法来返回这样一个响应：

```
    abort(404);
```

或是你可以选择提供一个响应：

```
    abort(403, 'Unauthorized action.');
```

你可以在请求的生命周期中任何时间点使用这个方法。

### 自定义 404 错误页面

要让所有的 404 错误返回自定义的视图，请建立一个 `resources/views/errors/404.blade.php` 文件。应用程序将会使用这个视图处理所有发生的 404 错误。

## 日志

Laravel 日志工具在强大的 [Monolog](https://github.com/seldaek/monolog) 函数库上提供一层简单的功能。Laravel 默认为应用程序建立每天的日志文件在 `storage/logs` 目录。你可以像这样把信息写到日志：

```
    Log::info('This is some useful information.');

    Log::warning('Something could be going wrong.');

    Log::error('Something is really going wrong.');
```

日志工具提供定义在 [RFC 5424](http://tools.ietf.org/html/rfc5424)的七个级别：**debug**、**info**、**notice**、**warning**、**error**、**critical** 和 **alert**。

也可以传入上下文相关的数据数组到日志方法里：

```
    Log::info('Log message', ['context' => 'Other helpful information']);
```

Monolog 有很多其他的处理方法可以用在日志上。如有需要，你可以取用 Laravel 底层使用的 Monolog 实例：

```
    $monolog = Log::getMonolog();
```

你也可以注册事件来捕捉所有传到日志的消息：

#### 注册日志事件监听器

```
    Log::listen(function($level, $message, $context)
    {
        //
    });
```