# 认证

## 介绍

Laravel 让实现认证机制变得非常简单。事实上，几乎所有的设置默认就已经完成了。有关认证的配置文件都放在 `config/auth.php` 里，而在这些文件里也都包含了良好的注释描述每一个选项的所对应的认证服务。

Laravel 默认在 `app` 文件夹内就包含了一个使用默认 Eloquent 认证驱动的 `App\User`模型。

注意：当为这个认证模型设计数据库结构时，密码字段至少有60个字符宽度。同样，在开始之前，请先确认您的 `users` (或其他同义) 数据库表包含一个名为 `remember_token` 长度为 100 的`string`类型、可接受 null 的字段。这个字段将会被用来储存「记住我」的 session token。也可以通过在迁移文件中使用 `$table->rememberToken();` 方法。 当然， Laravel 5 自带的 migrations 里已经设定了这些字段。

假如您的应用程序并不是使用 Eloquent ，您也可以使用 Laravel 的查询构造器做 `database` 认证驱动。


## 用户认证

Laravel 已经预设了两个认证相关的控制器。 `AuthController` 处理新的用户注册和「登陆」，而 `PasswordController` 可以帮助已经注册的用户重置密码。

每个控制器使用 trait 引入需要的方法。在大多数应用上，你不需要修改这些控制器。这些控制器用到的视图放在 `resources/views/auth` 目录下。你可以依照需求修改這些视图。

### 用户注册

要修改应用注册新用户时所用到的表单字段，可以修改 `App\Services\Registrar` 类。这个类负责验证和建立应用的新用户。

`Registrar` 的 `validator` 方法包含新用户时的验证规则，而 `Registrar` 的 `create` 方法负责在数据库中建立一条新的 `User` 记录。你可以自由的修改这些方法。`Registrar` 方法是通过`AuthenticatesAndRegistersUsers` trait 的中的 `AuthController` 调用的。

### 手动认证

如果你不想使用预设的 `AuthController`，你需要直接使用 Laravel 的身份验证类来管理用户认证。别担心，这也很简单的！首先，让我们看看 `attempt` 方法：

```
<?php namespace App\Http\Controllers;

use Auth;
use Illuminate\Routing\Controller;

class AuthController extends Controller {

    /**
     * Handle an authentication attempt.
     *
     * @return Response
     */
    public function authenticate()
    {
        if (Auth::attempt(['email' => $email, 'password' => $password]))
        {
            return redirect()->intended('dashboard');
        }
    }

}
```

`attempt` 方法可以接受由键值对组成的数组作为第一个参数。`password` 的值会先进行 哈希。数组中的其他 值会被用来查询数据表里的用户。所以，在上面的示例中，会根据 `email` 列的值找出用户。如果找到该用户，会比对数据库中存储的哈希过的密码以及数组中的哈希过后的 `password`值。假设两个哈希后的密码相同，会重新为用户启动认证通过的 session。

如果认证成功， `attempt` 将会返回 `true`。否则则返回 `false`。

**`注意：在上面的示例中，并不一定要使用 email 字段，这只是作为示例。你应该使用对应到数据表中的「username」的任何键值。
intended 方法会重定向到用户尝试要访问的 URL ， 其值会在进行认证过滤前被存起来。也可以给这个方法传入一个预设的 URI，防止重定向的网址无法使用。`**

### 以特定条件验证用户

在认证过程中，你可能会想要加入额外的认证条件：

```
if (Auth::attempt(['email' => $email, 'password' => $password, 'active' => 1]))
{
    // The user is active, not suspended, and exists.
}
```
### 判断用户是否已验证

判断一个用户是否已经登录，你可以使用 `check` 方法：

```
if (Auth::check())
{
    // The user is logged in...
}
```

### 认证用户并且「记住」他

假如你想要在应用中提供「记住我」的功能，你可以传入布尔值作为 `attempt` 方法的第二个参数，这样就可以保留用户的认证身份（或直到他手动登出为止）。当然，你的 `users` 数据表必需包括一个字符串类型的 `remember_token` 列來储存「记住我」的标识。

