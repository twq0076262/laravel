# HTTP 路由

## 基本路由

您将在 `app/Http/routes.php` 中定义应用中的大多数路由，这个文件加载了 `App\Providers\RouteServiceProvider` 类。 大多数基本的 Laravel 路由都只接受一个 URI 和 一个 `闭包(Closure)` 参数：

### 基本 GET 路由

```
Route::get('/', function()
{
    return 'Hello World';
});
```
### 其他基础路由


```
Route::post('foo/bar', function()
{
    return 'Hello World';
});
Route::put('foo/bar', function()
{
    //
});

Route::delete('foo/bar', function()
{
    //
});
```
### 为多种请求注册路由

```
Route::match(['get', 'post'], '/', function()
{
    return 'Hello World';
});
```
### 注册路由响应所有 HTTP 请求

```
Route::any('foo', function()
{
    return 'Hello World';
});
```
通常情况下，您将会需要为您的路由产生 URL，您可以使用 `url` 辅助函数来操作：

```
$url = url('foo');
```

## CSRF 保护

Laravel 提供简易的方法，让您可以保护您的应用程序不受到 CSRF (跨网站请求伪造) 攻击。跨网站请求伪造是一种恶意的攻击，借以代表经过身份验证的用户执行未经授权的命令。

Laravel 会自动在每一位用户的 `session` 中放置随机的 token ，这个 token 将被用来确保经过验证的用户是实际发出请求至应用程序的用户：

### 插入 CSRF Token 到表单

```
<input type="hidden" name="_token" value="<?php echo csrf_token(); ?>">
```
当然也可以在 Blade 模板引擎使用：

```
<input type="hidden" name="_token" value="{{ csrf_token() }}">
```
您不需要手动验证在 POST、PUT、DELETE 请求的 CSRF token。 `VerifyCsrfToken` HTTP 中间件将保存在 session 中的请求输入的 token 配对来验证 token 。

### X-CSRF-TOKEN

除了寻找 CSRF token 作为「POST」参数，中间件也检查 `X-XSRF-TOKEN` 请求头，比如，你可以把 token 存放在 meta 标签中, 然后使用 jQuery 将它加入到所有的请求头中：

```
<meta name="csrf-token" content="{{ csrf_token() }}" />

$.ajaxSetup({
    headers: {
        'X-CSRF-TOKEN': $('meta[name="csrf-token"]').attr('content')
    }
});
```
现在所有的 AJAX 请求会自动加入 CSRF token：

```
$.ajax({
  url: "/foo/bar",
})
```

### X-XSRF-TOKEN

Laravel 也在 cookie 中存放了名为 `XSRF-TOKEN` 的 CSRF token。你可以使用这个 cookie 值来设置 `X-XSRF-TOKEN` 请求头。一些 Javascript 框架，比如 Angular ，会自动设置这个值。


**`注意： X-CSRF-TOKEN 和 X-XSRF-TOKEN 的不同点在于前者使用的是纯文本而后者是一个加密的值，因为在 Laravel 中 cookies 始终是被加密过的。如果你使用 csrf_token() 函数来作为 token 的值， 你需要设置 X-CSRF-TOKEN 请求头。`**


## 方法欺骗

HTML 表单没有支持 `PUT` 、`PATCH` 或 `DELETE` 请求。所以当定义 `PUT` 、`PATCH` 以及 `DELETE` 路由并在 HTML 表单中被调用的时候，您将需要添加隐藏 `_method` 字段在表单中。

发送的 `_method` 字段对应的值会被当做 HTTP 请求方法。举例来说：

```
<form action="/foo/bar" method="POST">
    <input type="hidden" name="_method" value="PUT">
    <input type="hidden" name="_token" value="<?php echo csrf_token(); ?>">
</form>
```

## 路由参数

当然，您可以获取请求路由的 URI 区段。

### 基础路由参数

```
Route::get('user/{id}', function($id)
{
    return 'User '.$id;
});
```
### 可选择的路由参数

```
Route::get('user/{name?}', function($name = null)
{
    return $name;
});
```
### 带默认值的路由参数

```
Route::get('user/{name?}', function($name = 'John')
{
    return $name;
});
```
### 使用正则表达式限制参数

```
Route::get('user/{name}', function($name)
{
    //
})
->where('name', '[A-Za-z]+');

Route::get('user/{id}', function($id)
{
    //
})
->where('id', '[0-9]+');
```
### 使用条件限制数组

```
Route::get('user/{id}/{name}', function($id, $name)
{
    //
})
->where(['id' => '[0-9]+', 'name' => '[a-z]+'])
```
### 定义全局模式

