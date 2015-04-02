# Eloquent ORM

## 介绍

Laravel 的 Eloquent ORM 提供了漂亮、简洁的 ActiveRecord 实现来和数据库的互动。 每个数据库表会和一个对应的「模型」互动。

在开始之前，记得把`config/database.php` 里的数据库连接配置好。

## 基本用法

我们先从建立一个 Eloquent 模型开始。模型通常放在`app` 目录下，但是您可以将它们放在任何地方，只要能通过 composer.json 自动载入。所有的 Eloquent 模型都继承于`Illuminate\Database\Eloquent\Model` 。

#### 定义一个 Eloquent 模型

```
	class User extends Model {}
```

你也可以通过 `make:model` 命令自动生成 Eloquent 模型：

```
	php artisan make:model User
```

注意我们并没有告诉 Eloquent User 模型会使用哪个数据库表。若没有特别指定，系统会默认自动对应名称为「类名称的小写复数形态」的数据库表。所以，在上面的例子中， Eloquent 会假设`User` 模型将把数据存在`users` 数据库表。您也可以在类中定义`table` 属性自定义要对应的数据库表。

```
    class User extends Model {

        protected $table = 'my_users';

    }
```

> **注意：** Eloquent 也会假设每个数据库表都有一个字段名称为`id` 的主键。您可以在类里定义`primaryKey` 属性来重写。同样的，您也可以定义`connection` 属性，指定模型连接到指定的数据库连接。

定义好模型之后，您就可以从数据库表新增及获取数据了。注意在默认情况下，在数据库表里需要有`updated_at` 和`created_at` 两个字段。如果您不想设定或自动更新这两个字段，则将类里的`$timestamps` 属性设为 false即可。

#### 取出所有模型数据

```
    $users = User::all();
```

#### 根据主键取出一条数据

```
    $user = User::find(1);

    var_dump($user->name);
```

> **提示：** 所有[查询构造器](queries.md)里的方法，查询 Eloquent 模型时也可以使用。

#### 根据主键取出一条数据或抛出异常

有时, 您可能想要在找不到模型数据时抛出异常，通过`App::error` 捕捉异常处理并显示 404 页面。

```
    $model = User::findOrFail(1);

    $model = User::where('votes', '>', 100)->firstOrFail();
```

要注册错误处理，可以监听`ModelNotFoundException`

```
    use Illuminate\Database\Eloquent\ModelNotFoundException;

    App::error(function(ModelNotFoundException $e)
    {
        return Response::make('Not Found', 404);
    });
```

#### Eloquent 模型结合查询语法

```
    $users = User::where('votes', '>', 100)->take(10)->get();

    foreach ($users as $user)
    {
        var_dump($user->name);
    }
```

#### Eloquent 聚合查询

当然，您也可以使用查询构造器的聚合查询方法。

```
    $count = User::where('votes', '>', 100)->count();
```

如果没办法使用流畅接口产生出查询语句，也可以使用`whereRaw` 方法：

```
    $users = User::whereRaw('age > ? and votes = 100', array(25))->get();
```

#### 拆分查询

如果您要处理非常多（数千条）Eloquent 查询结果，使用`chunk` 方法可以让您顺利工作而不会消耗大量内存：

```
    User::chunk(200, function($users)
    {
        foreach ($users as $user)
        {
            //
        }
    });
```

传到方法里的第一个参数表示每次「拆分」要取出的数据数量。第二个参数的闭合函数会在每次取出数据时被调用。

#### 指定查询时连接数据库

您也可以指定在执行 Eloquent 查询时要使用哪个数据库连接。只要使用`on` 方法：

```
    $user = User::on('connection-name')->find(1);
```

如果您在使用[读取 / 写入连接](database.md), 您可以通过如下命令来强制查询使用`写入` 连接：

```
    $user = User::onWriteConnection()->find(1);
```

## 批量赋值

在建立一个新的模型时，您把属性以数组的方式传入模型的构造方法，这些属性值会经由批量赋值存成模型数据。这一点非常方便，然而，若盲目地将用户输入存到模型时，可能会造成**严重的**安全隐患。如果盲目的存入用户输入，用户可以随意的修改**任何**以及**所有**模型的属性。基于这个理由，所有的 Eloquent 模型默认会阻止批量赋值 。

我们以在模型里设定`fillable` 或`guarded` 属性作为开始。

#### 定义模型`Fillable` 属性

`fillable` 属性指定了哪些字段支持批量赋值 。可以设定在类的属性里或是实例化后设定。

```
    class User extends Model {

        protected $fillable = array('first_name', 'last_name', 'email');

    }
```

在上面的例子里，只有三个属性允许批量赋值。

#### 定义模型`Guarded` 属性


`guarded` 与`fillable` 相反，是作为「黑名单」而不是「白名单」：

```
    class User extends Model {

        protected $guarded = array('id', 'password');

    }
```

> **注意：** 使用`guarded` 时，`Input::get()` 或任何用户可以控制的未过滤数据，永远不应该传入`save` 或`update` 方法，因为没有在「黑名单」内的字段可能被更新。

#### 阻挡所有属性被批量赋值