```
if (Auth::attempt(['email' => $email, 'password' => $password], $remember))
{
    // The user is being remembered...
}
```
假如有使用「记住我」功能，可以使用 `viaRemember`方法判定用户是否拥有「记住我」的 cookie 來判定用户认证：

```
if (Auth::viaRemember())
{
    //
}
```
### 以 ID 认证用户

要通过 ID 来认证用户，使用 `loginUsingId` 方法：

```
Auth::loginUsingId(1);
```
### 验证用户信息而不登陆

`validate` 方法可以让你验证用户凭证信息而不用真的登陆应用：

```
if (Auth::validate($credentials))
{
    //
}
```

### 在单一请求内登陆用户

你也可以使用 `once` 方法來让用户在单一请求内登陆。不会有任何 session 或 cookie 产生：

```
if (Auth::once($credentials))
{
    //
}
```

### 手动登陆用户

假如你需要将一个已经存在的用户实例登陆应用，你可以调用 `login` 方法并且传入用户实例：

```
Auth::login($user);
```
这个方式和使用 attempt方法验证用户凭证信息是一样的。

### 用户登出

```
Auth::logout();
```
当然，假设你使用 Laravel 內建的认证控制器，预设提供了让用户登出的方法。

### 认证事件

当 `attempt` 方法被调用时，`auth.attempt` 事件 会被触发。假设用户尝试认证成功并且登陆了，`auth.login` 事件会被触发。


## 取得经过认证的用户

当用户通过认证后，有几种方式取得用户实例。

首先， 你可以从 `Auth` facade 取得用户：

```
<?php namespace App\Http\Controllers;

use Illuminate\Routing\Controller;

class ProfileController extends Controller {

    /**
     * Update the user's profile.
     *
     * @return Response
     */
    public function updateProfile()
    {
        if (Auth::user())
        {
            // Auth::user() returns an instance of the authenticated user...
        }
    }

}
```

第二种，你可以使用 `Illuminate\Http\Request` 实例取得认证过的用户：

```
<?php namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Illuminate\Routing\Controller;

class ProfileController extends Controller {

    /**
     * Update the user's profile.
     *
     * @return Response
     */
    public function updateProfile(Request $request)
    {
        if ($request->user())
        {
            // $request->user() returns an instance of the authenticated user...
        }
    }

}
```

第三，你可以使用 `Illuminate\Contracts\Auth\Authenticatable contract` 类型提示。这个类型提示可以用在控制器的构造方法，控制器的其他方法，或是其他可以通过服务容器 解析的类的构造方法：

```
<?php namespace App\Http\Controllers;

use Illuminate\Routing\Controller;
use Illuminate\Contracts\Auth\Authenticatable;

class ProfileController extends Controller {

    /**
     * Update the user's profile.
     *
     * @return Response
     */
    public function updateProfile(Authenticatable $user)
    {
        // $user is an instance of the authenticated user...
    }

}
```

## 保护路由

路由中间件 只允许通过认证的用户访问指定的路由。Laravel 默认提供了 `auth` 中间件，放在 `app\Http\Middleware\Authenticate.php`。 你需要做的只是将其加到一个路由定义中：

```
// With A Route Closure...

Route::get('profile', ['middleware' => 'auth', function()
{
    // Only authenticated users may enter...
}]);

// With A Controller...

Route::get('profile', ['middleware' => 'auth', 'uses' => 'ProfileController@show']);
```

## HTTP 基本认证

HTTP 基本认证提供了一个快速的方式来认证用户而不用特定设置一个「登入」页。在您的路由内设定 `auth.basic` 中间件则可启动这个功能：

### 用 HTTP 基本认证保护路由

```
Route::get('profile', ['middleware' => 'auth.basic', function()
{
    // Only authenticated users may enter...
}]);
```

默认情况下 `basic` 中间件会使用用户的 `email` 列当做「 username 」。

### 设定无状态的 HTTP 基本过滤器

你可能想要使用 HTTP 基本认证，但不会在 session 里设置用户身份的 cookie，这在 API 认证时特別有用。如果要這樣做，定义一个中间件并调用 `onceBasic` 方法：

```
public function handle($request, Closure $next)
{
    return Auth::onceBasic() ?: $next($request);
}
```

