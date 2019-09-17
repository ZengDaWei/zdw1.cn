---
title: laravel_valet
date: 2019-06-12 17:02:48
tags:
- laravel
- haxibiao
category: laravel
cover: "https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1560340399025&di=4f6db3933fd2aa05eacbd49f7c25d476&imgtype=0&src=http%3A%2F%2Fb-ssl.duitang.com%2Fuploads%2Fitem%2F201608%2F06%2F20160806123506_LYnvP.jpeg"
---

给你瞧瞧魔法

<!-- more -->

# Laravel_Valet

今天下午在看一篇博客的时候，发现了这个神奇的东西，然后经过我的简单尝试，感觉像是发现了新大陆，真真真的好用！！！

**Real Like Magic！！**

首先，我讲一下这是个什么东西

- 它能让你的 laravel 项目瞬间与 nginx 配置到一起
- 它能让你的项目一键分享  (就是谁都可以访问，非局域网)
- 它的安装和配置，是真滴简单

[点击查看 中文文档](https://learnku.com/docs/laravel/5.8/valet/3883)



得BB两句

1. 本篇文章适合mac开发环境的哥们观看  (你要是女生就先加我**微信**)

2. mac 我默认你是安装了Homebrew的，没装你就赶紧装一个

3. 要是Linux环境，我更推荐 Homestead

## Install Valet

我们这里使用composer 来安装，记得改成中国镜像同志们

```sh
composer global require laravel/valet
```

然后安装完成后，敲一下 valet ，看看有没有帮助出来，没有就代表你没有配置到你的$PATH路径里面去。

网上一堆文章，我不是搬运工，自己搜

初始化

```sh
valet install
```

是的，它已经配置完成了，就是这么简单，兄弟我当时惊到了，怎么这么简单



## project run

然后，跑到你的laravel项目目录中，执行命令

```sh
valet park
valet open
```

然后就自动跳到浏览器打开了页面，而且还访问成功的，是不是非常之神奇



有些项目提供单个站点而不是整个目录，就执行命令

```sh
valet link
valet open
```



然后就看到了胜利的曙光，这TM快



如果你想让你的项目，还能被别人访问，那就执行

```sh
valet share
```

然后打开浏览器，按下 command + V ，没错，它已经帮你整到粘贴板上了，非常贴心