如果你想让特定路由参数总是遵询特定的正则表达式，可以使用 `pattern` 方法。在 `RouteServiceProvider` 的 `boot` 方法里定义模式：

```
$router->pattern('id', '[0-9]+');
```
定义模式之后，会作用在所有使用这个特定参数的路由上：

```
Route::get('user/{id}', function($id)
{
    // 只有 {id} 是数字才被调用。
});
```
### 取得路由参数

如果需要在路由外部取得其参数，使用 `input` 方法：

```
if ($route->input('id') == 1)
{
    //
}
```
你也可以使用 `Illuminate\Http\Request` 实体取得路由参数。当前请求的实例可以通过 `Request facade` 取得，或透过类型提示 `Illuminate\Http\Request` 注入依赖：

```
use Illuminate\Http\Request;

Route::get('user/{id}', function(Request $request, $id)
{
    if ($request->route('id'))
    {
        //
    }
});
```
## 命名路由

命名路由让你更方便于产生 URL 与重定向特定路由。您可以用 `as` 的数组键值指定名称给路由：

```
Route::get('user/profile', ['as' => 'profile', function()
{
    //
}]);
```
也可以为控制器动作指定路由名称：

```
Route::get('user/profile', [
    'as' => 'profile', 'uses' => 'UserController@showProfile'
]);
```
现在你可以使用路由名称产生 URL 或进行重定向：

```
$url = route('profile');

$redirect = redirect()->route('profile');
```

`currentRouteName` 方法会返回目前请求的路由名称：

```
$name = Route::currentRouteName();
```

## 路由群组

有时候您需要嵌套过滤器到群组的路由上。不需要为每个路由去嵌套过滤器，您只需使用路由群组：

```
Route::group(['middleware' => 'auth'], function()
{
    Route::get('/', function()
    {
        // Has Auth Filter
    });

    Route::get('user/profile', function()
    {
        // Has Auth Filter
    });
});
```
您一样可以在 `group` 数组中使用 `namespace` 参数，指定在这群组中控制器的命名空间：

```
Route::group(['namespace' => 'Admin'], function()
{
    //
});
```

**`注意： 在默认情况下，RouteServiceProvider 包含内置您命名空间群组的 routes.php 文件，让您不须使用完整的命名空间就可以注册控制器路由。`**


### 子域名路由

Laravel 路由一样可以处理通配符的子域名，并且从域名中传递您的通配符参数：

#### 注册子域名路由

```
Route::group(['domain' => '{account}.myapp.com'], function()
{

    Route::get('user/{id}', function($account, $id)
    {
        //
    });

});
```

### 路由前缀

群组路由可以通过群组的描述数组中使用 `prefix` 选项，将群组内的路由加上前缀：

```
Route::group(['prefix' => 'admin'], function()
{

    Route::get('user', function()
    {
        //
    });

});
```
## 路由模型绑定

Laravel 模型绑定提供方便的方式将模型实体注入到您的路由中。例如，比起注入 User ID ，你可以选择注入符合给定 ID 的 User 类实体。

首先，使用路由的 `model` 方法指定特定参数要对应的类，您应该在 `RouteServiceProvider::boot` 方法定义您的模型绑定：

### 绑定参数至模型

```
public function boot(Router $router)
{
    parent::boot($router);

    $router->model('user', 'App\User');
}
```
然后定义一个有 `{user}` 参数的路由：

```
Route::get('profile/{user}', function(App\User $user)
{
    //
});
```
因为我们已经将 `{user}` 参数绑定到 `App\User` 模型，所以 `User` 实体将被注入到路由。所以举例来说，请求至  `profile/1 `将注入 ID 为 1 的 `User` 实体。


**`注意： 如果在数据库中找不到匹配的模型实体，将引发 404 错误。`**

如果您想要自定「没有找到」的行为，将闭包作为第三个参数传入 `model `方法：

```
Route::model('user', 'User', function()
{
    throw new NotFoundHttpException;
});
```
如果您想要使用您自己决定的逻辑，您应该使用 `Router::bind`方法。闭包通过 `bind` 方法将传递 URI 区段数值，并应该返回您想要被注入路由的类实体：

```
Route::bind('user', function($value)
{
    return User::where('name', $value)->first();
});
```

## 抛出 404 错误

这里有两种方法从路由手动触发 404 错误。首先，您可以使用 `abort` 辅助函数：

```
abort(404);
```
`abort` 辅助函数只是简单抛出带有特定状态代码的 `Symfony\Component\HttpFoundation\Exception\HttpException` 。

第二，您可以手动抛出 `Symfony\Component\HttpKernel\Exception\NotFoundHttpException` 的实体。

有关如何处理 404 异常状况和自定响应的更多信息，可以参考错误章节内的文档。