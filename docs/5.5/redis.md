# Redis

- [簡介](#introduction)
    - [設定](#configuration)
    - [Predis](#predis)
    - [PhpRedis](#phpredis)
- [與 Redis 互動](#interacting-with-redis)
    - [Pipelining Commands](#pipelining-commands)
- [Pub / Sub](#pubsub)

<a name="introduction"></a>
## 簡介

[Redis](https://redis.io) 是一個開源、優異的 key-value 儲存。通常 Redis 被用來作為資料結構伺服器的一種，其中資料的 key 可以包含 [strings](https://redis.io/topics/data-types#strings)、[hashes](https://redis.io/topics/data-types#hashes)、[lists](https://redis.io/topics/data-types#lists)、[sets](https://redis.io/topics/data-types#sets)以及[sorted sets](https://redis.io/topics/data-types#sorted-sets)。

在 Laravel 使用 Redis 前，必須先使用 Composer 安裝 `predis/predis` 套件：

    composer require predis/predis

或是你可以透過 PECL 安裝 [PhpRedis](https://github.com/phpredis/phpredis) 擴充套件。這個擴充套件的安裝過程較為複雜，但是或許能滿足 Redis 使用量較高的應用程式較佳的效能需求。

<a name="configuration"></a>
### 設定

應用程式所需的 Redis 設定檔案位於 `config/database.php`，在這個檔案中，你會看到一組包含你應用程式使用 Redis 伺服器的 `redis` 陣列：

    'redis' => [

        'client' => 'predis',

        'default' => [
            'host' => env('REDIS_HOST', 'localhost'),
            'password' => env('REDIS_PASSWORD', null),
            'port' => env('REDIS_PORT', 6379),
            'database' => 0,
        ],

    ],

預設的伺服器設定檔案應足以滿足開發環境所需，不過，你仍可以輕鬆的透過修改這個陣列來符合你的執行環境。在你設定檔案內，每個被定義的 Redis 伺服器必須有一個 `name`、`host`、`port`。

#### 設定 Clusters

如果你的應用程式使用了 Redis cluster，你可以在 Redis 設定檔案設定內的 `clusters` 選項設定 cluster 相關的資訊：

    'redis' => [

        'client' => 'predis',

        'clusters' => [
            'default' => [
                [
                    'host' => env('REDIS_HOST', 'localhost'),
                    'password' => env('REDIS_PASSWORD', null),
                    'port' => env('REDIS_PORT', 6379),
                    'database' => 0,
                ],
            ],
        ],

    ],

預設的情況下，clusters 會在你的所有節點進行 client-side sharding，允許你匯集節點並建立一個大量可用的記憶體。然而，請注意在 client-side 的 sharding 操作是不會處理故障轉移的。因此，Redis cluster 主要適用於從另一個主要的資料儲存進行資料的快取。如果你想要使用原生的 Redis clustering，你可以在設定檔內使用 `options` 選項進行相關的設定。

    'redis' => [

        'client' => 'predis',

        'options' => [
            'cluster' => 'redis',
        ],

        'clusters' => [
            // ...
        ],

    ],

<a name="predis"></a>
### Predis

對於設定檔案內預設的 `host`、`port`、`database` 以及 `password` 選項，Predis 支援了額外的[連線參數](https://github.com/nrk/predis/wiki/Connection-Parameters)，這可以在 Redis 伺服器內定義。要使用這些額外的設定選項，你可以在 `config/database.php` 直接新增這些參數至你的 Redis 伺服器設定：

    'default' => [
        'host' => env('REDIS_HOST', 'localhost'),
        'password' => env('REDIS_PASSWORD', null),
        'port' => env('REDIS_PORT', 6379),
        'database' => 0,
        'read_write_timeout' => 60,
    ],

<a name="phpredis"></a>
### PhpRedis

> {note} 如果你透過 PECL 安裝 PhpRedis 擴充套件，你將需要在 `config/app.php` 設定檔內重新命名 `Redis` 的別名。

如果是使用 PhpRedis 擴充套件，你可以在 Redis 設定檔將 `client` 選項更改為 `phpredis`。這個選項可以在 `config/database.php` 設定檔中找到：

    'redis' => [

        'client' => 'phpredis',

        // Redis 其餘的設定...
    ],

此外對於預設的 `host`、`port`、`database` 以及 `password` 伺服器設定選項，PhpRedis 支援了以下額外的連線參數：`persistent`、`prefix`、`read_timeout` 以及 `timeout`，你可以在 `config/database.php` 設定檔內設定這些 Redis 伺服器相關的選項：

    'default' => [
        'host' => env('REDIS_HOST', 'localhost'),
        'password' => env('REDIS_PASSWORD', null),
        'port' => env('REDIS_PORT', 6379),
        'database' => 0,
        'read_timeout' => 60,
    ],

<a name="interacting-with-redis"></a>
## 與 Redis 互動

你可以透過呼叫多個在 `Redis` [facade](/laravel_tw/docs/5.5/facades) 的方法來與 Redis 互動。`Redis` facade 支援了動態方法，意味著呼叫任何 facade 內的 [Redis 指令](https://redis.io/commands) 會直接傳遞至 Redis。在這個範例中，我們會藉由使用在 `Redis` facade 中的 `get` 方法來執行 Redis 的 `GET` 指令：

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use Illuminate\Support\Facades\Redis;

    class UserController extends Controller
    {
        /**
         * Show the profile for the given user.
         *
         * @param  int  $id
         * @return Response
         */
        public function showProfile($id)
        {
            $user = Redis::get('user:profile:'.$id);

            return view('user.profile', ['user' => $user]);
        }
    }

當然，如上述所描述的，你可以在 `Redis` facade 上執行任何的 Redis 指令。Laravel 使用了 magic methods 傳遞這些指令至 Redis 伺服器。所以只需傳遞 Redis 命令期望的參數：

    Redis::set('name', 'Taylor');

    $values = Redis::lrange('names', 5, 10);

然而，你或許會想要使用 `command` 方法直接傳遞指令，這個方法接受使用命令的名稱作為第一個個參數，同時將值使用陣列包裝作為第二個參數：

    $values = Redis::command('lrange', ['name', 5, 10]);

#### Using Multiple Redis Connections

You may get a Redis instance by calling the `Redis::connection` method:

    $redis = Redis::connection();

This will give you an instance of the default Redis server. You may also pass the connection or cluster name to the `connection` method to get a specific server or cluster as defined in your Redis configuration:

    $redis = Redis::connection('my-connection');

<a name="pipelining-commands"></a>
### Pipelining Commands

當你要以單一操作傳送多個指令至伺服器時可以使用 Pipelining 的方式執行。`pipeline` 方法接受一個參數：一個接收 Redis 實例的 Closure。你可以將所有的命令發送到這個 Redis 實例，它們將在一個操作中被執行：

    Redis::pipeline(function ($pipe) {
        for ($i = 0; $i < 1000; $i++) {
            $pipe->set("key:$i", $i);
        }
    });

<a name="pubsub"></a>
## Pub / Sub

Laravel 提供了一個方便的介面操作 Redis 的 `publish` 及 `subscribe` 指令。這些 Redis 指令允許你在指定的「頻道」上監聽訊息。你可以從其他的應用程式，甚至使用其他的程式語言來推播訊息到頻道，讓你方便的在應用程式和進程之間溝通。

首先，必須先使用 `subscribe` 方法設定頻道監聽。因為呼叫 `subscribe` 方法是一個持續執行的進程，我們將會調用 [Artisan 命令](/laravel_tw/docs/5.5/artisan) 呼叫這個方法：

    <?php

    namespace App\Console\Commands;

    use Illuminate\Console\Command;
    use Illuminate\Support\Facades\Redis;

    class RedisSubscribe extends Command
    {
        /**
         * The name and signature of the console command.
         *
         * @var string
         */
        protected $signature = 'redis:subscribe';

        /**
         * The console command description.
         *
         * @var string
         */
        protected $description = 'Subscribe to a Redis channel';

        /**
         * Execute the console command.
         *
         * @return mixed
         */
        public function handle()
        {
            Redis::subscribe(['test-channel'], function ($message) {
                echo $message;
            });
        }
    }

現在，我們可以使用 `publish` 方法在頻道上推播訊息。

    Route::get('publish', function () {
        // Route logic...

        Redis::publish('test-channel', json_encode(['foo' => 'bar']));
    });

#### 通配字元訂閱

你可以使用 `psubscribe` 方法來訂閱全部的頻道，這在需要獲取特定匹配的頻道時是非常實用的。參數 `$channel` 會作為頻道名稱以第二參數的形式，利用 callback `Closure` 的方式傳入：

    Redis::psubscribe(['*'], function ($message, $channel) {
        echo $message;
    });

    Redis::psubscribe(['users.*'], function ($message, $channel) {
        echo $message;
    });
