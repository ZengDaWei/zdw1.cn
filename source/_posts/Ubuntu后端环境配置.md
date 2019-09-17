---
title: Ubuntu后端环境配置
date: 2019-07-26 12:13:23
tags:
- Ubuntu
- 后端
category: Ubuntu
cover: "https://ss3.bdstatic.com/70cFv8Sh_Q1YnxGkpoWK1HF6hhy/it/u=311231001,2044367052&fm=26&gp=0.jpg"
---
一篇详细的教程
写了如何搭建ubuntu后端环境
<!-- more -->
# Ubuntu后端环境配置

简述一下，我们需要安装的东西

1. LNMP 
2. composer
3. nvm 

## LNMP

这步最为关键，因为安装的东西都很重要，首先我们去官网下载LNMP到本地

```sh
// 先进入到home
cd ~
// 下载LNMP
wget http://soft.vpser.net/lnmp/lnmp1.6.tar.gz -cO lnmp1.6.tar.gz && tar zxf lnmp1.6.tar.gz && cd lnmp1.6 && ./install.sh lnmp
// 执行安装
sudo bash ~/lnmp1.6/install.sh
```

在安装时，它会让我们选择，MySQL，PHP版本

![img](https://lnmp.org/images/1.5/lnmp1.5-install-1.png)

Mysql 选择 5.7.*

![img](https://lnmp.org/images/1.5/lnmp1.5-install-2.png)

这是让我们设置root 密码，设置为 `localdb001`

![img](https://lnmp.org/images/1.5/lnmp1.5-install-3.png)

询问是否需要启用MySQL InnoDB，InnoDB引擎默认为开启，一般建议开启，直接回车或输入 y 

![img](https://lnmp.org/images/1.5/lnmp1.5-install-4.png)

选择 PHP 版本为 7.2.6

![img](https://lnmp.org/images/1.5/lnmp1.5-install-5.png)

可以选择不安装、Jemalloc或TCmalloc，输入对应序号回车，直接回车为默认为不安装。（一般不用安装）

![img](https://lnmp.org/images/1.5/lnmp1.5-install-success.png)

安装完成后的默认地址：

- PHP：`usr/local/php`，php.ini 位置在 `usr/local/php/etc/php.ini`
- Nginx: `usr/local/nginx`
- MySQL: `usr/local/mysql`



### 安装 VS CODE

[点击下载VScode](https://code.visualstudio.com/Download)

安装完成后，要设置VSCODE，用命令行唤醒

1. 打开VScode，按下 control + p
2. 输入 Install ‘code' command in PATH
3. 运行即可

然后就可以使用VS CODE了，其中还有非常多的插件需要安装，以便我们的开发，这步可以稍后再做

### code hosts

首先，我们需要改变 hosts 文件的权限，让他可写可读，再来添加我们需要的域名配置

```sh
sudo chmod -R 777 /etc/hosts
code /etc/hosts
```

这里的配置信息不便透露，可以通过邮件向后端开发同事询问



## 安装 composer 

执行这两行命令即可

```sh
sudo php -r "copy('https://install.phpcomposer.com/installer', 'composer-setup.php');"
sudo php composer-setup.php
```

## 安装 nvm

```sh
cd ~
wget -qO- https://raw.githubusercontent.com/creationix/nvm/v0.33.2/install.sh | bash
source ~/.bashrc
```

这样 nvm 就生效了

然后我们需要通过 nvm 安装 node

```sh
nvm install v10.16.0
```

执行完成后，就可以发现，我们node已经生效，版本为 10.16.0



### 编辑 php.ini 文件

我们刚刚安装好的，php.ini 是不能正常使用的，有些函数被禁止使用了，这步也非常简单

```sh
code /usr/local/php/etc/php.ini
```

然后我们通过 vscode 搜索（control + f）输入，disable_function ，就可以找到被禁止使用的函数，然后我们直接将，disable_function 这行 代码，全部替换

```php
disable_functions = passthru,system,chroot,chgrp,chown,popen,ini_alter,ini_restore,dl,openlog,syslog,readlink,symlink,popepassthru,stream_socket_server
```

然后保存

## 跑项目

首先，我们需要用 git clone 一个项目到本地

先配置编程目录

```sh
sudo mkdir /data/www -p
cd /data/www
```

然后，我们需要配置我们的git信息

```sh
git config --global user.name "这里写你的名字"
$ git config --global user.email 这里写你的邮箱
```

配置完成后，我们还需要配置一点免登陆得配置

```sh
code ~/.netrc
```

然后输入以下信息

```rc
machine code.datizhuanqian.cn
login 这里写代码库账号
password 这里写代码库密码
```

然后我们开始拉项目

```sh
cd /data/www
// 这里我用懂美味做例子
git clone http://code.datizhuanqian.cn/web/dongmeiwei.com.git
cd dongmeiwei.com
```

然后，我们初始化项目

```sh
cp .env.local .env
mysql -uroot -p这里写你的数据库密码
```

进入到数据库命令行界面后，需要创建数据库

```mysql
create database dongmeiwei;
\q
```

创建完成后，退出

然后执行以下几行命令，这里为了保证没问题，要先改变本地目录权限

```sh
sudo chmod -R 777 .
composer install
npm install
npm run dev
```

这几步，耗时比较长，需要等待

### 配置别名

有时候，我们经常敲的命令，太长了，为了效率，我们需要简化

```sh
code ~/.bashrc
```

然后，将以下几行配置，复制到文件中

```sh
alias sql='mysql -uroot -plocaldb001'
alias art='php artisan'
alias rna='react-native run-android'
alias .. ='cd ..'
alias gcm='git checkout master'
alias gcs='git checkout staging'
alias gcd='git checkout develop'
```

复制完成后，需要重新加载一遍

```sh
source ~/.bashrc
```



### 获取线上数据

我们跑项目，需要获取实时的数据

执行以下命令，目录名称根据自己的更换

```sh
cd /data/www/dongmeiwei.com/
sudo chmod -R +x bash/
bash bash/getsql.sh
```

执行完毕，就可以了。



## 配置nginx

首先进入到nginx配置文件目录

```sh
cd /usr/local/nginx/conf
code .
```

然后我们要改变 vhosts 这个目录的权限

```sh
sudo chmod -R 777 vhosts
```

然后需要添加一个文件，当我们的项目配置文件

```sh
cd /vhosts
touch dongmeiwei.com.conf
```

然后复制这段配置信息到里面

```nginx
server {
    listen       80;
    server_name  l.dongmeiwei.com;
    root         /data/www/dongmeiwei.com/public/;
    client_max_body_size 200m;
   
    location / {
    	try_files $uri $uri/ /index.php?$query_string;
		if (!-e $request_filename){
		    rewrite ^/(.*) /index.php last;
		}
        index  index.html index.htm index.php;
        include     /usr/local/nginx/conf/enable-php.conf;
    }
}

```

然后，再执行

```sh
sudo lnmp restart
```

然后，访问 `l.dongmeiwei.com`就可以了