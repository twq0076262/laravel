# 基本用法

## 配置

Laravel 让连接数据库和执行查找变得相当容易。数据库相关配置文件都在 `config/database.php`。 在这个文件你可以定义所有的数据库连接，以及指定默认的数据库连接。默认文件中已经有所有支持的数据库系统例子了。

目前 Laravel 支持四种数据库系统： MySQL、Postgres、SQLite、以及 SQL Server。

## 读取/写入连接

有时候你可能希望使用特定数据库连接进行 SELECT 操作，同时使用另外的连接进行 INSERT 、 UPDATE 、以及 DELETE 操作。 Laravel 让这些变得轻松简单，并确保你不论在使用原始查找、查找构建器、或者是 Eloquent ORM 使用的都是正确的连接。

来看看如何配置读取/写入连接，让我们来看以下的例子：

```
'mysql' => [
    'read' => [
        'host' => '192.168.1.1',
    ],
    'write' => [
        'host' => '196.168.1.2'
    ],
    'driver'    => 'mysql',
    'database'  => 'database',
    'username'  => 'root',
    'password'  => '',
    'charset'   => 'utf8',
    'collation' => 'utf8_unicode_ci',
    'prefix'    => '',
],
```

注意我们加了两个键值到配置文件数组中： `read` 及 `write`。 两个键值都包含了单一键值的数组：`host`。`read` 及 `write` 的其余数据库配置会从 `mysql` 数组中合并。 所以，如果我们想要覆写配置值，只要将选项放入 `read` 和 `write` 数组即可。 所以在上面的例子里， `192.168.1.1` 将被用作「读取」连接，而 `192.168.1.2` 将被用作「写入」连接。数据库凭证、 前缀、字符编码配置、以及其他所有的配置会共用 `mysql` 数组里的配置。

## 执行查找

如果配置好数据库连接，就可以通过 `DB` facade 执行查找。

### 执行 Select 查找

```
$results = DB::select('select * from users where id = ?', [1]);
```

`select` 方法会返回一个 `array` 结果。

### 执行 Insert 语法

```
DB::insert('insert into users (id, name) values (?, ?)', [1, 'Dayle']);
```

### 执行 Update 语法

```
DB::update('update users set votes = 100 where name = ?', ['John']);
```

### 执行 Delete 语法

```
DB::delete('delete from users');
```

>
注意： update 和 delete 语法会返回在操作中所影响的数据笔数。

### 执行一般语法

DB::statement('drop table users');

### 监听查找事件

你可以使用 `DB::listen` 方法，去监听查找的事件：

```
DB::listen(function($sql, $bindings, $time)
{
    //
});
```

## 数据库事务处理

你可以使用 `transaction` 方法，去执行一组数据库事务处理的操作：

```
DB::transaction(function()
{
    DB::table('users')->update(['votes' => 1]);

    DB::table('posts')->delete();
});
```

>
注意：在 `transaction` 闭包若抛出任何异常会导致事务自动回滚。

有时候你可能需要自己开始一个事务：

```
DB::beginTransaction();
```

你可以通过 `rollback` 的方法回滚事务：

```
DB::rollback();
```

最后，你可以通过 `commit` 的方法提交事务：

```
DB::commit();
```

## 获取连接

若要使用多个连接，可以通过 `DB::connection` 方法取用：

```
$users = DB::connection('foo')->select(...);
```

你也可以取用原始底层的 PDO 实例：

```
$pdo = DB::connection()->getPdo();
```

有时候你可能需要重新连接到特定的数据库：

```
DB::reconnect('foo');
```

如果你因为超过了底层 PDO 实例的 `max_connections` 的限制，需要关闭特定的数据库连接，可以通过 `disconnect` 方法:

```
DB::disconnect('foo');
```

## 查找日志纪录

Laravel 可以在内存里访问这次请求中所有的查找语句。然而在有些例子下要注意，比如一次添加 大量的数据，可能会导致应用程序耗损过多内存。 如果要启用日志，可以使用 `enableQueryLog` 方法：

```
DB::connection()->enableQueryLog();
```

要得到执行过的查找纪录数组，你可以使用 `getQueryLog` 方法：

```
$queries = DB::getQueryLog();
```