---
title: laravel版本升级
date: 2019-07-16 18:19:22
tags: 
- laravel
- haxibiao
category: laravel
cover: "https://ss1.bdstatic.com/70cFuXSh_Q1YnxGkpoWK1HF6hhy/it/u=1398815277,75276716&fm=26&gp=0.jpg"
---

Hash-fun 人员观看

<!-- more -->

# Hash-fun laravel版本升级

boss已经给我们演示过点墨阁的升级，主要步骤如下

- 先升级laravel版本到5.8，在过程中发现，如有组件包不维护了，直接弃用该包，后期再加
- 修复 agent 不 require 错误
- composer install nova2 
- 修复 lighthouse 与 旧 gql 的冲突

## 升级到laravel 5.8.*

### 快捷方式

我们需要更新后端的gql架构，就需要先更新laravel版本，这样就会涉及到一个组件包的版本兼容问题，所以在这里追求效率就直接去`code lib` 中的点墨阁复制`composer.json`和 `composer.lock` 来替代本地的，再来执行 `composer install`



### 手动升级

我们也可以一个一个更新版本，首先我们要知道哪些包是不兼容的，我们再去把这个包，拿去https://packagist.org/搜索，找到与之合适的版本，填入到composer.json中。



我推荐，使用快捷方式，毕竟坑都已经被前人填好了，直接就开始使用吧！

## 解决代码版本冲突

其实在install nova2 之前，之前的laravel版本兼容问题还是有一些的，我们需要到`config/app.php`文件中，去修改其中的`providers`数组中的内容，移除掉我们在laravel升级过程中删除掉的包，这里我贴出来那些包是我移除的

```php
// Overtrue\LaravelWechat\ServiceProvider::class,
// Collective\Remote\RemoteServiceProvider::class,
// Jenssegers\\ServiceProvider::class,
// SimpleSoftwareIO\QrCode\QrCodeServiceProvider::class,
// Bugsnag\BugsnagLaravel\BugsnagServiceProvider::class,
// YueCode\Cos\QCloudCosServiceProvider::class,
```

然后，我们只是移除了这些`providers`，有些包，我们再代码中也使用了，这样的话，我们就要去代码中去修改，比如说这里的 `Agent`，我们就需要把其中多次使用的方法抽调出来，移到 `Helpers/view.php`中，然后再这些使用到的地方，直接调用需要使用的`Agent函数`



> 这里的直接操作，是直接将通过在Helpers/view.php中添加对应的函数，然后通过VSCODE，一键替换一下。



> 这步操作，简单来说，就是包虽然删除了，但代码中还在使用这个包提供的东西，我们就要把包中我们常用的东西，抽出来，不能让这个包影响主体业务正常运行。

学的不是仅仅把代码注释了，而是在下次更新的时候，我们也知道，该如何操作了，是**思维**。

## install nova2

[nova 官方文档](https://nova.laravel.com/docs/2.0/installation.html#installing-nova)

nova，是一个非常好用，而且方便的后台管理模块

```sh
// install nova
php artisan nova:install

// 数据库 nova 的表生成
php artisan migrate
```

然后我们就可以访问nova了，默认是直接项目后面加上`/nova`就可以访问的

如果不能访问，很有可能是路由拦截，或者其他路由阻碍了nova，这个需要去`web.php`中修复一下



登录nova，nova默认是按照数据库中的`users`表来登录的，不过也可以自己编写验证规则，查阅官方文档。



## 修复old_graphql 与 lighthouse 的冲突

老的graphql，它所监听的路由是 `/graphql`，貌似还不能更改，而这个就与新的lighthouse冲突了

不过boss也已经编写好了修复的脚本，在项目目录下，执行以下命令

```sh
bash bash/fix_vender.sh
```

如果没有这个脚本，去点墨阁复制一下到本地，再提交上去

Lighthouse 的路由前缀在 `config/lighthouse.php`文件中的`prefix`

Play-ground的路由前缀在`config/graphql-playground.php`中的`route_name`

可以根据项目需要修改。