上面的例子中，`id` 和`password` 属性**不会**被批量赋值，而所有其他的属性则允许批量赋值。您也可以使用 guard 属性阻止所有属性被批量赋值：

```
    protected $guarded = array('*');
```

## 新增，更新，删除

要从模型新增一条数据到数据库，只要建立一个模型实例并调用`save` 方法即可。

#### 储存新的模型数据

```
    $user = new User;

    $user->name = 'John';

    $user->save();
```

> **注意：** 通常 Eloquent 模型主键值会自动递增。但是您若想自定义主键，将`incrementing` 属性设成 false 。

也可以使用`create` 方法存入新的模型数据，新增完后会返回新增的模型实例。但是在新增前，需要先在模型类里设定好`fillable` 或`guarded` 属性，因为 Eloquent 默认会防止批量赋值。

在新模型数据被储存或新增后，若模型有自动递增主键，可以从对象取得`id` 属性值：

```
    $insertedId = $user->id;
```

#### 在模型里设定 Guarded 属性

```
    class User extends Model {

        protected $guarded = array('id', 'account_id');

    }
```

#### 使用模型的 Create 方法

```
    // 在数据库中建立一个新的用户...
    $user = User::create(array('name' => 'John'));

    // 以属性找用户，若没有则新增并取得新的实例...
    $user = User::firstOrCreate(array('name' => 'John'));

    // 以属性找用户，若没有则建立新的实例...
    $user = User::firstOrNew(array('name' => 'John'));
```

#### 更新取出的模型

要更新模型，可以取出它，更改属性值，然后使用`save` 方法：

```
    $user = User::find(1);

    $user->email = 'john@foo.com';

    $user->save();
```

#### 储存模型和关联数据

有时您可能不只想要储存模型本身，也想要储存关联的数据。您可以使用`push` 方法达到目的：

```
    $user->push();
```

您可以结合查询语句，批次更新模型：

```
    $affectedRows = User::where('votes', '>', 100)->update(array('status' => 2));
```

> **注意： ** 若使用 Eloquent 查询构造器批次更新模型，则不会触发模型事件。

#### 删除模型

要删除模型，只要使用实例调用`delete` 方法：

```
    $user = User::find(1);

    $user->delete();
```

#### 按主键值删除模型

```
    User::destroy(1);

    User::destroy(array(1, 2, 3));

    User::destroy(1, 2, 3);
```

当然，您也可以结合查询语句批次删除模型：

```
    $affectedRows = User::where('votes', '>', 100)->delete();
```

#### 只更新模型的时间戳

如果您只想要更新模型的时间戳，您可以使用`touch` 方法：

```
    $user->touch();
```

## 软删除

通过软删除方式删除了一个模型后，模型中的数据并不是真的从数据库被移除。而是会设定`deleted_at`时间戳。要让模型使用软删除功能，只要在模型类里加入`SoftDeletingTrait` 即可：

```
    use Illuminate\Database\Eloquent\SoftDeletes;

    class User extends Model {

        use SoftDeletes;

        protected $dates = ['deleted_at'];

    }
```

要加入`deleted_at` 字段到数据库表，可以在迁移文件里使用 s`oftDeletes` 方法：

```
    $table->softDeletes();
```

现在当您使用模型调用`delete` 方法时，`deleted_at`字段会被更新成现在的时间戳。在查询使用软删除功能的模型时，被「删除」的模型数据不会出现在查询结果里。

#### 强制查询软删除数据

要强制让已被软删除的模型数据出现在查询结果里，在查询时使用`withTrashed` 方法：

```
    $users = User::withTrashed()->where('account_id', 1)->get();
```

`withTrashed` 也可以用在关联查询：

```
    $user->posts()->withTrashed()->get();
```

如果您只想查询被软删除的模型数据，可以使用`onlyTrashed` 方法：

```
    $users = User::onlyTrashed()->where('account_id', 1)->get();
```

要把被软删除的模型数据恢复，使用`restore` 方法：
```
    $user->restore();
```

您也可以结合查询语句使用`restore` ：

```
    User::withTrashed()->where('account_id', 1)->restore();
```

如同`withTrashed` ，`restore` 方法也可以用在关联对象：

```
    $user->posts()->restore();
```

如果想要真的从模型数据库删除，使用`forceDelete` 方法：

```
    $user->forceDelete();
```

`forceDelete` 方法也可以用在关联对象：

```
    $user->posts()->forceDelete();
```

要确认模型是否被软删除了，可以使用`trashed` 方法：

```
    if ($user->trashed())
    {
        //
    }
```

## 时间戳

默认 Eloquent 会自动维护数据库表的`created_at` 和`updated_at` 字段。只要把这两个「时间戳」字段加到数据库表， Eloquent 就会处理剩下的工作。如果不想让 Eloquent 自动维护这些字段，把下面的属性加到模型类里：

#### 关闭自动更新时间戳

```
    class User extends Model {

        protected $table = 'users';

        public $timestamps = false;

    }
```

#### 自定义时间戳格式

如果想要自定义时间戳格式，可以在模型类里重写`getDateFormat` 方法：

```
    class User extends Model {

        protected function getDateFormat()
        {
            return 'U';
        }

    }
```

## 范围查询

