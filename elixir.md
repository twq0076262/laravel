# Laravel Elixir

* 简介
* 安装
* 使用方式
* Gulp
* 默认目录
* 功能扩充

## 简介

Laravel Elixir 提供了简洁流畅的 API，让你能够为你的 Laravel 应用程序定义基本的 [Gulp](http://gulpjs.com/) 任务。Elixir 支持许多常见的 CSS 与 JavaScrtip 预处理器，甚至包含了测试工具。

如果你曾经对于使用 Gulp 及编译资源感到困惑，那么你绝对会爱上 Laravel Elixir！

## 安装与配置

### 安装 Node

在开始使用 Elixir 之前，你必须先确定你的开发环境上有安装 Node.js。

```
    node -v
```

默认情况下，Laravel Homestead 会包含你所需的一切；当然，如果你没有使用 Vagrant，那么你可以浏览 [Node 的下载页](https://nodejs.org/download/)进行安装。别担心，安装是很简单又快速的！

### Gulp

接着你需要全局安装 [Gulp](http://gulpjs.com/) 的 NPM 安装包，如这样：

```
    npm install --global gulp
```
### Laravel Elixir

最后的步骤就是安装 Elixir！伴随着新安装的 Laravel，你会发现根目录有个名为 `package.json` 的文件。想像它就如同你的 `composer.json` 文件，只是它定义的是 Node 的依赖，而不是 PHP。你可以使用以下的命令进行安装依赖的动作：

```
    npm install
```

## 使用方式

现在你已经安装好 Elixir，未来任何时候你都能进行编译及合并文件！ 在项目根目录下的 `gulpfile.js` 已经包含了所有的 Elixir 任务。

#### 编译 Less
```
    elixir(function(mix) {
        mix.less("app.less");
    });
```

在上述例子中，Elixir 会假设你的 Less 文件保存在 `resources/assets/less` 里。

#### 编译多个 Less 文件

```
    elixir(function(mix) {
        mix.less([
            'app.less',
            'something-else.less'
        ]);
    });
```

#### 编译 Sass

```
    elixir(function(mix) {
        mix.sass("app.scss");
    });
```

在上述例子中，Elixir 会假设你的 Sass 文件保存在 `resources/assets/sass` 里。`sass` 方法只能被调用一次，如果你想编译多个 Sass 文件，可以向 `sass` 方法传入一个数组。

默认情况下， Elixir 会使用 LibSass 作为编译引擎。 在某些情况下, 使用 Ruby 版本的 Sass 编译可能更有优势，虽然运行不是很快但是有更多的特性。假设你已经安装了 Ruby 和 Sass gem (`gem install sass`), 你可以使用 Ruby 模式，比如像这样：

```
    elixir(function(mix) {
        mix.rubySass("app.sass");
    });
```

#### 无 Source Maps 编译

```
    elixir.config.sourcemaps = false;

    elixir(function(mix) {
        mix.sass("app.scss");
    });
```

Source maps 默认情况下是开启的。因此, 每当一个文件被编译，你都会在当前目录下看到 `*.css.map` 文件。这个映射文件允许你在进行 debugging 的时候，追溯编译后的样式选择器到原有的 Sass 或者 Less 文件！如果你想要关闭这个功能，可以使用上面的代码。

#### 编译 CoffeeScript
```
    elixir(function(mix) {
        mix.coffee();
    });

```

在上述例子中，Elixir 会假设你的 CoffeeScript 文件保存在 `resources/assets/coffee` 里。

#### 编译所有的 Less 及 CoffeeScript

```
    elixir(function(mix) {
        mix.less()
           .coffee();
    });
```

#### 触发 PHPUnit 测试

```
    elixir(function(mix) {
        mix.phpUnit();
    });
```

#### 触发 PHPSpec 测试

```
    elixir(function(mix) {
        mix.phpSpec();
    });
```

#### 合并样式文件
```
    elixir(function(mix) {
        mix.styles([
            "normalize.css",
            "main.css"
        ]);
    });
```

传递给此方法的文件路径均相对于 `resources/css` 目录。

#### 合并样式文件且保存在自定义的路径

```
    elixir(function(mix) {
        mix.styles([
            "normalize.css",
            "main.css"
        ], 'public/build/css/everything.css');
    });
```

#### 从特定相对目录合并样式文件

```
    elixir(function(mix) {
        mix.styles([
            "normalize.css",
            "main.css"
        ], 'public/build/css/everything.css', 'public/css');
    });
```

`styles` 与 `scrtips` 方法可以通过传入第三个参数来决定来源文件的相对目录。

#### 合并指定目录里所有的样式文件

```
    elixir(function(mix) {
        mix.stylesIn("public/css");
    });
```

#### 合并脚本文件

```
    elixir(function(mix) {
        mix.scripts([
            "jquery.js",
            "app.js"
        ]);
    });
```

同样的，传递给此方法的文件路径均相对于 `resources/js` 目录

#### 合并指定目录里所有的脚本文件

```
    elixir(function(mix) {
        mix.scriptsIn("public/js/some/directory");
    });
```

#### 合并多组脚本文件

```
    elixir(function(mix) {
        mix.scripts(['jquery.js', 'main.js'], 'public/js/main.js')
           .scripts(['forum.js', 'threads.js'], 'public/js/forum.js');
    });
```

#### 压缩文件并加上哈希的版本号

```
    elixir(function(mix) {
        mix.version("css/all.css");
    });
```

这个动作会为你的文件名称加上独特的哈希值，以防止文件被缓存。举例来说，产生出来的文件名称可能像这样：`all-16d570a7.css`。

接着在你的视图中，你能够使用 `elixir()` 函数来正确加载名称被哈希后的文件。举例如下：

    

程序的作用下，`elixir()` 函数会将参数内的源文件名转换成被哈希后的文件名并加载。是否有如释重担的感觉呢？

您也可以传给一个数组给 `version` 方法来为多个文件进行版本管理：

```
    elixir(function(mix) {
        mix.version(["css/all.css", "js/app.js"]);
    });
```

    
 http://www.google-analytics.com/ga.js"> src="{{ elixir(" js="" app.js")="" }}"="">

#### 复制文件到新的位置

```
    elixir(function(mix) {
        mix.copy('vendor/foo/bar.css', 'public/css/bar.css');
    });
```
#### 将整个目录都复制到新的位置

```
    elixir(function(mix) {
        mix.copy('vendor/package/views', 'resources/views');
    });
```

#### 方法连接

当然，你能够串连 Elixir 大部份的方法来建立一连串的任务：

```
    elixir(function(mix) {
        mix.less("app.less")
           .coffee()
           .phpUnit()
           .version("css/bootstrap.css");
    });
```

## Gulp

现在你已经告诉 Elixir 要执行的任务，接着只需要在命令行执行 Gulp。

#### 执行一次所有注册的任务

```
    gulp
```

#### 监控文件变更

```
    gulp watch
```

#### 仅编译 javascript

```
    gulp scripts
```

#### 仅编译 css 样式

```
    gulp styles
```

#### 监控测试以及 PHP 类的变更

```
    gulp tdd
```

> **提示：** 所有的任务都会使用开发环境进行，所以压缩功能不会被执行。如果要使用上线环境，可以使用 `gulp --production`。

## 功能扩充

你甚至能够建立自己的 Gulp 任务至 Elixir 里。想像一下，你想加入一个有趣的任务，使用终端机后会打打印一些消息。看起来可能会如下：

```
     var gulp = require("gulp");
     var shell = require("gulp-shell");
     var elixir = require("laravel-elixir");

     elixir.extend("message", function(message) {

         gulp.task("say", function() {
             gulp.src("").pipe(shell("say " + message));
         });

         return this.queueTask("say");

     });
```

请注意我们 `扩增（ extend ）` Elixir 的 API 时所使用的第一个参数，稍后我们需要在 Gulpfile 中使用它，以及建立 Gulp 任务所使用的回调函数。

如果你想要让你的自定义任务能被监控，只要在监控器注册就行了。

```
    this.registerWatcher("message", "**/*.php");
```

这行程序的意思是指，当符合正则表达式的文件一经修改，就会触发 `message` 任务。

很好！接着你可以将这行程序写在 Gulpfile 的顶端，或者将它放到自定义任务的文件里。如果你选择后者，那么你必须将它加载至你的 Gulpfile，例如：

```
    require("./custom-tasks")
```

大功告成！最后你只需要将他们结合。

```
    elixir(function(mix) {
        mix.message("Tea, Earl Grey, Hot");
    });
```

加入之后，每当你触发 Gulp，Picard 就会要求一些茶。