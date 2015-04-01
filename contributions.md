# 贡献指南

缺陷报告
核心开发讨论区
如何选取分支?
安全缺陷
代码风格

## 缺陷报告

为了促进有效积极的合作，相对于仅提交 缺陷报告 来说， Laravel 团队更鼓励使用 GitHub的 Pull Request。 当然也可以用 Pull Request 的方式发送含有失败单元测试的「缺陷报告」。

当您在呈递缺陷报告的时候，请确保您所提交的问题含有标题和清晰的描述。同时应该附带尽可能详细的与问题相关的信息和代码示例。 缺陷报告的目标是尽可能的方便您与他人去重现错误并修复它。

请谨记，建立缺陷报告是希望您与其他遇到同样问题的人一起解决这个问题。 但请不要期望其他人会主动的过来修复它。 创建缺陷报告是为了给您和他人提供一个修复问题的切入点。

Laravel 框架的源代码托管在 Github， 以下列出了每个 Laravel 相关项目仓库的连接:

Laravel Framework
Laravel Application
Laravel Documentation
Laravel Cashier
Laravel Envoy
Laravel Homestead
Laravel Homestead Build Scripts
Laravel Website
Laravel Art

## 核心开发讨论区

讨论区在 (Freenode) 上的 #laravel-dev IRC 频道， 讨论内容包括缺陷，新特性和计划实施的已有特性. Laravel 项目维护者 Taylor Otwell 通常会在周一至周五的美国芝加哥时间 8am-5am 上线 (UTC-06:00 or America/Chicago)， 当然其它时间他也会偶尔出现。

＃laravel-dev IRC 频道是对所有人开放的，欢迎任何有兴趣的朋友参与进来讨论或哪怕只是围观!


## 如何选择分支?

所有的 缺陷修正都应该提交到最后一版的稳定分支。 永远 不要把缺陷修正提交到 master 分支除非这些正是在下个发行版本中他们要修复的特性。

那些 完全向后兼容 并随当前 Laravel 版发行的 非重要 特性也许可以提交到最后一版的稳定分支。

那些在下一个 Laravel 发行版中将要出现的 重要的 新特性应该总是被提交到 master 分支。

如果您也不确定你写的特性是否重要时，请到 (Freenode) 的 #laravel-dev IRC 频道 问一下 Taylor Otwell。


## 安全缺陷

如果你在 Laravel 中发现安全缺陷，烦请以电子邮件的方式发送给 Taylor Otwell taylorotwell@gmail.com。所有的安全缺陷都将会被及时的处理掉。


## 代码风格

Laravel 框架遵循 PSR-4 和 PSR-1 代码标准。除了这些以外，如下的代码标准也应该被遵守:

类命名空间的声明必须与 `<?php` 处在同一行。

类的起始花括号 { 必须与类名处在同一行。

函数和控制结构必须使用 Allman 样式 括起来。

缩进使用制表符，对齐使用空格。