如果你使用 PHP FastCGI，HTTP 基本认证可能无法正常运行。请在你的 `.htaccess` 文件内新增以下代码：

```
RewriteCond %{HTTP:Authorization} ^(.+)$
RewriteRule .* - [E=HTTP_AUTHORIZATION:%{HTTP:Authorization}]
```

## 忘记密码与重设

### 模型与数据表

大多数的 web 应用程序都会提供用户忘记密码的功能。为了不让开发者重复实现这个功能，Laravel 提供了方便的方法来发送忘记密码通知及密码重设的功能。

在开始之前，请先确认您的 `User` 模型实现了 `Illuminate\Contracts\Auth\CanResetPassword`接口。当然，默认 Laravel 的 User 模型本身就已实现，并且引入Illuminate\Auth\Passwords\CanResetPassword来包括所有需要实现的接口方法。

### 生成 Reminder 数据表迁移

接下来，我们需要生成一个数据库表来储存重设密码标志。Laravel 默认已经包含了这个迁移表，放在 `database/migrations` 的目录下。你所需要作的只有执行迁移：

```
php artisan migrate
```
### 密码重设控制器

Laravel 还包含了 `Auth\PasswordController` 其中包含重设用户密码的功能。甚至一些视图，可以让你直接开始使用！视图放在 `resources/views/auth` 目录下。你可以按照你的应用程序设计，自由的修改這些视图。

你的使用者会收到一封 e-mail，內含连接指向 `PasswordController` 中的 `getReset` 方法。这个方法会显示密码重设表单，允许用户重新设定密码。在密码重新设定完之后，用户将会自动登录到应用中，然后被重定向到 `/home`。你可以通过 `PasswordController` 中的 `redirectTo` 來定义重设密码后要重定向的位置：

```
protected $redirectTo = '/dashboard';
```
**`注意：默认情况下，密码重设 tokens 会在一小时后过期。你可以修改 config/auth.php 文件中的 reminder.expire 更改 这个设定。`**

## 第三方登陆认证

除了传统的以表单进行的认证，Laravel 还提供了简单、易用的方式，使用 Laravel Socialite 进行 OAuth 认证。Socialite 目前支持的认证有 Facebook、 Twitter、Google、以及GitHub 和 Bitbucket 。

如果要开始使用第三方认证，请将下面的代码加入到你的 `composer.json`文件內：

```
"laravel/socialite": "~2.0"
```
接下來，在你的 `config/app.php` 配置文件中注册 `Laravel\Socialite\SocialiteServiceProvider`。也可以注册 facade：

```
'Socialize' => 'Laravel\Socialite\Facades\Socialite',
```

你需要在应用程序中加入 OAuth 服务所需的凭证。这些凭证都放在 `config/services.php` 配置文件里，并根据应用的需求使用 `facebook`、`twitter`、`google` 或 `github` 作为对应的键值。例如：

```
'github' => [
    'client_id' => 'your-github-app-id',
    'client_secret' => 'your-github-app-secret',
    'redirect' => 'http://your-callback-url',
],
```

接下來就准备认证用户了！你会需要两个路由：一个用于将用户重定向至认证提供网站，另一个用于认证之后，从认证服务接收回调。下面是一个使用 `Socialize` facade 的示例：

```
public function redirectToProvider()
{
    return Socialize::with('github')->redirect();
}

public function handleProviderCallback()
{
    $user = Socialize::with('github')->user();

    // $user->token;
}
```

`redirect` 方法将用户重定向到认证 OAuth 的网站，而 `user` 方法会获取返回的请求，以及从认证网站取得的用户信息。在重定向至用户之前，你也可以设定请求的「 scopes 」：

```
return Socialize::with('github')->scopes(['scope1', 'scope2'])->redirect();
```
一旦你取得用户实例，你能获取到更多的用户详细信息：

### 获取用户资料

```
$user = Socialize::with('github')->user();

// OAuth Two Providers
$token = $user->token;

// OAuth One Providers
$token = $user->token;
$tokenSecret = $user->tokenSecret;

// All Providers
$user->getId();
$user->getNickname();
$user->getName();
$user->getEmail();
$user->getAvatar();
```