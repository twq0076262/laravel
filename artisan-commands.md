# 开发

## 简介

除了 Artisan 本身提供的命令之外，您也可以为您的应用程序建立属于你自己的命令。你可以将自定义命令存放在 `app/Console/commands` 目录底下；然而，您也可以任意选择存放位置，只要您的命令能够被 `composer.json` 自动加载。

## 建立自定义命令

### 自动创建类（Class）

要创建一个新的自定义命令，您可以使用 `make:console` 这个 Artisan 命令，这将会自动产生一个 Command stub 协助您开始创建您的自定义命令：

#### 自动创建一个新的命令类

```
php artisan make:console FooCommand
```

上面的命令将会协助你自动创建一个类，并保存为文件 `app/Console/FooCommand.php`。

在创建自定义命令时，加上 --`command` 这个选项，将可以指定之后在终端机使用此自定义命令时，所要输入的自定义命令名称：

```
php artisan make:console AssignUsers --command=users:assign
```

### 撰写自定义命令

一旦你的自定义命令被创建后，你需要填写自定义命令的 `名称（name）` 与 `描述（description）`，您所填写的内容将会被显示在 Artisan 的 `list` 画面中。

当您的自定义命令被执行时，将会调用 fire 方法，您可以在此为自定义命令加入任何的逻辑判断。

### 参数与选项

你可以通过 `getArguments` 与 `getOptions` 为自定义命令自行定义任何需要的参数与选项。这两个方法都会返回一组命令数组，并由选项数组的清单所组成。

当定义 `arguments` 时，该数组值的定义分别如下：

```
array($name, $mode, $description, $defaultValue)
```

参数 `mode` 可以是下列其中一项： `InputArgument::REQUIRED` 或 `InputArgument::OPTIONAL`。

当定义 `options` 时，该数组值的定义分别如下：

```
array($name, $shortcut, $mode, $description, $defaultValue)
```

对选项而言，参数 `mode` 可以是下列其中一项：`InputOption::VALUE_REQUIRED`, `InputOption::VALUE_OPTIONAL`, `InputOption::VALUE_IS_ARRAY`, `InputOption::VALUE_NONE`。

模式为 `VALUE_IS_ARRAY` 表示调用命令时可以多次使用此选项来传入多个值：

```
php artisan foo --option=bar --option=baz
```

模式为 `VALUE_NONE` 则表示将此选项纯粹作为一种有或无的「开关」使用：

```
php artisan foo --option
```

### 取得输入值（参数与选项）

当您的自定义命令执行时，您需要让您的应用程序可以访问到这些参数和选项的值，要做到这一点，您可以使用 `argument` 和 `option` 方法：

#### 取得自定义命令被输入的参数

```
$value = $this->argument('name');
```

####取得自定义命令被输入的所有参数

```
$arguments = $this->argument();
```

#### 取得自定义命令被输入的选项

```
$value = $this->option('name');
```

#### 取得自定义命令被输入的所有选项

```
$options = $this->option();
```

### 产生输出

想要显示信息到终端屏幕上，您可以使用 `info`、`comment`、`question` 和 `error` 方法。每一种方法将会依据它所代表的目的，分别对应一种适当的 ANSI 颜色。

#### 显示一般消息到终端屏幕

```
$this->info('Display this on the screen');
```

#### 显示错误消息到终端屏幕

```
$this->error('Something went wrong!');
```

### 询问式输入

您也可以使用 `ask` 和 `confirm` 方法来提示用户进行输入：

#### 提示用户进行输入

```
$name = $this->ask('What is your name?');
```

#### 提示用户进行加密输入

```
$password = $this->secret('What is the password?');
```

#### 提示用户进行确认

```
if ($this->confirm('Do you wish to continue? [yes|no]'))
{
    //
}
```

您也可以指定一个默认值给 `confirm` 方法，可以是 `true` 或 `false`：

```
$this->confirm($question, true);
```

### 调用其它命令

有时候您可能希望在您的命令内部调用其它命令，此时您可以使用 `call` 方法：

```
$this->call('command:name', ['argument' => 'foo', '--option' => 'bar']);
```

## 注册自定义命令

### 注册一个 Artisan 命令

一旦你的自定义命令撰写完成后，你需要将它注册于 Artisan 它才能被使用。这通常位于 `app/Console/Kernel.php` 这个文件中。在此文件的 `commands` 属性，你会找到一份命令的清单。若要注册你的自定义命令，很简单的你只要将它加入清单中。当 Artisan 启动时，被列于此属性中的所有命令都将被 [service container](http://laravel.com/docs/5.0/container) 解析，并且被注册于 Artisan 。




