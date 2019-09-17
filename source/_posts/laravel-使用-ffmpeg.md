---
title: laravel 使用 ffmpeg
date: 2019-09-06 14:08:18
tags:
  - laravel
  - FFMpeg
category: laravel
---

# Laravel 使用 FFMpeg

心血来潮，想玩玩 laravle 使用 FFMpeg

## 安装 FFMpeg （Ubuntu）

```sh
sudo apt-get install ffmpeg
```

执行一下，等待完成

安装完成后，ffmpeg 和 ffprobe 位置通常在 `/usr/bin` 目录下，如果不在，执行 `which ffmpeg` 获取位置信息

## 使用集成包 PHP-FFMpeg

[GitHub 地址](https://github.com/PHP-FFMpeg/PHP-FFMpeg)

引入到项目

```sh
composer require php-ffmpeg/php-ffmpeg
```

引入完成，它需要制定 两个配置文件信息，以便我们正常使用，也就是上文所讲的 ffmpeg 和 ffprobe

全局配置

到 `AppServiceProvider.php` 中添加代码

```php
	public function boot()
    {
        $this->registerSingleObject();
    }
    public function registerSingleObject()
    {
        // register ffmpeg and ffprobe
        $this->app->singleton('ffmpeg', function ($app) {
            return \FFMpeg\FFMpeg::create([
                'ffmpeg.binaries'  => [
                    exec('which ffmpeg'),
                ],
                'ffprobe.binaries' => [
                    exec('which ffprobe'),
                ],
            ]);
        });
        $this->app->singleton('ffprobe', function ($app) {
            return \FFMpeg\FFProbe::create([
                'ffprobe.binaries' => [
                    exec('which ffprobe'),
                ],
            ]);
        });
    }
```

使用单例模式获取 `FFMpeg` 和 `FFProbe` 对象，其中 `exec('which ffmpeg')` 是获取 程序位置信息，以便创建类

## 基础封装

举例：

- 视频的第一秒为封面
- 获取视频基础信息

```php
<?php

namespace App\Helpers;

use FFMpeg\Coordinate\TimeCode;
use Illuminate\Support\Str;

class FFMpegUtil
{

    // 获取视频信息
    public static function getVideoInfo($streamPath)
    {
        $ffprobe = app('ffprobe');
        $stream  = $ffprobe->streams($streamPath)->videos()->first();
        return $stream ? $stream->all() : [];
    }

    // 截取
    public static function getCover($streamPath, $fromSecond)
    {
        $ffmpeg   = app('ffmpeg');
        $video    = $ffmpeg->open($streamPath);
        $frame    = $video->frame(TimeCode::fromSeconds($fromSecond)); //提取第几秒的图像
        $fileName = 'video/' . Str::random(12) . '.jpg';
        if (!is_dir(storage_path("video"))) {
            mkdir(storage_path("video"), 0777);
        }
        $frame->save(storage_path($fileName));
        return $fileName;
    }
}

```

## 业务使用

接受 Request 对象传入的 视频 为例子

```php
public function saveVideotoQiniu($file)
    {
        Auth::loginUsingId(1);
        if ($user = getUser()) {

            // 1.判断是否存在此视频
            $path  = $file->getRealPath();
            $hash  = md5_file($path);
            $video = Video::firstOrNew(['json->hash' => $hash]);
            if ($video->id) {
                $video->touch();
                return $video;
            }

            // 2.保存到 云
            $cdn_path = $this->saveFile($file);
            $db_path  = getPath($cdn_path);

            // 3.获取截图
            $fileName = FFMpegUtil::getCover($path, 1);
            $image    = $this->saveImage(new UploadedFile(storage_path($fileName), 'file.jpg'));

            //4.设置视频信息
            $data     = [];
            $data     = FFMpegUtil::getVideoInfo($path);
            $duration = array_get($data, 'duration');
            $duration = $duration > 0 ? ceil($duration) : $duration;

            $video->path    = $db_path;
            $video->user_id = $user->id;
            $video->setJsonData('width', array_get($data, 'width'));
            $video->setJsonData('height', array_get($data, 'height'));
            $video->duration = $duration;
            $video->setJsonData('cover', $image->path);
            $video->save();
        }
    }
```

例子中的 `saveImage` 是将图片上传到 云端的函数，返回上传后的图片 url

到此，文章完毕
