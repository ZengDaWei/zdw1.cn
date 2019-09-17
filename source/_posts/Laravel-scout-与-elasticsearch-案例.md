---
title: Laravel scout 与 elasticsearch 案例
date: 2019-09-17 15:01:16
tags:
  - laravel
  - elasticsearch
category: elasticsearch
---

# Laravel scout 与 elasticsearch 案例

下文将 elasticsearch 简称为 es

我们要使用 es 作为搜索引擎，首先我们需要安装它

elastic 需要 Java 8 环境，先检查 Java 环境再安装次软件

## 安装

```sh
$ wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-5.5.1.zip
$ unzip elasticsearch-5.5.1.zip
$ cd elasticsearch-5.5.1/
```

接着，进入解压后的目录，运行以下的命令，尝试启动 elastic

```sh
$ ./bin/elasticsearch
```

注意：

初次安装，非常有可能会出现以下报错：

```log
max virtual memory areas vm.maxmapcount [65530] is too low
```

运行以下命令即可解决：

```sh
$ sudo sysctl -w vm.max_map_count=262144
```

如果一切正常，elastic 默认会在本机的 `9200`端口运行，请求该端口，会获得以下信息

```json
$ curl localhost:9200

{
  "name" : "atntrTf",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "tf9250XhQ6ee4h7YI11anA",
  "version" : {
    "number" : "5.5.1",
    "build_hash" : "19c13d0",
    "build_date" : "2017-07-18T20:44:24.823Z",
    "build_snapshot" : false,
    "lucene_version" : "6.6.0"
  },
  "tagline" : "You Know, for Search"
}
```

默认情况下，elastic 只允许本机访问，如果需要远程访问权限，需要修改 elastic 安装目录的 `config/elasticserach.yml`文件，去掉`netword.host`的注释，将它的值 改为`0.0.0.0`，然后重启 elastic

```sh
network.host: 0.0.0.0
```

上面代码中，设成`0.0.0.0`让任何人都可以访问。线上服务不要这样设置，要设成具体的 IP

## 基本概念

elastic 本质上是一个分布式数据库，允许多台服务器协同工作，每台服务器可以允许多个 elastic 实例，单个 elastic 实例称为一个节点，一组节点构成一个集群

它有一些重要的概念

1. Index（索引）
2. Type（类型）
3. Document（文档）
4. Shards（分片）
5. Replicasedit（副本）

分片和副本暂时用不到，就简单的说明一下

```reStructuredText
- 副本是乘法，越多越浪费，但也越保险。
- 分片是除法，分片越多，单分片数据就越少也越分散。
```

由于里面的概念内容比较多，贴出两个讲解的非常好的博客：

