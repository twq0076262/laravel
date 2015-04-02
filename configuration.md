# 配置

## 简介

所有 Laravel 框架的配置文件都放置在 `config` 目录下。 每个选项都有说明，因此你可以轻松地浏览这些文档，并且熟悉这些选项配置。


## 完成安装后

### 命名你的应用程序

在安装 Laravel 后，你可以「命名」你的应用程序。默认情况下，`app` 的目录是在 `App` 的命名空间 下，通过 Composer 使用 [PSR-4 自动载入规范](https://github.com/PizzaLiu/PHP-FIG/blob/master/PSR-4-autoloader-cn.md) 自动加载。不过，你可以轻松地通过 Artisan 命令` app:name `来修改命名空间，以配合你的应用程序名称。

举例来说，假设你的应用程序叫做「 Horsefly 」，你可以从安装的根目录执行下面的命令：

```
php artisan app:name Horsefly
```
重命名你的应用程序是完全可选的，你也可以保留原有的命名空间 `App` 。

### 其他配置

Laravel 几乎不需配置就可以马上使用。你可以自由的开始开发！然而，你可以浏览 `config/app.php` 文件和其他的文档。你可能希望依据你的本机而做更改，文件包含数个选项如`时区`和`语言环境`。

一旦 Laravel 安装完成，你应该同时 [配置本机环境](configuration.md)。

```
注意： 你不应该在正式环境中将 app.debug 配置为 true 。绝对！千万不要！
```

### 权限

Laravel 框架有一个目录需要额外权限：storage 目录必须让服务器有写入权限。


## 取得配置值

你可以很轻松的使用 `Config` facade 取得你的配置值：

```
$value = Config::get('app.timezone');

Config::set('app.timezone', 'America/Chicago');
```

你也可以使用 `config` 辅助方法：

```
$value = config('app.timezone');
```

## 环境配置

通常应用程序常常需要根据不同的执行环境而有不同的配置值。例如，你会希望在你的本机开发环境上会有与正式环境不同的缓存驱动（cache driver），通过配置文件，就可以轻松完成。

Laravel 通过 [DotEnv](https://github.com/vlucas/phpdotenv) Vance Lucas 写的一个 PHP 类库。 在全新安装好的 Laravel 里，你的应用程序的根目录下会包含一个 `.env.example` 文件。如果你通过 Composer 安装 Laravel，这个文件将自动被命名为 `.env`，不然你应该手动更改文件名。

当你的应用程序收到请求，这个文件所有的变量会被加载到 `$_ENV` 这个 PHP 超级全局变量里。你可以使用辅助方法 `env` 查看这些变量。事实上，如果你查看过 Laravel 配置文件，你会注意到几个选项已经在使用这个辅助方法！

根据你的本机服务器或者线上环境需求，你可以自由的修改你的环境变量。然而， 你的 `.env` 文件不应该被提交到应用程序的版本控制系统，因为每个开发人员或服务器使用你的应用程序可能需要不同的环境配置。

如果你是一个团队的开发者，不妨将 `.env.example` 文件包含到你的应用程序。通过例子配置文件里的预留值，你的团队中其他开发人员可以清楚地看到执行你的应用程序所需的哪些环境变量。

#### 取得目前应用程序的环境

你可以通过 `Application` 实例中的 `environment` 方法取得目前应用程序的环境：

```
$environment = $app->environment();
```
你也可以传递参数至 `environment `方法中，来确认目前的环境是否与参数相符合：

```
if ($app->environment('local'))
{
    // The environment is local
}

if ($app->environment('local', 'staging'))
{
    // The environment is either local OR staging...
}
```

如果想取得应用程序的实例，可以通过[服务容器](container.md)的 `Illuminate\Contracts\Foundation\Application contract` 来取得。当然，如果你想在[服务提供者](providers.md)中使用，应用程序实例可以通过实例变量 `$this->app` 取得。

也能通过 `App` facade 或者辅助方法 `app` 取得应用程序实例：

```
$environment = app()->environment();

$environment = App::environment();
```

## 配置缓存

为了让你的的应用程序提升一些速度，你可以使用 Artisan 命令 `config:cache` 将所有的配置文件缓存到单一文件。通过命令会将所有的配置选项合并成一个文件，让框架能够快速加载。

通常来说，你应该将执行 `config:cache` 命令作为部署工作的一部分。


## 维护模式

当你的应用程序处于维护模式时，所有的路由都会指向一个自定的视图。当你要更新或维护网站时，「关闭」整个网站是很简单的。维护模式会检查包含在应用程序的默认中间件堆栈。如果应用程序处于维护模式，`HttpException` 会抛出 503 的状态码。

启用维护模式，只需要执行 Artisan 命令 `down`：

```
php artisan down
```
关闭维护模式，请使用 Artisan 命令 `up`：

```
php artisan up
```
### 维护模式的响应模板

维护模式响应的默认模板放在 `resources/views/errors/503.blade.php`。

### 维护模式与队列

当应用程序处于维护模式中，将不会处理任何[队列](queues.md)工作。所有的队列工作将会在应用程序离开维护模式后继续被进行。


## 优雅链接

### Apache

Laravel 框架通过` public/.htaccess` 文件来让网址中不需要 `index.php`。如果你的服务器是使用 Apache ，请确认是否有开启 `mod_rewrite` 模块。

假设 Laravel 附带的 `.htaccess` 文件在 Apache 无法生效的话，请尝试下面的方法：

```
Options +FollowSymLinks
RewriteEngine On

RewriteCond %{REQUEST_FILENAME} !-d
RewriteCond %{REQUEST_FILENAME} !-f
RewriteRule ^ index.php [L]
```
### Nginx

若使用 Nginx ，可以在你的网站配置中增加下面的配置，以开启「优雅链接」：

```
location / {
    try_files $uri $uri/ /index.php?$query_string;
}
```
当然，如果你使用 [Homestead](homestead.md) 的话，优雅链接会自动的帮你配置完成。