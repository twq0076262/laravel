# HTTP 中间件

## 简介

HTTP 中间件提供一个方便的机制来过滤进入应用程序的 HTTP 请求，例如，Laravel 默认包含了一个中间件来检验用户身份验证，如果用户没有经过身份验证，中间件会将用户导向登录页面，然而，如果用户通过身份验证，中间件将会允许这个请求进一步继续前进。

当然，除了身份验证之外，中间件也可以被用来执行各式各样的任务，CORS 中间件负责替所有即将离开程序的响应加入适当的响应头，一个日志中间件可以记录所有传入应用程序的请求。 Laravel 框架已经内置一些中间件，包括维护、身份验证、CSRF 保护，等等。所有的中间件都位于 `app/Http/Middleware` 目录内。


## 建立中间件

要建立一个新的中间件，可以使用 `make:middleware` 这个 Artisan 命令：

```
php artisan make:middleware OldMiddleware
```
此命令将会 在 `app/Http/Middleware` 目录内置立一个名称为 `OldMiddleware` 的类。在这个中间件内我们只允许 `年龄` 大于 200 的才能访问路由，否则，我们会将用户重新导向 「home」 的 URI 。

```
<?php namespace App\Http\Middleware;

class OldMiddleware {

    /**
     * Run the request filter.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \Closure  $next
     * @return mixed
     */
    public function handle($request, Closure $next)
    {
        if ($request->input('age') < 200)
        {
            return redirect('home');
        }

        return $next($request);
    }

}
```

如你所见，若是 `年龄` 小于 `200` ，中间件将会返回 HTTP 重定向给客户端，否则，请求将会进一步传递到应用程序。只需调用带有 `$request` 的 `$next` 方法，即可将请求传递到更深层的应用程序(允许跳过中间件) HTTP 请求在实际碰触到应用程序之前，最好是可以层层通过许多中间件，每一层都可以对请求进行检查，甚至是完全拒绝请求。

### Before / After 中间件

在一个请求前后指定某个中间件取决于这个中间件自身。这个中间件可以执行在请求前执行一些 前置 操作：

```
<?php namespace App\Http\Middleware;

class BeforeMiddleware implements Middleware {

    public function handle($request, Closure $next)
    {
        // Perform action

        return $next($request);
    }
}
```
然后，这个中间件也可以在请求后执行一些 **后置** 操作：

```
<?php namespace App\Http\Middleware;

class AfterMiddleware implements Middleware {

    public function handle($request, Closure $next)
    {
        $response = $next($request);

        // Perform action

        return $response;
    }
}
```

## 注册中间件

### 全局中间件

若是希望中间件被所有的 HTTP 请求给执行，只要将中间件的类加入到 `app/Http/Kernel.php` 的 `$middleware` 属性清单列表中。

### 指派中间件给路由

如果你要指派中间件给特定的路由，你得先将中间件在 `app/Http/Kernel.php` 配置一个键值，默认情况下，这个文件内的 $routeMiddleware 属性已包含了 Laravel 目前配置的中间件，你只需要在清单列表中加上一组自定义的键值即可。 中间件一旦在 HTTP kernel 文件内被定义，你即可在路由选项内使用 `middleware` 键值来指派：

```
Route::get('admin/profile', ['middleware' => 'auth', function()
{
    //
}]);
```

## 可终止中间件

有些时候中间件需要在 HTTP 响应已被发送到用户端之后才执行，例如，Laravel 内置的 「session」 中间件，保存 session 数据是在响应已被发送到用户端 之后 才执行。为了做到这一点，你需要定义中间件为「可终止的」。

```
use Illuminate\Contracts\Routing\TerminableMiddleware;

class StartSession implements TerminableMiddleware {

    public function handle($request, $next)
    {
        return $next($request);
    }

    public function terminate($request, $response)
    {
        // Store the session data...
    }

}
```

如你所见，除了定义 `handle` 方法之外， `TerminableMiddleware` 定义一个 `terminate` 方法。这个方法接收请求和响应。一旦定义了 terminable 中间件，你需要将它增加到 HTTP kernel 文件的全局中间件清单列表中。

