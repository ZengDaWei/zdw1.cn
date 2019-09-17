---
title: queue_database
date: 2019-06-12 11:35:18
tags:
- Laravel
- Queue
categories: laravel
cover: "https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1560320784930&di=6e605216a92605f42716c912f0058f0d&imgtype=0&src=http%3A%2F%2Fb-ssl.duitang.com%2Fuploads%2Fitem%2F201611%2F30%2F20161130003250_AQmrZ.jpeg"
---

laravel_queue
<!-- more -->
# Laravel 任务队列

从现实生活中思考一个例子，**火车站买票**，我们看到的售票处前有一队长长的排队的人，这就可以理解为队列，售票员会按顺序一个个卖，这是一个例子，队列与之非常像



## 队列的配置文件

Config/queue.php ，这个文件是专用于配置队列的，默认队列驱动为 **'sync'**，如果想要更改，可以在 **.env**配置文件中，修改 `QUEUE_CONNECTION=sync` 就ok，一般会选择 database or redis 



## queue_database

将.env的配置修改为database 后，我们需要生成两个配置文件

```sh
php artisan queue:table
php artisan queue:failed-table
```

执行完成后，database/migrations/目录下会多出两个文件，为 **create_jobs_table  and create_failed_job_table**，从命名中，我们就能看出，一个为 工作表，一个为失败工作表，两个表都是为队列而生的，这样翻译有点直，但就这样理解也没啥错



## make job

然后我们的队列就可以开始干活了，前提是我们得给它指定工作干，所以，我们就先创建一个job

```sh
php artisan make:job EmailUserGoodPost // job's name random
```



这是我的代码，这里就模拟一下邮件发送

```php
<?php

namespace App\Jobs;

use App\User;
use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Bus\Dispatchable;
use Illuminate\Log\Logger;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Queue\SerializesModels;

class EmailUserNewPost implements ShouldQueue
{
    use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

    public $user;
    /**
     * Create a new job instance.
     *
     * @return void
     */
    public function __construct(User $user)
    {
        $this->user = $user;
    }

    /**
     * Execute the job.
     *
     * @return void
     */
    public function handle()
    {
        Logger::info('发送邮件给' . $this->user->name);
    }
}

```



然后我们带web.php中，这是我的代码

```php
<?php
use App\Jobs\EmailUserNewPost;
use App\User;

/*
|--------------------------------------------------------------------------
| Web Routes
|--------------------------------------------------------------------------
|
| Here is where you can register web routes for your application. These
| routes are loaded by the RouteServiceProvider within a group which
| contains the "web" middleware group. Now create something great!
|
 */

Route::get('/', function () {
    $users = User::all();
    foreach ($users as $user) {
        dispatch(new EmailUserNewPost($user));
    }
    dd('Done');
});
```

然后我们访问的时候，数据库中的jobs表就会多出几条记录，如果没有，那应该是Users表中没数据，可以使用tinker 生成一下假数据。



然后，如果我们想让队列开始工作，我们需要执行

```sh
php artisan queue:work
```

执行了这行命令，队列才会按顺序一个个处理。



## 执行失败的队列

在让队列工作的时候，可以指定她的timeout属性（秒为单位），意思是，如果这个job在指定的时间内没有完成，那么就会被归属于执行失败队列，然后默认会在90秒后被重新执行，这个90秒的设置是在 queue.php 中的，

database的 retry_after 设置，



将EmailUserNewPost.php 文件修改这样，lal是一个不存在的函数，执行会出错

```php
public function handle()
    {
        lal();
        logger('发送邮件给' . $this->user->name);
    }
```

然后我们执行命令 

```sh
php artisan job:work --timeout=80
```

指定了她的timeout，但运行后，我们发现，我们之前创建了一个failed_job表，但执行失败后，并没有到failed_job中，查询过后，发现我们需要在运行命令多加一个参数

```sh
php artisan job:work --timeout=80 --tries=5
```

tries 意思就是，执行多少遍后，还没有成功，就会被放入failed_job 中去。

我们还可以再EmailUserNewPost中，添加一个failed 函数，这样可以更好的记录失败的队列。

我这里是这样记录的

```php
public function failed(\Exception $exception)
    {
        Log::error('邮件发送失败' . $this->user->name);
    }
```



## 搞定失败的job



如果你想让failed_job中的表，全部重新执行一遍，那么就执行

```sh
php artisan queue:retry all
// 如果只想重新执行指定id的job
php artisan queue:retry id
```



如果不想重新执行，只想清空failed_job，就执行

```sh
php artisan queue:flush
```


