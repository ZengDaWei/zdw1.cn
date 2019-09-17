---
title: img_qiniu
date: 2019-06-09 22:07:01
tags:
- php
- laravel
cover: "https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1560099372723&di=da43968440ea3d7062f5b70063df4b18&imgtype=0&src=http%3A%2F%2Fwww.sinaimg.cn%2Fdy%2Fslidenews%2F21_img%2F2011_23%2F18723_385929_856708.jpg"
categories: laravel
---

七牛云存储图片
<!-- more -->
# 上传图片到七牛云

七牛云是国内的一家比较好的云服务商，简单易用



## composer安装组件包

```sh
 composer require "overtrue/laravel-filesystem-qiniu"
```

想了解的可以去网址中查看用法，非常简单，几行代码搞定

## 配置到laravel

添加一行到 config/app.php中

```php
'providers' => [
    // Other service providers...
    Overtrue\LaravelFilesystem\Qiniu\QiniuStorageServiceProvider::class,
],
```



## 配置参数

添加配置信息到 filesystem.php

```php
'disks'  => [
            //...
            'qiniu' => [
                'driver'     => 'qiniu',
                'access_key' => env('QINIU_ACCESS_KEY', '你的ak'),
                'secret_key' => env('QINIU_SECRET_KEY', '你的sk'),
                'bucket'     => env('QINIU_BUCKET', '你的bucket name'),
                'domain'     => env('QINIU_DOMAIN', '你的domain'),
            ],
            //...
        ],
```



## 测试用例

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Illuminate\Support\Facades\Storage;

class UploadController extends Controller
{
    public function upload(Request $request)
    {
        $file     = $request->file('file');
        $filename = 'img/' . $file->getClientOriginalExtension();
        Storage::disk('qiniu')->writeStream($filename,fopen($file->getRealPath(),'r'));
		return 'success';
    }
}

```



上传完成后可以在七牛云中查看图片的外链，步骤就是这些，非常简单