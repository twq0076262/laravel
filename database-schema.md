# 结构生成器

## 介绍

Laravel 的结构生成器 (`Schema`) 提供一个与数据库无关的数据表产生方法，它可以很好的处理 Laravel 支持的各种数据库类型，并且在不同系统间提供一致性的 API 操作。

## 建立与删除数据表

要建立一个新的数据表，可使用`Schema::create` 方法：

```
Schema::create('users', function($table)
{
    $table->increments('id');
});
```


传入 `create` 方法的第一个参数是数据表名称，第二个参数是 `Closure` 并接收`Blueprint` 对象被用来定义新的数据表。

要修改数据表名称，可使用 `rename` 方法：

```
Schema::rename($from, $to);
```


要指定特定连接来操作，可使用 `Schema::connection` 方法：

```
Schema::connection('foo')->create('users', function($table)
{
    $table->increments('id');
});
```


要移除数据表，可使用 `Schema::drop` 方法：

```
Schema::drop('users');

Schema::dropIfExists('users');
```

## 加入字段

更新现有的数据表，可使用 `Schema::table` 方法：

```
Schema::table('users', function($table)
{
    $table->string('email');
});
```


数据表产生器提供多种字段类型可使用，在您建立数据表时也许会用到：

| 命令                                              | 功能描述                                     |
| ----------------------------------------------- | ---------------------------------------- |
| `$table->bigIncrements('id');`                  | ID 自动增量，使用相当于「big integer」类型             |
| `$table->bigInteger('votes');`                  | 相当于 BIGINT 类型                            |
| `$table->binary('data');`                       | 相当于 BLOB 类型                              |
| `$table->boolean('confirmed');`                 | 相当于 BOOLEAN 类型                           |
| `$table->char('name', 4);`                      | 相当于 CHAR 类型，并带有长度                        |
| `$table->date('created_at');`                   | 相当于 DATE 类型                              |
| `$table->dateTime('created_at');`               | 相当于 DATETIME 类型                          |
| `$table->decimal('amount', 5, 2);`              | 相当于 DECIMAL 类型，并带有精度与基数                  |
| `$table->double('column', 15, 8);`              | 相当于 DOUBLE 类型，总共有 15 位数，在小数点后面有 8 位数     |
| `$table->enum('choices', array('foo', 'bar'));` | 相当于 ENUM 类型                              |
| `$table->float('amount');`                      | 相当于 FLOAT 类型                             |
| `$table->increments('id');`                     | 相当于 Incrementing 类型 (数据表主键)              |
| `$table->integer('votes');`                     | 相当于 INTEGER 类型                           |
| `$table->json('options');`                      | 相当于 JSON 类型                              |
| `$table->longText('description');`              | 相当于 LONGTEXT 类型                          |
| `$table->mediumInteger('numbers');`             | 相当于 MEDIUMINT 类型                         |
| `$table->mediumText('description');`            | 相当于 MEDIUMTEXT 类型                        |
| `$table->morphs('taggable');`                   | 加入整数 `taggable_id` 与字串 `taggable_type`   |
| `$table->nullableTimestamps();`                 | 与 `timestamps()` 相同，但允许 NULL             |
| `$table->smallInteger('votes');`                | 相当于 SMALLINT 类型                          |
| `$table->tinyInteger('numbers');`               | 相当于 TINYINT 类型                           |
| `$table->softDeletes();`                        | 加入 **deleted_at** 字段于软删除使用               |
| `$table->string('email');`                      | 相当于 VARCHAR 类型                           |
| `$table->string('name', 100);`                  | 相当于 VARCHAR 类型，并指定长度                     |
| `$table->text('description');`                  | 相当于 TEXT 类型                              |
| `$table->time('sunrise');`                      | 相当于 TIME 类型                              |
| `$table->timestamp('added_on');`                | 相当于 TIMESTAMP 类型                         |
| `$table->timestamps();`                         | 加入 **created_at** 和 **updated_at** 字段    |
| `$table->rememberToken();`                      | 加入 `remember_token` 使用 VARCHAR(100) NULL |
| `->nullable()`                                  | 标示此字段允许 NULL                             |
| `->default($value)`                             | 声明此字段的默认值                                |
| `->unsigned()`                                  | 配置整数是无分正负                                |


#### 在 MySQL 使用 After 方法