1. [阮一峰的讲解](https://www.cnblogs.com/jajian/p/9976900.html)
2. [ElastSearch 的技术分析](https://www.cnblogs.com/jajian/p/9976900.html)

看完了之后，我们可以用一个对比来了解一下其中重要的概念

```js
- 关系型数据库 -> Databases(库) -> Tables(表) -> Rows(行) -> Columns(列)。
- Elasticsearch -> Indeces(索引) -> Types(类型) -> Documents(文档) -> Fields(属性)。
```

Elasticsearch 集群可以包含多个索引(indices)（数据库），每一个索引可以包含多个类型 (Types)（表），每一个类型包含多个文档(documents)（行），然后每个文档包含多个字段(Fields)（列）。

虽然这么类比，但是毕竟是两个差异化的产品，而且上面也说过在以后的版本中类型 (Types) 可能会被删除，所以一般我们创建索引都是一个种类对应一个索引。生鲜就创建商品的索引，生活用品就创建生活用品的索引，而不会说创建一个商品的索引，里面既包含生鲜的类型，又包含生活用品的类型。

## Laravel scout 与 es

先安装 scout 包

```sh
composer require laravel/scout
```

再生成配置文件

```sh
php artisan vendor:publish --provider="Laravel\Scout\ScoutServiceProvider"
```

在`config/app.php`的 provider 中，添加

```php
Laravel\Scout\ScoutServiceProvider::class,
ScoutEngines\Elasticsearch\ElasticsearchProvider::class,
```

然后我们还需要在 scout.php 中，添加 es 的配置信息，在 algolia 后添加

```php
'elasticsearch' => [
        'index' => env('ELASTICSEARCH_INDEX', 'dongdianyi'),
        'hosts' => [
            env('ELASTICSEARCH_HOST', 'http://127.0.0.1:9200'),
        ],
],
```

我们还需要使用到 GuzzleHttp

安装一下

```sh
composer require guzzlehttp/guzzle
```

开始写代码，需要先使用 command ，让 ES 初始化一些数据

```sh
php artisan make:command InitEs
```

贴出我的代码

```php
<?php

namespace App\Console\Commands;

use GuzzleHttp\Client;
use Illuminate\Console\Command;

class ESInit extends Command
{

    protected $signature = 'es:init';

    protected $description = '初始化 es库';

    public function __construct()
    {
        parent::__construct();
    }

    public function handle()
    {
        $client = new Client();
        try {
            $this->createTemplate($client);
            $this->createIndex($client);
        } catch (\Exception $ex) {
            $ex->message();
        }
    }

    public function createTemplate(Client $client)
    {
        $url = config('scout.elasticsearch.hosts')[0] . '/_template/tmp';
        // 保证 template 不存在
        // $client->delete($url);
        $param = [
            'json' => [
                'template' => '*',
                'settings' => [
                    'number_of_shards' => 1,
                ],
                'mappings' => [
                    '_default_' => [
                        '_all'              => [
                            'enabled' => true,
                        ],
                        'dynamic_templates' => [
                            [
                                'strings' => [
                                    'match_mapping_type' => 'string',
                                    'mapping'            => [
                                        'type'         => 'text',
                                        'analyzer'     => 'ik_smart',
                                        'ignore_above' => 256,
                                        'fields'       => [
                                            'keyword' => [
                                                'type' => 'keyword',
                                            ],
                                        ],
                                    ],
                                ],
                            ],
                        ],
                    ],
                ],
            ],
        ];
        $client->put($url, $param);
        $this->info('elasticsearch template created done');
    }

    public function createIndex(Client $client)
    {
        // 创建 index
        $url = config('scout.elasticsearch.hosts')[0] . '/' . config('scout.elasticsearch.index');
        // 保证 index 不存在
        // $client->delete($url);
        $param = [
            'json' => [
                'settings' => [
                    'refresh_interval'   => '5s',
                    'number_of_shards'   => 1,
                    'number_of_replicas' => 0,
                ],
                'mappings' => [
                    '_default_' => [
                        '_all' => [
                            'enabled' => false,
                        ],
                    ],
                ],
            ],
        ];
        $client->put($url, $param);
        $this->info('elasticsearch index created done');
    }
}

```

然后，我们要规定，是那个模型需要被搜索

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;
use Laravel\Scout\Searchable;


class Article extends Model
{
    use Searchable;
    protected $table = 'posts';

    protected $fillable = [
        'url',
        'author',
        'title',
        'content',
        'post_date'
    ];

    public function toSearchableArray()
    {
        return [
            'title' => $this->title,
            'content' => $this->content
        ];
    }
}
```

在模型里面，使用 Searchable 和重载 toSearchableArray 函数就可以了

然后使用命令

```sh
php artisan scout:import "App\Article"
```

将目前数据库中的数据，按照 toSearchableArray 的规则导入，导入完成就可以了

## 验证结果

es 和 scout 的步骤已经走完了，接下来就可以使用了

先定义 graphql 接口

```js
searchArticles(keyWord: String!): [Article!]! @paginate(defaultCount: 10, builder: "App\\Article@searchArticles")
```

然后再解析

```php
public function searchArticles($rootValue, array $args, GraphQLContext $context, ResolveInfo $resolveInfo)
    {
        $query = $args['keyWord'];
        return Article::search($query);
    }
```

完成！，这里例子中我使用的是 ik_smart ，会自动按照中文分词的最大粒度去匹配
