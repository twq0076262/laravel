# Envoy 任务执行器


## 简介

Laravel Envoy 提供了简洁、轻量的语法用于定义在远程服务器上可执行的通用任务。通过 Blade 风格的语法，你可以很容易地设置任务从而完成部署、执行 Artisan 命令或其他更多工作。

> **注意：** Envoy 依赖 PHP 5.4 或更高版本，并且只能运行在 Mac / Linux 操作系统中。

## 安装

首先，通过 Composer 的 `global` 命令来安装 Envoy：

```
    composer global require "laravel/envoy=~1.0"
```

请务必将 `~/.composer/vendor/bin` 目录加入到 PATH 环境变量中，这样才能在命令行中执行 `envoy` 命令时找到可执行文件。

接下来，在项目的根目录下创建 `Envoy.blade.php` 文件。下面给出的实例代码你可以当做模板使用：

```
    @servers(['web' => '192.168.1.1'])

    @task('foo', ['on' => 'web'])
        ls -la
    @endtask
```

如上所示，在文件的开头首先定义了 `@servers` 数组。后续的任务声明中，你可以在 `on` 选项中直接引用。在 `@task` 声明里，你可以直接填写需要在服务器上执行的 Bash 脚本代码。

`init` 命令可以很方便地用来创建一个包含基本内容的 Envoy 文件：

```
    envoy init user@192.168.1.1
```

## 执行任务

使用 `run` 命令来执行任务：

```
    envoy run foo
```

如有需要，你还可以通过命令行向 Envoy 文件传递参数：

```
    envoy run deploy --branch=master
```

你可以通过 Blade 语法引用这些参数：

```
    @servers(['web' => '192.168.1.1'])

    @task('deploy', ['on' => 'web'])
        cd site
        git pull origin {{ $branch }}
        php artisan migrate
    @endtask
```

#### Bootstrapping

你可以在 `@setup` 指令中声明变量，并在 Envoy 文件中执行普通的 PHP 代码:

```
    @setup
        $now = new DateTime();

        $environment = isset($env) ? $env : "testing";
    @endsetup
```

还可以通过 `@include` 指令引入任意的 PHP 文件：

```
    @include('vendor/autoload.php');
```

## 多服务器

在多台服务器上执行一个任务是非常简单的，只需在声明任务时列出服务器名称即可：

```
    @servers(['web-1' => '192.168.1.1', 'web-2' => '192.168.1.2'])

    @task('deploy', ['on' => ['web-1', 'web-2']])
        cd site
        git pull origin {{ $branch }}
        php artisan migrate
    @endtask
```

默认情况下，任务将以串行的方式依次在每台服务器上执行。也就是说，任务在第一台服务器上执行完成后才会切换到下一台服务器上执行。

## 并行执行

如果你希望在多个服务器上并行执行一个任务，只需在任务声明处添加 `parallel` 选项即可：

```
    @servers(['web-1' => '192.168.1.1', 'web-2' => '192.168.1.2'])

    @task('deploy', ['on' => ['web-1', 'web-2'], 'parallel' => true])
        cd site
        git pull origin {{ $branch }}
        php artisan migrate
    @endtask
```

## 任务宏

"宏"可以让你只用一条命令就能顺序执行一组任务。例如：

```
    @servers(['web' => '192.168.1.1'])

    @macro('deploy')
        foo
        bar
    @endmacro

    @task('foo')
        echo "HELLO"
    @endtask

    @task('bar')
        echo "WORLD"
    @endtask

`deploy` 宏可以通过一条简单地命令启动并执行：

    envoy run deploy
```

## 通知

#### HipChat

任务执行完后，你可能希望发送一条通知信息到团队的 HipChat 聊天室，这一功能可以通过 `@hipchat` 指令实现：

```
    @servers(['web' => '192.168.1.1'])

    @task('foo', ['on' => 'web'])
        ls -la
    @endtask

    @after
        @hipchat('token', 'room', 'Envoy')
    @endafter
```

你还可以定制发送到 hipchat 聊天室的消息内容。任何在 `@setup` 或通过 `@include` 引入的变量都可以在消息中直接引用：

```
    @after
        @hipchat('token', 'room', 'Envoy', "$task ran on [$environment]")
    @endafter
```
Envoy 让你的团队时刻掌握服务器上任务执行的情况变得惊人的简单。

#### Slack

下面的代码实例可以将通知发送到 [Slack](https://slack.com/) 聊天室：

```
    @after
        @slack('team', 'token', 'channel')
    @endafter
```

## 更新 Envoy

通过 Composer 来更新 Envoy：

```
    composer global update
```