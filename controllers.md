# HTTP 控制器

简介
基础控制器
控制器中间件
隐式控制器
RESTful 资源控制器
依赖注入和控制器
路由缓存

## 简介

除了在单一的 `routes.php` 文件中定义所有的请求处理逻辑之外，你可能希望使用控制器类来组织此行为。控制器可将相关的 HTTP 请求处理逻辑组成一个类。控制器通常存放在 `app/Http/Controllers` 此目录中。


## 基础控制器

这里是一个基础控制器类的例子：

```
<?php namespace App\Http\Controllers;

use App\Http\Controllers\Controller;

class UserController extends Controller {

    /**
     * 显示所给定的用户个人数据。
     *
     * @param  int  $id
     * @return Response
     */
    public function showProfile($id)
    {
        return view('user.profile', ['user' => User::findOrFail($id)]);
    }

}
```
我们可以通过如下方式引导路由至对应的控制器动作：

```
Route::get('user/{id}', 'UserController@showProfile');
```
**`注意： 所有的控制器都应该扩展基础控制器类。`**

### 控制器和命名空间

有一点非常重要，那就是我们无需指明完整的控制器命名空间，在类名称中 `App\Http\Controllers` 之后的部分即可用于表示「根」命名空间。 `RouteServiceProvider` 默认会在包含根控制器命名空间的路由群组中，加载 `routes.php `此文件。

若你要在 `App\Http\Controllers` 此目录深层使用 PHP 命名空间以嵌套化或组织你的控制器，只要使用相对于 `App\Http\Controllers` 根命名空间的特定类名称即可。因此，若你的控制器类全名为 `App\Http\Controllers\Photos\AdminController`，你可以像这样注册一个路由：

```
Route::get('foo', 'Photos\AdminController@method');
```
### 命名控制器路由

和闭包路由一样，你也可以指定控制器路由的名称。

```
Route::get('foo', ['uses' => 'FooController@method', 'as' => 'name']);
```
### 指向控制器行为的 URL

要产生一个指向控制器行为的 URL，可使用 `action` 辅助方法。

```
$url = action('App\Http\Controllers\FooController@method');
```
若你想仅使用相对于控制器命名空间的类名称中的一部分，来产生指向控制器行为的 URL，可用 URL 产生器注册控制器的根命名空间。

```
URL::setRootControllerNamespace('App\Http\Controllers');

$url = action('FooController@method');
```

你可以使用 `currentRouteAction` 方法来获取正在执行的控制器行为名称：

```
$action = Route::currentRouteAction();
```

## 控制器中间件

中间件 可在控制器路由中指定，例如：

```
Route::get('profile', [
    'middleware' => 'auth',
    'uses' => 'UserController@showProfile'
]);
```
此外，你也可以在控制器构造器中指定中间件 ：

```
class UserController extends Controller {

    /**
     * 建立一个新的 UserController 实例。
     */
    public function __construct()
    {
        $this->middleware('auth');

        $this->middleware('log', ['only' => ['fooAction', 'barAction']]);

        $this->middleware('subscribed', ['except' => ['fooAction', 'barAction']]);
    }

}
```

## 隐式控制器

Laravel 让你能轻易地定义单一路由来处理控制器中的每一项行为。首先，用 `Route::controller` 方法定义一个路由：

```
Route::controller('users', 'UserController');
```
`Controller` 方法接受两个参数。第一个参数是控制器欲处理的 base URI，第二个是控制器的类名称。接着只要在你的控制器中加入方法，并在名称前加上它们所对应的 HTTP 请求。

```
class UserController extends BaseController {

    public function getIndex()
    {
        //
    }

    public function postProfile()
    {
        //
    }

    public function anyLogin()
    {
        //
    }

}
```
`index` 方法会响应控制器处理的根 URI ，在这个例子中是 `users` 。

如果你的控制器行为包含多个字词，你可以在 URI 中使用「破折号」语法来访问此行为。例如，下面这个在 `UserController` 中的控制器动作会响应 `users/admin-profile` 此一 URI ：

```
public function getAdminProfile() {}
```
### 设定路由名字

如果你想“命名”一些控制器的路由，你可以给 `controller` 方法传入第三个参数：

```
Route::controller('users', 'UserController', [
    'anyLogin' => 'user.login',
]);
```

## RESTful 资源控制器

资源控制器可让你无痛建立和资源相关的 RESTful 控制器。例如，你可能希望创建一个控制器，它可用来处理针对你的应用程序所保存相片的 HTTP 请求。我们可以使用 `make:controller Artisan` 命令，快速创建这样的控制器：

