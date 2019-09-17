---
title: queue_redis
date: 2019-06-12 14:39:28
tags:
- redis
- queue
- laravel
category: laravel
cover: "https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1560331765778&di=522481821819089ea48fca4d5b684c84&imgtype=0&src=http%3A%2F%2Fpic3.16pic.com%2F00%2F04%2F35%2F16pic_435554_b.jpg"
---

简单讲述一下redis在laravel_queue中的使用
<!-- more -->

# Queue_redis

在队列中，我们可以使用database作为驱动，可这样做会增大我们对数据库的压力，毕竟常常需要写入和删除数据，所以我们可以选择另外一个，也就是redis



## install redis

在mac中，使用 `brew install redis` ，我推荐使用包管理工具来安装，方便管理。

install 完成后，我们去修改一下redis的配置文件，路径在 /usr/local/etc/redis.conf ，我们将 daemonize 的值，改为yes，这步操作是让redis可以以守护模式启动



启动 redis 服务端

```sh
redis-server /usr/local/etc/redis.conf
```

我们启动的时候，指定一下它的配置文件



连接redis，这步需要先启动 redis-server，也就是上一步一定要执行完成

```sh
redis-cli
```

然后就连接上拉！



## install predis

在使用laravel的redis之前，要先使用composer安装 `predis`扩展包

```sh
composer require predis/predis
```



## Queue_connection_redis

在我们执行完成了之后，将 `.env`文件中的 queue_connection 的值改为 redis，还有这三个列改为对应的值

```sh
REDIS_HOST=127.0.0.1
REDIS_PASSWORD=null
REDIS_PORT=6379
```

然后我们就可以继续正常使用了，但是在使用过程中，发现 redis 不像 database 一样可以直接查看数据来判断哪些job执行成功了，那个失败了

所以，我觉得来个图形管理工具会舒服些



## Redis 监控面板 Horizon

Horizon，直译为 地平线，是laravel提供的一个队列管理工具

[点击查看 中文文档](https://learnku.com/docs/laravel/5.8/horizon/3945)

[点击查看 官方文档](https://laravel.com/docs/5.8/horizon)



安装 Horizon 扩展包

```sh
composer require laravel/horizon
```

(我发现laravel这个框架体系很庞大啊，啥都有，不太像java，java就显得啰嗦些了)



安装完成后，使用使用 `horizon:install` 发布 Artisan 命令

```sh
php artisan horizon:install
```



Horizon的配置项，就在`config/horizon.php`中，Horizon提供了三种均衡策略，simple auto 和  false，默认是simple，会将job均衡的分配给队列进程。

```php
'balance' => 'simple',
```



生成的配置信息就去翻阅文档吧，俺就不当搬运工了

## Horizon 首页图

[![VRYw7R.md.png](https://s2.ax1x.com/2019/06/12/VRYw7R.md.png)](https://imgchr.com/i/VRYw7R)







