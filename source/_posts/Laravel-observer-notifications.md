---
title: "Laravel observer, notifications"
date: 2019-08-30 16:41:42
tags:
  - php
  - laravel
categories: laravel
---

# Laravel observer, notifications

laravel 中 有很多开箱即用的功能，这三个就是很好用 并且 重要的模块

## Observer

了解过 observer 模式的哥们，应该知道，这个东西有多好用

创建 Observer，执行以下命令

```sh
php artisan make:observer LikeObserver -m Like
```

这里我加了一个参数， `-m like` ，加上了这个参数，我们就可以少些很多重复的代码

```php
<?php

namespace App\Observers;

use App\Like;
use App\Notifications\LikeArticle;

class LikeObserver
{

    public function created(Like $like)
    {
        $liked = $like->liked;
        if ($liked instanceof \App\Article) {
            $liked->user->notify(new LikeArticle($like->user, $liked));
            $liked->increment('count_likes');
        }

    }

    public function updated(Like $like)
    {

    }

    public function deleted(Like $like)
    {
        $liked = $like->liked;
        $liked->decrement('count_likes');
    }

    public function restored(Like $like)
    {

    }

    public function forceDeleted(Like $like)
    {

    }
}
```

以上，是一段很简单的代码，当创建一个 like 对象的时候，它会默认触发 到 `creared`函数，这里需要注意的一个点的，触发到 created 的时候，like 对象已经创建完成了，created 函数需要执行的代码，是与 点赞 相关的代码，比如 被点赞对象 的 `count_likes` 需要加 1，被点赞的文章的作者，也需要收到一条通知

### 注意

1. 执行到 `created` 函数时，like 对象已经创建了 （deleted 同理）
2. 我们可以在这个过程中，写 **必须执行** 的代码，以保证数据正常
3. laravel tinker 也会触发到 observer ，任何 laravel 成功的操作数据库 都会触发 observer

## notifications

消息通知，这个模块在内容型系统中会经常使用到，比如 点赞，评论，关注 ，收藏，等功能模块

创建通知，使用以下命令

```sh
php artisan make:notification CommentArticle
```

代码结构：

```php
<?php

namespace App\Notifications;

use App\Article;
use App\Comment;
use App\User;
use Illuminate\Bus\Queueable;
use Illuminate\Notifications\Notification;

class CommentArticle extends Notification
{
    use Queueable;

    private $article, $comment, $user;

    public function __construct(Article $article, Comment $comment, $user)
    {
        $this->article = $article;
        $this->comment = $comment;
        $this->user    = $user;
    }

    public function via($notifiable)
    {
        return ['database'];
    }

    public function toArray($notifiable)
    {
        return [
            'user_id'         => $this->user->id,
            'user_name'       => $this->user->name,
            'user_avatar'     => $this->user->avatar,
            'comment_content' => $this->comment->content,
            'article_title'   => $this->article->title,
            'article_id'      => $this->article->id,
        ];
    }
}

```

- private $article,$comment,\$user 这些都是可以定制的
- via 函数，声明以什么方式通知，可以有 email，数据库，等
- toArray，默认会存储到数据库中 notifications 表中的 data 字段中

我创建了一个 **评论文章**的消息通知，这个需要在 **用户对文章进行评论**时，给文章作者发通知

举例：

![img](https://user-gold-cdn.xitu.io/2019/3/27/169bd73aec2753e9?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

由此可见，消息表中，需要存放

1. 用户：头像，昵称，id
2. 文章：id，标题
3. 评论：内容，id

然后我们有要将，通知需要的数据，全部存在 data 字段中，再或者我们可以修改 notifications 表的数据结构

推荐一篇文章，将 laravel 的通知讲的很好，缺点优点很明显

[点击访问](https://juejin.im/post/5c9afee76fb9a0710466b3d4)
