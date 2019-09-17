---
title: haxibiao_backend_conf
date: 2019-06-04 14:26:07
tags:
- haxibiao
- php
- laravel
cover: "https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1559639782269&di=1608526bc594afb65827df261dd10c87&imgtype=0&src=http%3A%2F%2Fimg.9ku.com%2Fgeshoutuji%2Fsingertuji%2F3%2F31788%2F31788_5.jpg"
categories: haxibiao
---

haxibiao的开发人员看的东西
<!-- more -->

# haxibiao.com_backend_configuration

本次系统环境

- **系统：**mac
- **工作目录：**/data/www
- **PHP版本：**PHP 7.2.18 (cli) (built: May  2 2019 13:03:01) ( NTS )
- **MySQL版本：**mysql  Ver 14.14 Distrib 5.7.25, for osx10.14 (x86_64) using  EditLine wrapper
- **Nginx版本：**nginx/1.15.12
- **Git版本：** 2.20.1
- **Postgresql版本：**9.5.17
- **Composer版本：**version 1.8.5
- MAC环境下，我建议使用Homebrew安装环境 (简单易管理，好用)

## 克隆haxibiao到本地

第一步，我们需要先使用git将项目克隆到/data/www，这里需要注意，记得给www目录配置权限，如果还没有配置权限，执行命令

```sh
sudo chmod -R 777 /data/www
```

然后输入本机用户用户名即可，然后再执行以下命令，将code lib中的haxibiao克隆到/data/www

```sh
git clone http://code/web/haxibiao.com.git
```

## 配置laravel环境

执行完成后，进入haxibiao.com目录，执行以下命令，创建本地的.env文件(本机各项配置)

```sh
cp .env.local .env
```

再将MySQL和Postgresql配置信息填写上去

```php
DB_CONNECTION=pgsql     // Postgresql
DB_HOST=127.0.0.1       // ip
DB_PORT=5432       		  // 端口
DB_DATABASE=haxibiao    // 数据库
DB_USERNAME=postgres    // 用户名
DB_PASSWORD=localdb001  // 密码

MySQL_DB_HOST=127.0.0.1 // ip
MySQL_DB_PORT=3306 			// 端口
MySQL_DB_DATABASE=haxibiao //数据库
MySQL_DB_USERNAME=root     //用户名
MySQL_DB_PASSWORD=localdb001 // 密码
```

这是我本机上的配置信息，如有不同，改成自己的即可



### 安装Composer

```sh
php -r“copy（'https://getcomposer.org/installer'，'composer-setup.php'）;”

php -r“if（hash_file（'sha384'，'composer-setup.php'）==='48e3236262b34d30969dca3c37281b3b4bbe3221bda826ac6a9a62d6444cdb0dcd0615698a5cbe587c3f0fe57a54d8f5'）{echo'Installer verified';} else {echo'Installer corrupt'; unlink（'composer-setup。 php'）;} echo PHP_EOL;“

php composer-setup.php

php -r“unlink（'composer-setup.php'）;”
```

一共四行命令，按顺序，别搞错了

然后再配置中国镜像 (composer 就是php使用扩展包的工具，但是默认使用的是国外的镜像，身在中国的我们需要改一下)

执行一下名命令更改composer 镜像

```sh
composer config -g repo.packagist composer https://packagist.phpcomposer.com
```

就配置镜像好了

### 初始化项目

执行以下命令

```sh
composer install
// 这步是给项目安装所依赖的php库
npm install
npm run dev
// 前端同志这两步应该不用我BB
```



## 配置数据库

我们需要去创建haxibiao项目对应数据库

### PostgreSql

进入到自己本机的Postgresql的bin目录下，我本机上是

```sh
/usr/local/Cellar/postgresql@9.5/9.5.17/bin
```

如果你是使用brew安装的postgresql，那么路径是差不多了的，**但要注意版本名称，别进错了**

（默认情况下，使用homebrew 安装的软件都在 /usr/local/Cellar 目录下）

执行以下命令创建haxibiao数据库

```sh
./psql -U postgres
```

执行完成后，会发现进入了pgsql的命令行界面，在命令行界面，执行以下命令来创建haxibiao数据库

```sql
create database haxibiao;
```