#### 定义范围查询

范围查询可以让您轻松的重复利用模型的查询逻辑。要设定范围查询，只要定义有`scope` 前缀的模型方法：

```
    class User extends Model {

        public function scopePopular($query)
        {
            return $query->where('votes', '>', 100);
        }

        public function scopeWomen($query)
        {
            return $query->whereGender('W');
        }

    }
```

#### 使用范围查询

```
    $users = User::popular()->women()->orderBy('created_at')->get();
```

#### 动态范围查询

有时您可能想要定义可接受参数的范围查询方法。只要把参数加到方法里：

```
    class User extends Model {

        public function scopeOfType($query, $type)
        {
            return $query->whereType($type);
        }

    }
```

然后把参数值传到范围查询方法调用里：

```
    $users = User::ofType('member')->get();
```

## Global Scopes

有时您可能希望定义一个 scope 可以用于模型的所有查询中。本质上，这也是 Eloquent 的"软删除"功能的实现原理。Global scopes 是通过 PHP traits 的组合以及实现`Illuminate\Database\Eloquent\ScopeInterface` 接口来定义的。

首先，我们需要定义一个 trait。 这里我们用 Laravel 的`SoftDeletes` 举例：

```
    trait SoftDeletes {

        /**
         * Boot the soft deleting trait for a model.
         *
         * @return void
         */
        public static function bootSoftDeletes()
        {
            static::addGlobalScope(new SoftDeletingScope);
        }

    }
```

如果一个 Eloquent 模型引入了一个 trait ，而这个 trait 中带有符合`bootNameOfTrait` 惯例命名的方法 ,那么这个方法会在 Eloquent 模型启动的时候调用， 您可以在此时注册 global scope ，或者做一些其他您想要的操作。定义的 scope 必须实现`ScopeInterface` 接口，这个接口提供了两个方法：`apply` 和`remove`。

`apply` 方法接受一个`Illuminate\Database\Eloquent\Builder` 查询构造器对象以及它所应用的`Model`，用来添加这个 scope 所需的额外的`where` 子句。而`remove` 方法同样接受一个`Builder` 对象以及`Model` ，用来反向的执行`apply` 操作。也就是说，`remove` 方法应该移除已经添加的`where` 子句 (或者其他查询子句)。因此，我们的`SoftDeletingScope` 的方法应该如下：

```
    /**
     * Apply the scope to a given Eloquent query builder.
     *
     * @param  \Illuminate\Database\Eloquent\Builder  $builder
     * @return void
     */
    public function apply(Builder $builder, Model $model)
    {
        $model = $builder->getModel();

        $builder->whereNull($model->getQualifiedDeletedAtColumn());
    }

    /**
     * Remove the scope from the given Eloquent query builder.
     *
     * @param  \Illuminate\Database\Eloquent\Builder  $builder
     * @return void
     */
    public function remove(Builder $builder, Model $model)
    {
        $column = $model->getQualifiedDeletedAtColumn();

        $query = $builder->getQuery();

        foreach ((array) $query->wheres as $key => $where)
        {
            // If the where clause is a soft delete date constraint, we will remove it from
            // the query and reset the keys on the wheres. This allows this developer to
            // include deleted model in a relationship result set that is lazy loaded.
            if ($this->isSoftDeleteConstraint($where, $column))
            {
                unset($query->wheres[$key]);

                $query->wheres = array_values($query->wheres);
            }
        }
    }
```

## 关联

当然，您的数据库表很可能跟另一张表相关联。例如，一篇 blog 文章可能有很多评论，或是一张订单跟下单客户相关联。 Eloquent 让管理和处理这些关联变得很容易。 Laravel 有很多种关联类型：