若您使用 MySQL 数据库，您可以使用 `after` 方法来指定字段的顺序：

```
$table->string('name')->after('email');
```

## 修改字段

有时候您需要修改一个存在的字段，例如：您可能想增加保存文本字段的长度。通过 `change` 方法让这件事情变得非常容易！假设我们想要将字段 `name` 的长度从 25 增加到 50 的时候：

```
Schema::table('users', function($table)
{
    $table->string('name', 50)->change();
});
```


另外也能将某个字段修改为允许 NULL：

```
Schema::table('users', function($table)
{
    $table->string('name', 50)->nullable()->change();
});
```

## 修改字段名称

要修改字段名称，可在结构生成器内使用 `renameColumn` 方法，请确认在修改前`composer.json` 文件内已经加入 `doctrine/dbal`。

```
Schema::table('users', function($table)
{
    $table->renameColumn('from', 'to');
});
```

> **注意:** 目前暂时还不支持修改表中的结构为 `enum` 字段类型。

## 移除字段

要移除字段，可在结构生成器内使用 `dropColumn` 方法，请确认在移除前`composer.json` 文件内已经加入 `doctrine/dbal`。

#### 移除数据表字段

```
Schema::table('users', function($table)
{
    $table->dropColumn('votes');
});
```

#### 移除数据表多个字段

```
Schema::table('users', function($table)
{
    $table->dropColumn(array('votes', 'avatar', 'location'));
});
```

## 检查是否存在

#### 检查数据表是否存在

您可以轻松的检查数据表或字段是否存在，使用 `hasTable` 和 `hasColumn` 方法：

```
if (Schema::hasTable('users'))
{
    //
}
```

#### 检查字段是否存在

```
if (Schema::hasColumn('users', 'email'))
{
    //
}
```

## 加入索引

结构生成器支持多种索引类型，有两种方法可以加入，方法一，您可以在定义字段时顺便附加上去，或者是分开另外加入：

```
$table->string('email')->unique();
```


或者，您可以独立一行来加入索引，以下是支持的索引类型：

| 命令                                         | 功能描述                   |
| ------------------------------------------ | ---------------------- |
| `$table->primary('id');`                   | 加入主键 (primary key)     |
| `$table->primary(array('first', 'last'));` | 加入复合键 (composite keys) |
| `$table->unique('email');`                 | 加入唯一索引 (unique index)  |
| `$table->index('state');`                  | 加入基本索引 (index)         |

## 外键

Laravel 也支持数据表的外键约束：

```
$table->integer('user_id')->unsigned();
$table->foreign('user_id')->references('id')->on('users');
```


例子中，我们关注字段 `user_id` 参照到 `users` 数据表的 `id` 字段。请先确认已经建立外键！

您也可以指定选择在「on delete」和「on update」进行约束动作：

```
$table->foreign('user_id')
      ->references('id')->on('users')
      ->onDelete('cascade');
```


要移除外键，可使用 `dropForeign` 方法。外键的命名约定如同其他索引：

```
$table->dropForeign('posts_user_id_foreign');
```

> **注意:** 当外键有参照到自动增量时，记得配置外键为 `unsigned` 类型。

## 移除索引

要移除索引您必须指定索引名称，Laravel 默认有脉络可循的索引名称。简单地链接这些数据表与索引的字段名称和类型。举例如下：

| 命令                                          | 功能描述              |
| ------------------------------------------- | ----------------- |
| `$table->dropPrimary('users_id_primary');`  | 从「users」数据表移除主键   |
| `$table->dropUnique('users_email_unique');` | 从「users」数据表移除唯一索引 |
| `$table->dropIndex('geo_state_index');`     | 从「geo」数据表移除基本索引   |

## 移除时间戳记和软删除

要移除 `timestamps`、`nullableTimestamps` 或 `softDeletes` 字段类型，您可以使用以下方法：

| 命令                           | 功能描述                                  |
| ---------------------------- | ------------------------------------- |
| `$table->dropTimestamps();`  | 移除 **created_at** 和 **updated_at** 字段 |
| `$table->dropSoftDeletes();` | 移除 **deleted_at** 字段                  |


## 保存引擎

要配置数据表的保存引擎，可在结构生成器配置 `engine` 属性：

```
Schema::create('users', function($table)
{
    $table->engine = 'InnoDB';

    $table->string('email');
});
```