输入 `\q` 退出pgsql的命令行界面

### MySQL

执行命令进入到MySQL命令行界面

```sh
mysql -u root -plocaldb001 
// 这里的localdb001 是我本机上的数据库密码，如果有不同，请按实际情况更改
```

进入后，执行命令创建mysql的haxibiao的数据库

```sql
create database haxibiao;
```

执行完成后，输入 `\q` 退出MySQL的命令行界面



## 数据库填充

我们目前只是创建了数据库，还没有往里面填充数据

### 创建数据表

先进入到项目目录中，执行以下命令

```sh
cd /data/www/haxibiao.com
php artisan migrate
```

执行完成后，应该会提示 success 

如果失败，这步请联系后端人员或者发我邮件



### 填充数据

先下载数据文件

https://haxibiao.com/pgsqlfiles/haxibiao.sql.zip

访问就下载了

解压后，执行以下命令

```sql
mysql -uroot -plocaldb001 -Dhaxibiao<数据库文件绝对路径

// mysql -u账号 -p密码 -D数据库名 < sql文件绝对路径
```

执行成功，就完事了，如果想要最新数据库文件，请联系大佬（XXM,CZG），因为俺暂时没权限



## haxibiao跑起来

如果你已经成功执行完了之前的所有操作，那么项目就可以成功的跑起来了



### php artisan serve

如果你想省点力气不想配置nginx了，就在项目目录中执行以下命令

```sh
php artisan serve 
// 这个可以让项目在你本地跑起来
```

如果你想你的项目能被同事访问(局域网)，执行以下命令，查看本机ip

```sh
ifconfig
```

然后记住本机的ip，再执行以下命令

```sh
php artisan serve --port 本机ip
// 举例 ：php artisan serve --port 127.0.0.1
```



然后项目就可以跑起来啦！

### Nginx

nginx相比之前的就会有点小麻烦，详细看以下步骤



首先，先进入到nginx的目录，如果你是使用brew安装nginx，那么nginx的配置文件是在 `/usr/local/etc/nginx` 里面的，然后就开始配置nginx拉！



#### 修改配置

先到`nginx`目录下的 `servers` 目录里面，去创建`haxibiao.conf`文件，往里面放置以下内容

```nginx
server {
		
  	# 你访问的域名
    server_name l.haxibiao.com;
		
  	# root 对应的是本机上haxibiao项目中的public目录，如果有路径不同的，记得修改
    root /data/www/haxibiao.com/public;

    location / {
          try_files $uri $uri/ /index.php$is_args$args;
    }
    # php-fpm下文会讲 
    include /usr/local/etc/nginx/conf.d/php-fpm;
}
```



#### Php-fpm 配置

放置之后，还没完事，要去修改一下php-fpm.conf的信息，使用brew安装，php-fpm.conf的路径是

`/usr/local/etc/php/7.2/php-fpm.conf`，然后使用编辑器打开

打开后，要修改的地方有

1. daemonize = yes  ，允许后台运行
2. error_log = /usr/local/var/log/php-fpm.log，错误日志存放地址，我的地址是这样

修改完成后去启动 php-fpm， `sudo /usr/local/Cellar/php@7.2/7.2.18/sbin/php-fpm` ,你们记得把路径改成自己对应的路径。



#### 添加Nginx php-fpm

先进入到nginx 目录，再新建一个文件夹，叫`conf.d`，如果有就不用创建了，然后往里面添加一个文件，名称叫`php-fpm`，内容是

```nginx
location ~ \.php$ {
     try_files  $uri = 404;
     fastcgi_pass 127.0.0.1:9000; # php-fpm 端口
     fastcgi_index index.php;
     fastcgi_param SCRIPT_FILENAME
     $document_root$fastcgi_script_name;
     include  fastcgi_params;
}
```

这里是php-fpm的路径对应的是`haxibiao.conf`中include的地址，如果有不同的记得修改。



如果以上步骤都做好了，就执行 `sudo nginx` 开开启nginx服务吧，然后访问 l.haxibiao.com ， 就可以看到首页了。



**nginx 常用命令**

```sh
sudo nginx -s reload // 重启nginx
sudo nginx -s stop   // 停止nginx
sudo nginx           // 开启nginx
```



更多去搜索吧