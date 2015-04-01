# 扩展包开发

## 介绍

开发扩展包是添加功能到 Laravel 最主要的方法。扩展包可以是任何处理日期的方式。例如，[Carbon](https://github.com/briannesbitt/Carbon)，或是一个全套的 BDD testing 框架。例如，[Behat](https://github.com/Behat/Behat)

当然，有非常多不同类型的扩展包。有些扩展包是独立的，意思是此扩展包运作且兼容于任何的框架，不只有 Laravel。Carbon 以及 Behat 都是这类的扩展包。任何这类的扩展包只需要在您的`composer.json` 文件里配置就可以使用。

另一方面，其他的扩展包所设计的目的是只要在 Laravel 上使用。这些扩展包可能包含路由、控制器、视图以及扩展包的相关配置，目的是为了增加 Laravel 的应用。接下来的说明主要涵盖了 Laravel 开发这些扩展包的重点。

所有 Laravel 扩展包都发布到[Packagist](http://packagist.org) 以及[Composer](http://getcomposer.org)，所以学习这些美好的 PHP 扩展包管理工具是必须的。

## 视图

您扩展包内部的架构全部由您自己规划。然而，原则上会有一个或更多的[服务提供者](../providers). 服务提供者包含着所有的[服务容器](../container) 绑定，也定义了所有您扩展包的相关配置、视图以及语言文件在什么地方。

### 视图

扩展包的视图基本上使用两个双冒号来指定:

```
return view('package::view.name');
```

所有您所要做的只有告诉 Laravel 您所配置扩展包名称视图的位置在哪里。如果您的扩展包取名为 “courier” 您可能需要添加如下到您的服务提供者的`boot` 方法:

```
public function boot()
{
    $this->loadViewsFrom(__DIR__.'/path/to/views', 'courier');
}
```

现在您可以使用如下的语法来加载扩展包的视图:

```
return view('courier::view.name');
```

当您使用`loadViewsFrom` 方法，Laravel 实际上为了您的视图注册了**两个位置**。一个是您应用程序的`resources/views/vendor` 目录，一个是您指定的目录。所以使用我们的例子`courier` 当要求一个扩展包的视图时，Laravel 会第一时间检查是否有一个开发者自行自定义在`resources/views/vendor/courier` 的视图存在。然而如果还没有这个路径的视图被自定义。Laravel 会搜索您在扩展包`loadViewsFrom` 方法里所指定的视图。这个方法让个别的用户可以方便的自定义且覆写您在扩展包里的视图。

#### 视图的发布

发布扩展包的视图到`resources/views/vendor` 目录，您必须在服务提供者里的`boot` 方法里使用`publishes` 方法:

```
public function boot()
{
    $this->loadViewsFrom(__DIR__.'/path/to/views', 'courier');

    $this->publishes([
        __DIR__.'/path/to/views' => base_path('resources/views/vendor/courier'),
    ]);
}
```

现在当您扩展包的用户使用 Laravel 的命令`vendor:publish` 您的视图目录将会被复制到所特定的目录

如果您想要覆写已存在的文件，可以使用`--force`:

```
php artisan vendor:publish --force
```

> **注意:** 您可以使用`publishes` 方法，发布任何您的文件到**任何**您想要的地方。

## 语言

扩展包的语言文件基本上使用两个双冒号来指定:

```
return trans('package::file.line');
```

所有您所要做的只有告诉 Laravel 您所配置扩展包名称的语言位置在哪里。如果您的扩展包取名为 "courier" 您可能需要添加如下的语法到您的服务提供者的`boot` 方法:

```
public function boot()
{
    $this->loadTranslationsFrom(__DIR__.'/path/to/translations', 'courier');
}
```

注意在您的`translations` 目录里，必须要有更下一层的目录，例如`en``es``ru`。

现在您可以使用下方的语法来加载您扩展包的语言:

```
return trans('courier::file.line');
```

## 配置文件

基本上，您可能想要将您扩展包相关配置的文件发布到应用程序本身的配置目录`config`。这将允许您扩展包的用户简单的覆写这些默认的配置文件。

发布扩展包的配置文件只需要在服务提供者里的`boot` 方法里使用`publishes` 方法:

```
$this->publishes([
    __DIR__.'/path/to/config/courier.php' => config_path('courier.php'),
]);
```

现在当扩展包的用户执行`vendor:publish` 命令，您的文件将会被复制到特定的位置。当然只要配置文件已经被发布，就可以如其他配置文件一样被访问:

```
$value = config('courier.option');
```

您可能也选择想要合并您扩展包的配置文件和应用程序里的副本配置文件。这允许您的用户在已经被发布的副本配置文件里只包含任何他们想要覆写的配置选项。如果想要合并配置文件，可在服务提供者里的`register` 方法里使用`mergeConfigFrom`方法

```
$this->mergeConfigFrom(
    __DIR__.'/path/to/config/courier.php', 'courier'
);
```

## 公共资源

您的扩展包也许会有一些资源文件比如 JavaScript，CSS，和图片。如果要发布资源，可以在您的服务提供商的`boot` 方法中使用`publishes` 方法。在这个例子中，我们同样增加了 “public” 资源的分区标记。

```
$this->publishes([
    __DIR__.'/path/to/assets' => public_path('vendor/courier'),
], 'public');
```

现在当扩展包的用户执行`vendor:publish` 命令，您的文件将会被复制到特定的位置。由于每次扩展更新时都会覆盖这些资源，你可以使用`--force` 标识：

```
php artisan vendor:publish --tag=public --force
```

如果你想确保你的公共资源始终是最新的，你可以把这条命名添加到`composer.json` 文件中的`post-update-cmd` 列表中。

## 发布分类文件

您可能想要分别的发布一些分类的文件。举例，您可能想要您的用户可以分别发布扩展包的配置文件与静态资源文件。您可以使用`tagging` 来达成:

```
// Publish a config file
$this->publishes([
    __DIR__.'/../config/package.php' => config_path('package.php')
], 'config');

// Publish your migrations
$this->publishes([
    __DIR__.'/../database/migrations/' => base_path('/database/migrations')
], 'migrations');
```

您可以使用这些`tag`，来分别发布这些扩展包里的文件。

```
php artisan vendor:publish --provider="Vendor\Providers\PackageServiceProvider" --tag="config"
```

## 路由

在扩展包里加载一个路由文件，只需要在服务提供者里的`boot` 方法里使用`include` :

#### 根据服务提供者来包含一个路由文件

```
public function boot()
{
    include __DIR__.'/../../routes.php';
}
```

> **注意:** 如果您的扩展包里使用了控制器，您必须要确认您在`composer.json` 文件里的 auto-load 区块里，是否适当的配置这些控制器。