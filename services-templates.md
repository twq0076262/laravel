# 模板

## Blade 模板

Blade 是 Laravel 所提供的一个简单却又非常强大的模板引擎。不像控制器页面布局，Blade 是使用_模板继承_(template inheritance) 和_区块_(sections)。所有的 Blade 模板后缀名都要命名为`.blade.php`。

#### 定义一个 Blade 页面布局

```
<!-- Stored in resources/views/layouts/master.blade.php -->

<html>
    <body>
        @section('sidebar')
            This is the master sidebar.
        @show

        <div class="container">
            @yield('content')
        </div>
    </body>
</html>
```

#### 在视图模板中使用 Blade 页面布局

```
@extends('layouts.master')

@section('sidebar')
    @@parent

    <p>This is appended to the master sidebar.</p>
@stop

@section('content')
    <p>This is my body content.</p>
@stop
```

请注意 如果视图`继承(extend)` 了一个 Blade 页面布局会将页面布局中定义的区块用视图的所定义的区块重写。如果想要将页面布局中的区块内容也能在继承此布局的视图中呈现，那就要在区块中使用`@@parent` 语法指令，通过这种方式可以把内容附加到页面布局中，我们会在侧边栏区块或者页脚区块看到类似的使用。

有时候，如您不确定这个区块内容有没有被定义，您可能会想要传一个默认的值给 @yield。您可以传入第二个参数作为默认值给 @yield：

```
@yield('section', 'Default Content')
```

## 其他 Blade 控制语法结构

#### 在 Blade 视图中打印（Echoing）数据

```
Hello, {{ $name }}.

The current UNIX timestamp is {{ time() }}.
```

#### 检查数据是否存在后再打印数据

有时候您想要打印一个变量，但您不确定这个变量是否存在，通常情况下，您会想要这样写：:

```
{{ isset($name) ? $name : 'Default' }}

然而，除了写这种三元运算符语法之外，Blade 让您可以使用下面这种更简便的语法：

{{ $name or 'Default' }}
```

#### 使用花括号显示文字

如果您需要显示的一个字符串刚好被花括号包起来，您可以在花括号之前加上 @ 符号前缀来跳出 Blade 引擎的解析：

```
@{{ This will not be processed by Blade }}
```

如果您不想数据被转义, 也可以使用如下语法：

```
Hello, {!! $name !!}.
```

> **特别注意:** 在您的应用程序打印用户所提供的内容时要非常小心。请记得永远使用双重花括号来转义内容中的 HTML 实体字符串。

#### If 声明

```
@if (count($records) === 1)
    I have one record!
@elseif (count($records) > 1)
    I have multiple records!
@else
    I don't have any records!
@endif

@unless (Auth::check())
    You are not signed in.
@endunless
```

#### 循环

```
@for ($i = 0; $i < 10; $i++)
    The current value is {{ $i }}
@endfor

@foreach ($users as $user)
    <p>This is user {{ $user->id }}</p>
@endforeach

@forelse($users as $user)
    <li>{{ $user->name }}</li>
@empty
    <p>No users</p>
@endforelse

@while (true)
    <p>I'm looping forever.</p>
@endwhile
```

#### 加载子视图

```
@include('view.name')
```

您也可以通过传入数组的形式将数据传递给加载的子视图：

```
@include('view.name', ['some' => 'data'])
```

#### 重写区块

如果想要重写掉前面区块中的内容，您可以使用`overwrite` 声明：

```
@extends('list.item.container')

@section('list.item.content')
    <p>This is an item of type {{ $item->type }}</p>
@overwrite
```

#### 显示语言行

```
@lang('language.line')

@choice('language.line', 1)
```

#### 注释

```
{{-- This comment will not be in the rendered HTML --}}
```

## 扩展 Blade

Blade 甚至允许你定义自己的控制语法结构。 当一个 Blade 文件被编译时， 每一个自定义的扩展语法会与视图内容一起被调用， 您可以做任何的操作, 简单如`str_replace` 以及更为复杂的正则表达式。

Blade 的编译器带有一些辅助方法`createMatcher` 及`createPlainMatcher`，这些辅助方法可以产生您需要的表达式来帮助您构建自己的自定义扩展语法。

其中`createPlainMatcher` 方法是用在没有参数的语法指令如`@endif` 及`@stop` 等， 而`createMatcher` 方法是用在带参数的语法指令中。

下面的例子创建了一个`@datetime($var)` 语法命令, 这个命令只是简单的对`$var` 调用`->format()` 方法：

```
Blade::extend(function($view, $compiler)
{
    $pattern = $compiler->createOpenMatcher('datetime');

    return preg_replace($pattern, '$1<?php echo $2->format(\'m/d/Y H:i\')); ?>', $view);
});
```