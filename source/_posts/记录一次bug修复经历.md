---
title: 记录一次bug修复经历
date: 2019-09-04 13:36:52
tags:
  - haxibiao
  - laravel
category: haxibiao
---

# 一次 修 bug 的经历

这次修的这个 bug，挺有价值

## 背景

在几天前，懂点医 app 上线了，王彬苦刷了一晚上，终于智慧点够了，可以提现了，然后点击提现按钮的时候，心态崩了，出 bug 了，提现没有成功？

## 找到问题

我听了情况和截图后，先是拿到了王彬的账号，到数据库中一顿查，发现所有关于这次提现的记录都是正常无误的，再看看提现表，发现 `status = 0 （待处理）`，看到这个记录的时候，心里答案就隐隐若现了

## 证实问题

然后把数据库拉到本地，再执行

```sh
cd /data/www/dongdianyi
art queue:work --queue=withdraws
```

马上，提现记录表中的王彬的那条记录，status = -1 （提现失败），remark = 支付宝账号和真实姓名有误

由此可证，代码是没有问题的

再排查一番

**原因：**线上队列没开，没有处理 jobs 里面的数据

## 处理问题

那我琢磨着，不可能，每次都到线上去，开队列啊，然后就问问 czg ，

得到答案：线上队列处理，用的是 supervisor，这样就可以用 类似守护线程 运行指令

然后我就去看关于 supervisor 的东西了

### 得到的知识：

> Supervisor ([http://supervisord.org](http://supervisord.org/)) 是一个用 Python 写的进程管理工具，可以很方便的用来启动、重启、关闭进程（不仅仅是 Python 进程）。除了对单个进程的控制，还可以同时启动、关闭多个进程，比如很不幸的服务器出问题导致所有应用程序都被杀死，此时可以用 supervisor 同时启动所有应用程序而不是一个一个地敲命令启动。

#### 安装 Supervisor（ubuntu）

```sh
sudo apt-get install supervisor
```

#### supervisord 配置

```sh
echo_supervisord_conf > /etc/supervisord.conf
```

#### 启动 supervisord

```sh
supervisord -c /etc/supervisord.conf
# 这个 /etc/supervisord.conf为配置文件的位置
```

#### 查看 supervisord 是否在运行：

```sh
ps aux | grep supervisord
```

#### program 配置

上面我们已经把 supervisrod 运行起来了，现在可以添加我们要管理的进程的配置文件。这些配置可以都写到 supervisord.conf 文件里，如果应用程序很多，最好通过 include 的方式把不同的程序（组）写到不同的配置文件里。

```sh
[include]
files = /etc/supervisor/*.conf
```

然后创建 `/etc/supervisor/dongdianyi.conf`

内容如下：

```php
[program:laravel-worker-dongdianyi-queue-withdraws]
command = php /data/www/dongdianyi.com/artisan queue:work --queue=withdraws --sleep=3 --tries=1 --timeout=300  ; 启动命令
autostart = true     ; 在 supervisord 启动的时候也自动启动
startsecs = 5        ; 启动 5 秒后没有异常退出，就当作已经正常启动了
autorestart = true   ; 程序异常退出后自动重启
startretries = 3     ; 启动失败自动重试次数，默认是 3
user = www          ; 用哪个用户启动
stdout_logfile = /data/www/dongdianyi.com/storage/logs/worker.log ; 日志位置
```

然后运行之后，再提现一次，去 `worker.log` 里面查看信息，发现了

```verilog
[2019-09-03 15:21:17][1360] Processing: App\Jobs\ProcessWithdraw
[2019-09-03 15:21:17][1360] Processed:  App\Jobs\ProcessWithdraw
```

啊哈，队列 work 了

王彬的钱也到账了
