# Laravel 安装指南

安装 Composer
安装 Laravel
环境需求

## 安装 Composer

Laravel 框架使用 Composer 来管理其依赖性。所以，在你使用 Laravel 之前，你必须确认在你电脑上是否安装了 Composer。


## 安装 Laravel

通过 Laravel 安装工具

首先，使用 Composer 下载 Laravel 安装包：

```
composer global require "laravel/installer=~1.1"
```
请确定把 `~/.composer/vendor/bin` 路径放置于您的 `PATH `里， 这样 `laravel `执行文件就会存在你的系统。

一旦安装完成后，就可以使用 `laravel new `命令建立一份全新安装的 `Laravel` 应用，例如： `laravel new blog` 将会在当前目录下建立一个名为 `blog` 的目录， 此目录里面存放着全新安装的 Laravel 相关代码，此方法跟其他方法不一样的地方在于会提前安装好所有相关代码，不需要再通过 `composer install` 安装相关依赖，速度会快许多。

```
laravel new blog
```
### 通过 Composer Create-Project

你一样可以通过 Composer 在命令行执行 `create-project` 来安装 Laravel：

```
composer create-project laravel/laravel --prefer-dist
```
### 脚手架

Laravel 自带了用户注册和认证的脚手架。如果你想要移除这个脚手架，使用 fresh 命令即可：

php artisan fresh


## 环境需求

Laravel 框架有一些系统上的需求：

- PHP 版本 >= 5.4
- Mcrypt PHP 扩展
- OpenSSL PHP 扩展
- Mbstring PHP 扩展
- Tokenizer PHP 扩展

在 PHP 5.5 之后， 有些操作系统需要手动安装 PHP JSON 扩展包。如果你是使用 Ubuntu，可以通过 `apt-get install php5-json` 来进行安装。


## 配置

在你安装完 Laravel 后，首先需要做的事情是配置一个随机字串作为应用程序密钥。假设你是通过 comoser 安装 Laravel ，这个密钥会自动通过 `key:generate `命令帮你配置完成。

通常这个密钥应该有 32 字符长。这个密钥可以被配置在 `.env` 环境文件中。 **如果这密钥没有被配置的话，你的用户 **sessions **和其他的加密数据都是不安全的！**

Laravel 几乎不需配置就可以马上使用。你可以自由的开始开发！然而，你可以查看 `config/app.php` 文件和其他的文档。你可能希望根据你的应用程序而做更改，文件包含数个选项如 `时区` 和 `语言环境`。

一旦 Laravel 安装完成，你应该同时 配置本地环境。

```
注意： 你不应该在正式环境中将 app.debug 配置为 true。绝对！千万不要！
```

权限

Laravel 框架有一个目录需要额外配置权限：`storage` 要让服务器有写入的权限。


## 优雅链接

### Apache

Laravel 框架通过 `public/.htaccess` 文件来让网址中不需要 `index.php`。如果你的网页服务器是使用 Apache 的话，请确认是否有开启 `mod_rewrite `模块。

假设 Laravel 附带的 `.htaccess `文件在 Apache 无法生效的话，请尝试下面的方法：

```
Options +FollowSymLinks
RewriteEngine On

RewriteCond %{REQUEST_FILENAME} !-d
RewriteCond %{REQUEST_FILENAME} !-f
RewriteRule ^ index.php [L]
```
### Nginx

在 Nginx，在你的网站配置中增加下面的配置，可以使用「优雅链接」：

```
location / {
    try_files $uri $uri/ /index.php?$query_string;
}
```
当然，如果你使用 Homestead 的话，优雅链接会自动的帮你配置完成。