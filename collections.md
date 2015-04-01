## 集合

介绍
基本用法

# 介绍

`Illuminate\Support\Collection` 类提供一个流畅、方便的封装来操作数组数据。举个例子，查看下面的代码。我们将会使用 `collect` 辅助方法来用数组建立一个新的集合实例：

```
$collection = collect(['taylor', 'abigail', null])->map(function($name)
{
    return strtoupper($name);
})
->reject(function($name)
{
    return empty($name);
});
```

可以看到，`Collection` 类允许你链式调用它的方法对背后的数组执行流畅的映射和归纳。一般说来，每一个 `Collection` 的方法都返回一个全新的 `Collection` 实例。为了更深一步的了解，请继续阅读！


## 基本用法

### 建立集合

如上述，`collect` 辅助方法将会用给定的数组返回一个新的 `Illuminate\Support\Collection` 实例。你也可以在 `Collection` 类上使用 `make` 命令：

```
$collection = collect([1, 2, 3]);

$collection = Collection::make([1, 2, 3]);
```

当然，Eloquent 的对象集合总是以 `Collection` 实例返回；然而，你可以在应用程序的任何地方方便的使用 `Collection` 类。

### 探索集合

作为列出集合可以用的所有方法 (有很多) 的替代，请查看 类的 API 文档！