*   [一对一](#one-to-one)
*   [一对多](#one-to-many)
*   [多对多](#many-to-many)
*   [远层一对多关联](#has-many-through)
*   [多态关联](#polymorphic-relations)
*   [多态的多对多关联](#many-to-many-polymorphic-relations)
<a name="one-to-one"></a>
### 一对一

#### 定义一对一关联

一对一关联是很基本的关联。例如一个`User` 模型会对应到一个`Phone` 。 在 Eloquent 里可以像下面这样定义关联：

```
    class User extends Model {

        public function phone()
        {
            return $this->hasOne('App\Phone');
        }

    }
```

传到`hasOne` 方法里的第一个参数是关联模型的类名称。定义好关联之后，就可以使用 Eloquent 的[动态属性](#dynamic-properties)取得关联对象：

```
    $phone = User::find(1)->phone;
```

SQL 会执行如下语句：

```
    select * from users where id = 1

    select * from phones where user_id = 1
```

注意， Eloquent 假设对应的关联模型数据库表里，外键名称是基于模型名称。在这个例子里，默认`Phone` 模型数据库表会以`user_id` 作为外键。如果想要更改这个默认，可以传入第二个参数到`hasOne` 方法里。更进一步，您可以传入第三个参数，指定关联的外键要对应到本身的哪个字段：

```
    return $this->hasOne('App\Phone', 'foreign_key');

    return $this->hasOne('App\Phone', 'foreign_key', 'local_key');
```

#### 定义相对的关联

要在`Phone` 模型里定义相对的关联，可以使用`belongsTo` 方法：

```
    class Phone extends Model {

        public function user()
        {
            return $this->belongsTo('App\User');
        }

    }
```

在上面的例子里， Eloquent 默认会使用`phones` 数据库表的`user_id` 字段查询关联。如果想要自己指定外键字段，可以在`belongsTo` 方法里传入第二个参数：

```
    class Phone extends Model {

        public function user()
        {
            return $this->belongsTo('App\User', 'local_key');
        }

    }
```

除此之外，也可以传入第三个参数指定要参照上层数据库表的哪个字段：

```
    class Phone extends Model {

        public function user()
        {
            return $this->belongsTo('App\User', 'local_key', 'parent_key');
        }

    }
```
<a name="one-to-many"></a>
### 一对多

一对多关联的例子如，一篇 Blog 文章可能「有很多」评论。可以像这样定义关联：

```
    class Post extends Model {

        public function comments()
        {
            return $this->hasMany('App\Comment');
        }

    }
```

现在可以经由[动态属性](#dynamic-properties)取得文章的评论：

```
    $comments = Post::find(1)->comments;
```

如果需要增加更多条件限制，可以在调用`comments` 方法后面通过链式查询条件方法：

```
    $comments = Post::find(1)->comments()->where('title', '=', 'foo')->first();
```    

同样的，您可以传入第二个参数到`hasMany` 方法更改默认的外键名称。以及，如同`hasOne` 关联，可以指定本身的对应字段：

```
    return $this->hasMany('App\Comment', 'foreign_key');

    return $this->hasMany('App\Comment', 'foreign_key', 'local_key');
```

#### 定义相对的关联

要在`Comment` 模型定义相对应的关联，可使用`belongsTo` 方法：

```
    class Comment extends Model {

        public function post()
        {
            return $this->belongsTo('App\Post');
        }

    }
```
<a name="many-to-many"></a>
### 多对多

多对多关联更为复杂。这种关联的例子如，一个用户（ user ）可能用有很多身份（ role ），而一种身份可能很多用户都有。例如很多用户都是「管理者」。多对多关联需要用到三个数据库表：`users` ，`roles` ，和`role_user` 。`role_user` 枢纽表命名是以相关联的两个模型数据库表，依照字母顺序命名，枢纽表里面应该要有`user_id` 和`role_id` 字段。

可以使用`belongsToMany` 方法定义多对多关系：

```
    class User extends Model {

        public function roles()
        {
            return $this->belongsToMany('App\Role');
        }

    }
```

现在我们可以从`User` 模型取得 roles：

```
    $roles = User::find(1)->roles;
```

如果不想使用默认的枢纽数据库表命名方式，可以传递数据库表名称作为`belongsToMany` 方法的第二个参数：

```
    return $this->belongsToMany('App\Role', 'user_roles');
```

也可以更改默认的关联字段名称：

``` 
    return $this->belongsToMany('App\Role', 'user_roles', 'user_id', 'foo_id');
```

当然，也可以在`Role` 模型定义相对的关联：

```
    class Role extends Model {

        public function users()
        {
            return $this->belongsToMany('App\User');
        }

    }
```

### Has Many Through 远层一对多关联

「远层一对多关联」提供了方便简短的方法，可以经由多层间的关联取得远层的关联。例如，一个`Country` 模型可能通过`Users` 关联到很多`Posts` 模型。 数据库表间的关系可能看起来如下：

```
    countries
        id - integer
        name - string

    users
        id - integer
        country_id - integer
        name - string

    posts
        id - integer
        user_id - integer
        title - string
```

虽然`posts` 数据库表本身没有`country_id` 字段，但`hasManyThrough` 方法让我们可以使用`$country->posts` 取得 country 的 posts。我们可以定义以下关联：

```
    class Country extends Model {

        public function posts()
        {
            return $this->hasManyThrough('App\Post', 'User');
        }

    }
```

如果想要手动指定关联的字段名称，可以传入第三和第四个参数到方法里：

```
    class Country extends Model {

        public function posts()
        {
            return $this->hasManyThrough('App\Post', 'User', 'country_id', 'user_id');
        }

    }
```
<a name="has-many-through"></a>
### 多态关联

多态关联可以用一个简单的关联方法，就让一个模型同时关联多个模型。例如，您可能想让 photo 模型同时和一个 staff 或 order 模型关联。可以定义关联如下：

```
    class Photo extends Model {

        public function imageable()
        {
            return $this->morphTo();
        }

    }

    class Staff extends Model {

        public function photos()
        {
            return $this->morphMany('App\Photo', 'imageable');
        }

    }

    class Order extends Model {

        public function photos()
        {
            return $this->morphMany('App\Photo', 'imageable');
        }

    }
```

#### 取得多态关联对象

现在我们可以从 staff 或 order 模型取得多态关联对象：

```
    $staff = Staff::find(1);

    foreach ($staff->photos as $photo)
    {
        //
    }
```

#### 取得多态关联对象的拥有者

然而，多态关联真正神奇的地方，在于要从`Photo` 模型取得 staff 或 order 对象时：

```
    $photo = Photo::find(1);

    $imageable = $photo->imageable;
```

Photo 模型里的`imageable` 关联会返回`Staff` 或`Order` 实例，取决于这是哪一种模型拥有的照片。

#### 多态关联的数据库表结构

为了理解多态关联的运作机制，来看看它们的数据库表结构：

```
    staff
        id - integer
        name - string

    orders
        id - integer
        price - integer

    photos
        id - integer
        path - string
        imageable_id - integer
        imageable_type - string
```

要注意的重点是`photos` 数据库表的`imageable_id` 和`imageable_type`。在上面的例子里， ID 字段会包含 staff 或 order 的 ID，而 type 是拥有者的模型类名称。这就是让 ORM 在取得`imageable` 关联对象时，决定要哪一种模型对象的机制。
<a name="many-to-many-polymorphic-relations"></a>
### 多态的多对多关联

#### Polymorphic Many To Many Relation Table Structure 多态的多对多关联数据库表结构

除了一般的多态关联，也可以使用多对多的多态关联。例如，Blog 的`Post` 和`Video` 模型可以共用多态的`Tag` 关联模型。首先，来看看数据库表结构：

```
    posts
        id - integer
        name - string

    videos
        id - integer
        name - string

    tags
        id - integer
        name - string

    taggables
        tag_id - integer
        taggable_id - integer
        taggable_type - string
```

现在，我们准备好设定模型关联了。`Post` 和`Video` 模型都可以经由`tags` 方法建立`morphToMany` 关联：

 ```
    class Post extends Model {

        public function tags()
        {
            return $this->morphToMany('App\Tag', 'taggable');
        }

    }
```

在`Tag` 模型里针对每一种关联建立一个方法：

```
    class Tag extends Model {

        public function posts()
        {
            return $this->morphedByMany('App\Post', 'taggable');
        }

        public function videos()
        {
            return $this->morphedByMany('App\Video', 'taggable');
        }

    }
```

## 关联查询

#### 根据关联条件查询

在取得模型数据时，您可能想要以关联模型作为查询限制。例如，您可能想要取得所有「至少有一篇评论」的Blog 文章。可以使用`has` 方法达成目的：

```
    $posts = Post::has('comments')->get();
```

也可以指定运算符和数量：

```
    $posts = Post::has('comments', '>=', 3)->get();
```

也可以使用"点号"的形式来获取嵌套的`has` 声明：

```
    $posts = Post::has('comments.votes')->get();
```

如果想要更进阶的用法，可以使用`whereHas` 和`orWhereHas` 方法，在`has` 查询里设置 "where" 条件 ：

```
    $posts = Post::whereHas('comments', function($q)
    {
        $q->where('content', 'like', 'foo%');

    })->get();
```

### 动态属性

Eloquent 可以经由动态属性取得关联对象。 Eloquent 会自动进行关联查询，而且会很聪明的知道应该要使用`get`（用在一对多关联）或是`first` （用在一对一关联）方法。可以经由和「关联方法名称相同」的动态属性取得对象。例如，如下面的模型对象`$phone`：

```
    class Phone extends Model {

        public function user()
        {
            return $this->belongsTo('App\User');
        }

    }

    $phone = Phone::find(1);
```

您可以不用像下面这样打印用户的 email ：

```
    echo $phone->user()->first()->email;
```

而可以简写如下：

```
    echo $phone->user->email;
```

> **注意：** 若取得的是许多关联对象，会返回`Illuminate\Database\Eloquent\Collection` 对象。

## 预载入

预载入是用来减少 N + 1 查询问题。例如，一个`Book` 模型数据会关联到一个`Author` 。关联会像下面这样定义：

```
    class Book extends Model {

        public function author()
        {
            return $this->belongsTo('App\Author');
        }

    }
```

现在考虑下面的代码：

```
    foreach (Book::all() as $book)
    {
        echo $book->author->name;
    }
```

上面的循环会执行一次查询取回所有数据库表上的书籍，然而每本书籍都会执行一次查询取得作者。所以若我们有 25 本书，就会进行 26次查询。

很幸运地，我们可以使用预载入大量减少查询次数。使用`with` 方法指定想要预载入的关联对象：

```
    foreach (Book::with('author')->get() as $book)
    {
        echo $book->author->name;
    }
```

现在，上面的循环总共只会执行两次查询：

```
    select * from books

    select * from authors where id in (1, 2, 3, 4, 5, ...)
```

使用预载入可以大大提高程序的性能。

当然，也可以同时载入多种关联：

```
    $books = Book::with('author', 'publisher')->get();
```

甚至可以预载入巢状关联：

```
    $books = Book::with('author.contacts')->get();
```

上面的例子中，`author` 关联会被预载入， author 的`contacts` 关联也会被预载入。

### 预载入条件限制

有时您可能想要预载入关联，同时也想要指定载入时的查询限制。下面有一个例子：

```
    $users = User::with(array('posts' => function($query)
    {
        $query->where('title', 'like', '%first%');

    }))->get();
```

上面的例子里，我们预载入了 user 的 posts 关联，并限制条件为 post 的 title 字段需包含 "first" 。

当然，预载入的闭合函数里不一定只能加上条件限制，也可以加上排序：

```
    $users = User::with(array('posts' => function($query)
    {
        $query->orderBy('created_at', 'desc');

    }))->get();
```

### 延迟预载入

也可以直接从模型的 collection 预载入关联对象。这对于需要根据情况决定是否载入关联对象时，或是跟缓存一起使用时很有用。

```
    $books = Book::all();

    $books->load('author', 'publisher');
```

你可以传入一个闭包来对查询构建器进行条件限制：

```
    $books->load(['author' => function($query)
    {
        $query->orderBy('published_date', 'asc');
    }]);
```

## 新增关联模型

#### 附加一个关联模型

您常常会需要加入新的关联模型。例如新增一个 comment 到 post 。除了手动设定模型的`post_id` 外键，也可以从上层的`Post` 模型新增关联的 comment ：

```
    $comment = new Comment(array('message' => 'A new comment.'));

    $post = Post::find(1);

    $comment = $post->comments()->save($comment);
```

上面的例子里，新增的 comment 模型中`post_id` 字段会被自动设定。

如果想要同时新增很多关联模型：

```
    $comments = array(
        new Comment(array('message' => 'A new comment.')),
        new Comment(array('message' => 'Another comment.')),
        new Comment(array('message' => 'The latest comment.'))
    );

    $post = Post::find(1);

    $post->comments()->saveMany($comments);
```

### 从属关联模型 ( Belongs To )

要更新`belongsTo` 关联时，可以使用`associate` 方法。这个方法会设定子模型的外键：

```
    $account = Account::find(10);

    $user->account()->associate($account);

    $user->save();
```

### 新增多对多关联模型 ( Many To Many )

您也可以新增多对多的关联模型。让我们继续使用`User` 和`Role` 模型作为例子。我们可以使用`attach` 方法简单地把 roles 附加给一个 user：

#### 附加多对多模型

``` 
    $user = User::find(1);

    $user->roles()->attach(1);
```

也可以传入要存在枢纽表中的属性数组：

``` 
    $user->roles()->attach(1, array('expires' => $expires));
```

当然，有`attach` 方法就会有相反的`detach` 方法：

```
    $user->roles()->detach(1);
```

`attach` 和`detach` 都可以接受ID数组作为参数：

```
    $user = User::find(1);

    $user->roles()->detach([1, 2, 3]);

    $user->roles()->attach([1 => ['attribute1' => 'value1'], 2, 3]);
```

#### 使用 Sync 方法同时附加一个以上多对多关联

您也可以使用`sync` 方法附加关联模型。`sync` 方法会把根据 ID 数组把关联存到枢纽表。附加完关联后，枢纽表里的模型只会关联到 ID 数组里的 id ：

```
    $user->roles()->sync(array(1, 2, 3));
```

#### Sync 时在枢纽表加入额外数据

也可以在把每个 ID 加入枢纽表时，加入其他字段的数据：

```
    $user->roles()->sync(array(1 => array('expires' => true)));
```

有时您可能想要使用一个命令，在建立新模型数据的同时附加关联。可以使用`save` 方法达成目的：

```
    $role = new Role(array('name' => 'Editor'));

    User::find(1)->roles()->save($role);
```

上面的例子里，新的`Role` 模型对象会在储存的同时关联到`user` 模型。也可以传入属性数组把数据加到关联数据库表：

```
    User::find(1)->roles()->save($role, array('expires' => $expires));
```

## 更新上层时间戳

当模型`belongsTo` 另一个模型时，比方说一个`Comment` 属于一个`Post` ，如果能在子模型被更新时，更新上层的时间戳，这将会很有用。例如，当`Comment` 模型更新时，您可能想要能够同时自动更新`Post` 的`updated_at` 时间戳。 Eloquent 让事情变得很简单。只要在子关联的类里，把关联方法名称加入`touches` 属性即可：

```
    class Comment extends Model {

        protected $touches = array('post');

        public function post()
        {
            return $this->belongsTo('App\Post');
        }

    }
```

现在，当您更新`Comment` 时，对应的`Post` 会自动更新`updated_at` 字段：

```
    $comment = Comment::find(1);

    $comment->text = 'Edit to this comment!';

    $comment->save();
```

## 使用枢纽表

如您所知，要操作多对多关联需要一个中间的数据库表。 Eloquent 提供了一些有用的方法可以和这张表互动。例如，假设`User` 对象关联到很多`Role` 对象。取出这些关联对象时，我们可以在关联模型上取得`pivot` 数据库表的数据：

```
    $user = User::find(1);

    foreach ($user->roles as $role)
    {
        echo $role->pivot->created_at;
    }
```

注意我们取出的每个`Role` 模型对象会自动给一个`pivot` 属性。这属性包含了枢纽表的模型数据，可以像一般的 Eloquent 模型一样使用。

默认`pivot` 对象只会有关联键的属性。如果您想让 pivot 可以包含其他枢纽表的字段，可以在定义关联方法时指定那些字段：

```
    return $this->belongsToMany('App\Role')->withPivot('foo', 'bar');
```

现在可以在`Role` 模型的`pivot` 对象上取得`foo` 和`bar` 属性了。

如果您想要可以自动维护枢纽表的`created_at` 和`updated_at` 时间戳，在定义关联方法时加上`withTimestamps` 方法：

```
    return $this->belongsToMany('App\Role')->withTimestamps();
```

#### 删除枢纽表的关联数据

要删除模型在枢纽表的所有关联数据，可以使用`detach` 方法：

```
    User::find(1)->roles()->detach();
```

注意，如上的操作不会移除`roles` 数据库表里面的数据，只会移除枢纽表里的关联数据。

#### 更新枢纽表的数据

有时您只想更新枢纽表的数据，而没有要移除关联。如果您想更新枢纽表，可以像下面的例子使用`updateExistingPivot` 方法：

```
    User::find(1)->roles()->updateExistingPivot($roleId, $attributes);
```

#### 自定义枢纽模型

Laravel 允许您自定义枢纽模型。要自定义模型，首先要建立一个继承 Eloquent 的「基本」模型类。在其他的 Eloquent 模型继承这个自定义的基本类，而不是默认的 Eloquent 。在基本模型类里，加入下面的方法返回自定义的枢纽模型实例：

```
    public function newPivot(Model $parent, array $attributes, $table, $exists)
    {
        return new YourCustomPivot($parent, $attributes, $table, $exists);
    }
```

## 集合

所有 Eloquent 查询返回的数据，如果结果多于一条，不管是经由`get` 方法或是`relationship`，都会转换成集合对象返回。这个对象实现了`IteratorAggregate` PHP 接口，所以可以像数组一般进行遍历。而集合对象本身还拥有很多有用的方法可以操作模型数据。

#### 确认集合中里是否包含特定键值

例如，我们可以使用`contains` 方法，确认结果数据中，是否包含主键为特定值的对象。

```
    $roles = User::find(1)->roles;

    if ($roles->contains(2))
    {
        //
    }
```

集合也可以转换成数组或 JSON：

```
    $roles = User::find(1)->roles->toArray();

    $roles = User::find(1)->roles->toJson();
```

如果集合被转换成字符串类型，会返回 JSON 格式：

```
    $roles = (string) User::find(1)->roles;
```

#### 集合遍历

Eloquent 集合里包含了一些有用的方法可以进行循环或是进行过滤：

```
    $roles = $user->roles->each(function($role)
    {
        //
    });
```

#### 集合过滤

过滤集合时，回调函数的使用方式和[array_filter]((http://php.net/manual/en/function.array-filter.php)) 里一样。

```
    $users = $users->filter(function($user)
    {
        return $user->isAdmin();
    });
```

> **注意：** 如果要在过滤集合之后转成 JSON，转换之前先调用`values` 方法重设数组的键值。

#### 遍历传入集合里的每个对象到回调函数

```
    $roles = User::find(1)->roles;

    $roles->each(function($role)
    {
        //
    });
```

#### 依照属性值排序

```
    $roles = $roles->sortBy(function($role)
    {
        return $role->created_at;
    });
```

#### 依照属性值排序

```
    $roles = $roles->sortBy('created_at');
```

#### 返回自定义的集合对象

有时您可能想要返回自定义的集合对象，让您可以在集合类里加入想要的方法。可以在 Eloquent 模型类里重写`newCollection` 方法：

```
    class User extends Model {

        public function newCollection(array $models = array())
        {
            return new CustomCollection($models);
        }

    }
```

## 获取器和修改器

#### 定义获取器

Eloquent 提供了一种便利的方法，可以在获取或设定属性时进行转换。要定义获取器，只要在模型里加入类似`getFooAttribute` 的方法。注意方法名称应该使用驼峰式大小写命名，而对应的 database 字段名称是下划线分隔小写命名：

```
    class User extends Model {

        public function getFirstNameAttribute($value)
        {
            return ucfirst($value);
        }

    }
```

上面的例子中， first_name 字段设定了一个获取器。注意传入方法的参数是原本的字段数据。

#### 定义修改器

修改器的定义方式也是类似的：

```
    class User extends Model {

        public function setFirstNameAttribute($value)
        {
            $this->attributes['first_name'] = strtolower($value);
        }

    }
```

## 日期转换器

默认 Eloquent 会把`created_at` 和`updated_at` 字段属性转换成[Carbon](https://github.com/briannesbitt/Carbon) 实例，它提供了很多有用的方法，并继承了 PHP 原生的`DateTime` 类。

您可以通过重写模型的`getDates` 方法，自定义哪个字段可以被自动转换，或甚至完全关闭这个转换：

```
    public function getDates()
    {
        return array('created_at');
    }
```

当字段是表示日期的时候，可以将值设为 UNIX timestamp 、日期字符串（ Y-m-d ）、 日期时间（ date-time ）字符串，当然还有`DateTime` 或`Carbon` 实例。

要完全关闭日期转换功能，只要从`getDates` 方法返回空数组即可：

```
    public function getDates()
    {
        return array();
    }
```

## 属性类型转换

如果您想要某些属性始终转换成另一个数据类型, 您可以在模型中增加`casts` 属性。否则，您需要为每个属性定义修改器，这样会增加更多的时间开销。这里有一个使用`casts` 属性的例子：

```
    /**
     * 需要被转换成基本类型的属性值。
     *
     * @var array
     */
    protected $casts = [
        'is_admin' => 'boolean',
    ];
```

现在当你获取`is_admin` 属性时始终会是布尔类型，甚至在数据库中存储的这个值是一个整型也会被转换。其他支持的类型转换值有：`integer`,`real`,`float`,`double`,`string`,`boolean`,`object` 和`array` 。

如果您存储的值是一个序列化的 JSON 时，那么`array` 类型转换将会非常有用。比如，您的数据表里有一个 TEXT 类型的字段存储着序列化后的 JSON 数据， 通过增加`array` 类型转换, 当获取这个属性的时候会自动反序列化成 PHP 的数组：

```
    /**
     * 需要被转换成基本类型的属性值。
     *
     * @var array
     */
    protected $casts = [
        'options' => 'array',
    ];
```

现在，当你使用 Eloquent 模型时：

```
    $user = User::find(1);

    // $options 是一个数组...
    $options = $user->options;

    // options 会自动序列化成 JSON...
    $user->options = ['foo' => 'bar'];
```

## 模型事件

Eloquent 模型有很多事件可以触发，让您可以在模型操作的生命周期的不同时间点，使用下列方法绑定事件：`creating`,`created`,`updating`,`updated`,`saving`,`saved`,`deleting`,`deleted`,`restoring`,`restored`。

当一个对象初次被储存到数据库，`creating` 和`created` 事件会被触发。如果不是新对象而调用了`save` 方法，`updating` /`updated` 事件会被触发。而两者的`saving` /`saved` 事件都会被触发。

#### 使用事件取消数据库操作

如果`creating` 、`updating` 、`saving` 、`deleting` 事件返回 false 的话，就会取消数据库操作

```
    User::creating(function($user)
    {
        if ( ! $user->isValid()) return false;
    });
```

#### 注册事件监听者的方式

您可以在`EventServiceProvider` 中注册您的模型事件绑定。比如：

```
    /**
     * Register any other events for your application.
     *
     * @param  \Illuminate\Contracts\Events\Dispatcher  $events
     * @return void
     */
    public function boot(DispatcherContract $events)
    {
        parent::boot($events);

        User::creating(function($user)
        {
            //
        });
    }
```

## 模型观察者

要整合模型的事件处理，可以注册一个模型观察者。观察者类里要设定对应模型事件的方法。例如，观察者类里可能有`creating`、`updating` 、`saving` 方法，还有其他对应模型事件名称的方法：

例如，一个模型观察者类可能看起来如下：

```
    class UserObserver {

        public function saving($model)
        {
            //
        }

        public function saved($model)
        {
            //
        }

    }
```

可以使用`observe` 方法注册一个观察者实例：

```
    User::observe(new UserObserver);
```

## 模型 URL 生成

当你把一个模型实例传递给`route` 或者`action` 方法时，模型的主键会被插入到生成的 URI 中。比如：

```
    Route::get('user/{user}', 'UserController@show');

    action('UserController@show', [$user]);
```

在这个例子中`$user->id` 属性会被插入到生成的 URL 的`{user}` 这个占位符中。不过，如果你想使用其他的属性而不是 ID 的话，你可以覆盖模型的`getRouteKey` 方法：

```
    public function getRouteKey()
    {
        return $this->slug;
    }
```

## 转换成数组 / JSON

#### 将模型数据转成数组

当构建 JSON API 时，您可能常常需要把模型和关联对象转换成数组或JSON。所以Eloquent里已经包含了这些方法。要把模型和已载入的关联对象转成数组，可以使用`toArray` 方法：

```
    $user = User::with('roles')->first();

    return $user->toArray();
```

注意也可以把整个的模型集合转换成数组：

```
    return User::all()->toArray();
```

#### 将模型转换成 JSON

要把模型转换成 JSON，可以使用`toJson` 方法：

```
    return User::find(1)->toJson();
```

#### 从路由中返回模型

注意当模型或集合被转换成字符串类型时会自动转换成 JSON 格式，这意味着您可以直接从路由返回 Eloquent 对象！

```
    Route::get('users', function()
    {
        return User::all();
    });
```

#### 转换成数组或 JSON 时隐藏属性

有时您可能想要限制能出现在数组或 JSON 格式的属性数据，比如密码字段。只要在模型里增加`hidden` 属性即可

```
    class User extends Model {

        protected $hidden = array('password');

    }
```

> **注意：** 要隐藏关联数据，要使用关联的_方法_名称，而不是动态获取的属性名称。

此外，可以使用`visible` 属性定义白名单：

```
    protected $visible = array('first_name', 'last_name');
```

有时候您可能想要增加不存在数据库字段的属性数据。这时候只要定义一个获取器即可：

```
    public function getIsAdminAttribute()
    {
        return $this->attributes['admin'] == 'yes';
    }
```

定义好获取器之后，再把对应的属性名称加到模型里的`appends` 属性：

```
    protected $appends = array('is_admin');
```

把属性加到`appends` 数组之后，在模型数据转换成数组或 JSON 格式时就会有对应的值。在`appends` 数组中定义的值同样遵循模型中`visible` 和`hidden` 的设定。