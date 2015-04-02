# 缓存

配置
缓存用法
递增与递减
缓存标签
缓存事件
数据库缓存

## 配置

Laravel 为各种不同的缓存系统提供一致的 API 。缓存配置文件位在 `config/cache.php` 。您可以在此为应用程序指定使用哪一种缓存系统， Laravel 支持各种常见的后端缓存系统，如 [Memcached](http://memcached.org/) 和 [Redis](http://redis.io/) 。

缓存配置文件也包含多个其他选项，在文件里都有说明，所以请务必先阅读过。 Laravel 默认使用`文件` 缓存系统，该系统会保存序列化、缓存对象在文件系统中。在大型应用程序上，建议使用保存在内存内的缓存系统，如 Memcached 或 APC 。你甚至可以以同一个缓存系统配置多个缓存配置。

在 Laravel 中使用 Redis 缓存系统前， 必须先使用 Composer 安装 `predis/predis` 扩展包 (~1.0) 。


## 缓存用法

### 保存对象到缓存中

```
Cache::put('key', 'value', $minutes);
```

### 使用 Carbon 对象配置缓存过期时间

```
$expiresAt = Carbon::now()->addMinutes(10);

Cache::put('key', 'value', $expiresAt);
```
### 若是对象不存在，则将其存入缓存中

```
Cache::add('key', 'value', $minutes);
```
当对象确实被加入缓存时，使用 `add` 方法将会返回 `true` 否则会返回 `false` 。

### 确认对象是否存在

```
if (Cache::has('key'))
{
    //
}
```
### 从缓存中取得对象

```
$value = Cache::get('key');
```

### 取得对象或是返回默认值

```
$value = Cache::get('key', 'default');

$value = Cache::get('key', function() { return 'default'; });
```
### 永久保存对象到缓存中

```
Cache::forever('key', 'value');
```

有时候您会希望从缓存中取得对象，而当此对象不存在时会保存一个默认值，您可以使用 `Cache::remember` 方法：

```
$value = Cache::remember('users', $minutes, function()
{
    return DB::table('users')->get();
});
```

您也可以结合 `remember` 和 `forever` 方法：

```
$value = Cache::rememberForever('users', function()
{
    return DB::table('users')->get();
});
```

请注意所有保存在缓存中的对象皆会被序列化，所以您可以任意保存各种类型的数据。

### 从缓存拉出对象

如果您需要从缓存中取得对象后将它删除，您可以使用 `pull` 方法：

```
$value = Cache::pull('key');
```
### 从缓存中删除对象

```
Cache::forget('key');
```
### 获取特定的缓存存储

当使用多种缓存存储时，你可以通过 `store` 方法来访问它们：

```
$value = Cache::store('foo')->get('key');
```

## 递增与递减

除了`文件`与`数据库`以外的缓存系统都支持`递增`和`递减`操作：

### 递增值

```
Cache::increment('key');

Cache::increment('key', $amount);
```

### 递减值

```
Cache::decrement('key');

Cache::decrement('key', $amount);
```

## 缓存标签

**`注意： 文件或数据库这类缓存系统均不支持缓存标签。此外，使用带有「forever」的缓存标签时，挑选 memcached 这类缓存系统将获得最好的性能，它会自动清除过期的纪录。`**

### 访问缓存标签

缓存标签允许您标记缓存内的相关对象，然后使用特定名称更新所有缓存标签。要访问缓存标签可以使用 `tags` 方法。

您可以保存缓存标签，通过将有序标签列表当作参数传入，或者作为标签名称的有序数组：

```
Cache::tags('people', 'authors')->put('John', $john, $minutes);

Cache::tags(array('people', 'artists'))->put('Anne', $anne, $minutes);
```

您可以结合使用各种缓存保存方法与标签，包含 `remember`, `forever`, 和 `rememberForever` 。您也可以从已标记的缓存中访问对象，以及使用其他缓存方法如 `increment` 和 `decrement` 。

### 从已标记的缓存中访问对象

要访问已标记的缓存，可传入相同的有序标签列表。

```
$anne = Cache::tags('people', 'artists')->get('Anne');

$john = Cache::tags(array('people', 'authors'))->get('John');
```

您可以更新所有已标记的对象，使用指定名称或名称列表。例如，以下例子将会移除带有 `people` 或 `authors` 或者两者皆有的所有缓存标签，所以「Anne」和「John」皆会从缓存中被移除:

```
Cache::tags('people', 'authors')->flush();
```

对照来看，以下例子将只会移除带有 `authors` 的标签，所以「John」会被移除，但是「Anne」不会。

```
Cache::tags('authors')->flush();
```

## 缓存事件

你可以通过监听缓存操作时对应的事件来执行特定的代码：

```
Event::listen('cache.hit', function($key, $value) {
    //
});

Event::listen('cache.missed', function($key) {
    //
});

Event::listen('cache.write', function($key, $value, $minutes) {
    //
});

Event::listen('cache.delete', function($key) {
    //
});

```
## 数据库缓存

当使用`数据库`缓存系统时，您必须配置一张数据表来保存缓存对象。数据表的 `Schema` 声明例子如下：

```
Schema::create('cache', function($table)
{
    $table->string('key')->unique();
    $table->text('value');
    $table->integer('expiration');
});
```