```
php artisan make:controller PhotoController
```
接着，我们注册一个指向此控制器的资源路由：

```
Route::resource('photo', 'PhotoController');
```
此单一路由声明创建了多个路由，用来处理各式各样和相片资源相关的 RESTful 行为。同样地，产生的控制器已有各种和这些行为绑定的方法，包含用来通知你它们处理了那些 URI 及动词。

### 由资源控制器处理的行为

动词	路径	行为	路由名称
GET	/photo	索引	photo.index
GET	/photo/create	创建	photo.create
POST	/photo	保存	photo.store
GET	/photo/{photo}	显示	photo.show
GET	/photo/{photo}/edit	编辑	photo.edit
PUT/PATCH	/photo/{photo}	更新	photo.update
DELETE	/photo/{photo}	删除	photo.destroy

### 自定义资源路由

除此之外，你也可以指定让路由仅处理一部分的行为：

```
Route::resource('photo', 'PhotoController',
                ['only' => ['index', 'show']]);

Route::resource('photo', 'PhotoController',
                ['except' => ['create', 'store', 'update', 'destroy']]);
```
所有的资源控制器行为默认都有个路由名称。然而你可在选项中传递一个 `names` 数组来重载这些名称：

```
Route::resource('photo', 'PhotoController',
                ['names' => ['create' => 'photo.build']]);
```
### 处理嵌套资源控制器

在你的路由声明中使用「点」号来「嵌套化」资源控制器：

```
Route::resource('photos.comments', 'PhotoCommentController');
```
此路由会注册一个「嵌套的」资源，可透过像 `photos/{photos}/comments/{comments}` 这样的 URL 来访问。

```
class PhotoCommentController extends Controller {

    /**
     * 显示指定照片的评论。
     *
     * @param  int  $photoId
     * @param  int  $commentId
     * @return Response
     */
    public function show($photoId, $commentId)
    {
        //
    }

}
```
### 在资源控制器中加入其他的路由

除了默认的资源路由外，若你还需要在资源控制器中加入其他路由，应该在调用 `Route::resource` 之前先定义它们：

```
Route::get('photos/popular', 'PhotoController@method');

Route::resource('photos', 'PhotoController');
```
## 依赖注入和控制器

### 构造器注入

Laravel 服务容器 用于解析所有的 Laravel 控制器。因此，你可以在控制器所需要的构造器中，对依赖作任何的类型限制。

```
<?php namespace App\Http\Controllers;

use Illuminate\Routing\Controller;
use App\Repositories\UserRepository;

class UserController extends Controller {

    /**
     * 用户保存库实例。
     */
    protected $users;

    /**
     * 创建新的控制器实例。
     *
     * @param  UserRepository  $users
     * @return void
     */
    public function __construct(UserRepository $users)
    {
        $this->users = $users;
    }

}
```
当然了，你也可以对任何的 Laravel contract 作类型限制。只要容器能解析它，你就可以对它作类型限制。

### 方法注入

除了建构器注入外，你也可以对控制器方法的依赖作类型限制。例如，让我们对某个方法的 `Request` 实例作类型限制：

```
<?php namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Illuminate\Routing\Controller;

class UserController extends Controller {

    /**
     * 保存一个新的用户。
     *
     * @param  Request  $request
     * @return Response
     */
    public function store(Request $request)
    {
        $name = $request->input('name');

        //
    }

}
```
如果你的控制器方法预期由路由参数取得输入，只要在其他的依赖之后列出路由参数即可：

```
<?php namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Illuminate\Routing\Controller;

class UserController extends Controller {

    /**
     * 保存一个新的用户。
     *
     * @param  Request  $request
     * @param  int  $id
     * @return Response
     */
    public function update(Request $request, $id)
    {
        //
    }

}
```

**`注意： 方法注入和 模型绑定 是完全兼容的。容器可智能地判断那些参数和模型相关以及那些参数应该被注入。`**

## 路由缓存

若您的应用只使用了控制器路由，你可利用 Laravel 的路由缓存。使用路由缓存，将大幅降低注册应用程序所有路由所需要的时间。某些情况下，路由注册甚至可以快上 100 倍。要产生路由缓存，只要执行 `route:cache` Artisan 命令：

```
php artisan route:cache
```
就是这样！你的缓存路由文件将会被用来代替 `app/Http/routes.php` 此一文件。记住，若你增加了任何新的路由，你就 必须产生一个新的路由缓存。因此在应用部署时，你可能会希望只要执行 `route:cache` 命令：

要移除路由缓存文件，但不希望产生新的缓存，可使用 `route:clear` 命令：

```
php artisan route:clear
```