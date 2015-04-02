# Session

## 配置

由于 HTTP 协定是无状态（Stateless）的，所以 session 提供一种保存用户数据的方法。Laravel 支持了多种 session 后端驱动，并通过清楚、统一的 API 提供使用。也内置支持如[Memcached](http://memcached.org)、[Redis](http://redis.io) 和数据库的后端驱动。

session 的配置文件配置在`config/session.php` 中，请务必看一下 session 配置文件中可用的选项配置及注释。Laravel 默认使用`file` 的 session 驱动，它在大多的应用中可以良好运作。

如果你想在 Laravel 中使用`Redis` sessions，你需要先通过 Composer 安装`predis/predis` 扩展包 (~1.0)。

> **注意：** 如果你需要加密所有的 session 数据，就将选项`encrypt` 配置为`true` 。

#### 保留键值

Laravel 框架在内部有使用`flash` 作为 session 的键值，所以应该避免 session 使用此名称。

## 使用 Session

获取 session 有很多种方式，可以通过 HTTP request 类的`session` 方法，`Session` facade 或者`session` 辅助函数。如果在调用`session` 辅助函数时没有传入参数，会返回整个 session 对象。比如：

```
session()->regenerate();
```

#### 保存对象到 Session 中

```
Session::put('key', 'value');

session(['key' => 'value']);
```

#### 保存对象进 Session 数组值中

```
Session::push('user.teams', 'developers');
```

#### 从 Session 取回对象

```
$value = Session::get('key');

$value = session('key');
```

#### 从 Session 取回对象，若无则返回默认值

```
$value = Session::get('key', 'default');

$value = Session::get('key', function() { return 'default'; });
```

#### 从 Session 取回对象，并删除

```
$value = Session::pull('key', 'default');
```

#### 从 Session 取出所有对象

```
$data = Session::all();
```

#### 判断对象在 Session 中是否存在

```
if (Session::has('users'))
{
    //
}
```

#### 从 Session 中移除对象

```
Session::forget('key');
```

#### 清空所有 Session

```
Session::flush();
```

#### 重新产生 Session ID

```
Session::regenerate();
```

## 暂存数据（Flash Data）

有时你可能希望暂存一些数据，并只在下次请求有效。你可以使用`Session::flash` 方法来达成目的：

```
Session::flash('key', 'value');
```

#### 刷新当前暂存数据，延长到下次请求

```
Session::reflash();
```

#### 只刷新指定快闪数据

```
Session::keep(array('username', 'email'));
```

## 数据库 Sessions

当使用`database` session 驱动时，你必需建置一张保存 session 的数据表。下方例子使用`Schema` 来建表：

```
Schema::create('sessions', function($table)
{
    $table->string('id')->unique();
    $table->text('payload');
    $table->integer('last_activity');
});
```

当然你也可以使用 Artisan 命令`session:table` 来建 migration 表：

```
php artisan session:table

composer dump-autoload

php artisan migrate
```

## Session 驱动

session 配置文件中的「driver」定义了 session 数据将以哪种方式被保存。Laravel 提供了许多良好的驱动：

*   `file` - sessions 将保存在`storage/framework/sessions`。
*   `cookie` - sessions 将安全保存在加密的 cookies 中。
*   `database` - sessions 将保存在你的应用程序数据库中。
*   `memcached` /`redis` - sessions 将保存在一个高速缓存的系统中。
*   `array` - sessions 将单纯的以 PHP 数组保存，只存活在当次请求。

> **注意：** array 驱动典型应用在[unit tests](testing.md) 环境下，所以不会留下任何 session 数据。