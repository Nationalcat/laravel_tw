---
layout: post
title: cache
---
# Cache

- [設定](#configuration)
    - [快取預先需求](#driver-prerequisites)
- [快取的使用](#cache-usage)
    - [取得一個快取的實例](#obtaining-a-cache-instance)
    - [從快取中取得項目](#retrieving-items-from-the-cache)
    - [存放項目到快取中](#storing-items-in-the-cache)
    - [刪除快取中的項目](#removing-items-from-the-cache)
    - [快取輔助函式](#the-cache-helper)
- [快取標籤](#cache-tags)
    - [寫入被標記的快取項目](#storing-tagged-cache-items)
    - [取得被標記的快取項目](#accessing-tagged-cache-items)
    - [刪除被標記的快取項目](#removing-tagged-cache-items)
- [加入客製化的快取驅動](#adding-custom-cache-drivers)
    - [撰寫一個驅動](#writing-the-driver)
    - [註冊快取驅動](#registering-the-driver)
- [快取事件](#events)

<a name="configuration"></a>
## 設定

Laravel 提供一個快速、統一的 API 方法給各式各樣不同的快取驅動。這些設定放置在 `config/cache.php` 當中。在這個設定檔裡面，可以自由指定你想要用哪一個來當作你應用程式的預設快取伺服器。Laravel 支援許多熱門的快取驅動，像是 [Memcached](https://memcached.org) 和 [Redis](http://redis.io)，以及其他更多的選擇。

這個快取的設定檔同時也包含了其他的選項，請確保你都有讀過這些選項的說明。Laravel 使用 `file` 作為預設快取的驅動。這個驅動儲存了序列化的快取物件在檔案系統中。建議你為大型的應用程式選用一套強勁的快取驅動，像是 Memcached 或是 Redis 等等。你可能也會想為同一個驅動設定多個設定檔。

<a name="driver-prerequisites"></a>
### 快取預先需求

#### 資料庫

當使用 `database` 這個快取驅動，你需要設置一個資料表來放置快取的內容，你可以看一下範例 Schema 如何宣告這樣的資料表：

    Schema::create('cache', function ($table) {
        $table->string('key')->unique();
        $table->text('value');
        $table->integer('expiration');
    });

> {tip} 你可以使用 `php artisan cache:table` 這個 Artisan 指令來產生一個較合適的資料庫遷移結構。 

#### Memcached

使用 Memcached 驅動需要先安裝 [Memcached PECL 套件](https://pecl.php.net/package/memcached)。 你可以在 `config/cache.php` 設定檔中列出你所有的 Memcached 伺服器。

    'memcached' => [
        [
            'host' => '127.0.0.1',
            'port' => 11211,
            'weight' => 100
        ],
    ],

你可能也會設置 host 選項到 UNIX 的 socket 路徑中，如果你有這麼做，記得 port 選項要設為 0：

    'memcached' => [
        [
            'host' => '/var/run/memcached/memcached.sock',
            'port' => 0,
            'weight' => 100
        ],
    ],

#### Redis

在選擇使用 Redis 作為 Laravel 的快取前，你需要先經由 Composer 安裝 `predis/predis` 套件（~1.0），或是經由 PECL 安裝 PhpRedis 擴充功能。

更多有關設定 Redis 的資訊，請參考 [Laravel 的文件頁面](/laravel_tw/docs/5.5/redis#configuration)。

<a name="cache-usage"></a>
## 快取的使用

<a name="obtaining-a-cache-instance"></a>
### 取得一個快取的實例

`Illuminate\Contracts\Cache\Factory` 和 `Illuminate\Contracts\Cache\Repository` [contracts](/laravel_tw/docs/5.5/contracts) 提供了存取 Laravel 快取服務的機制。而 `Factory` contract 則為你的應用程式提供了存取所有快取驅動的機制。`Repository` contract 是一般的快取驅動實作，它會依照你的快取設定檔變化。

然而，你可能也需要使用 `Cache` facade，我們會在整份文件中使用它。 `Cache` facade 提供了方便又簡潔的方法存取現行實作的 Laravel 快取 constract：

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Support\Facades\Cache;

    class UserController extends Controller
    {
        /**
         * 顯示應用程式中所有使用者的列表。
         *
         * @return Response
         */
        public function index()
        {
            $value = Cache::get('key');

            //
        }
    }

#### 存取多個快取儲存

使用 `Cache` facade，你可能會透過 `store` 存取多個快取儲存。被傳送到 `store` 的鍵，應該會對應到你的 `cache` 設定檔中的 `stores` 設定陣列的其中一項：

    $value = Cache::store('file')->get('foo');

    Cache::store('redis')->put('bar', 'baz', 10);

<a name="retrieving-items-from-the-cache"></a>
### 從快取中取得項目

在 `Cache` facade 中，`get` 方法可以用來取得快取中的項目。如果快取中沒有這個項目，那麼將會回傳 `null`。如果你希望的話，也可以傳遞第二個參數給 `get` 方法來指定找不到項目時的預設回傳值。

    $value = Cache::get('key');

    $value = Cache::get('key', 'default');

你甚至可以傳入一個`閉包`作為預設值，當指定的項目不存在快取中時，閉包將會被回傳。傳入一個閉包讓你可以延後存取資料庫，或從外部服務中取出資料作為找不到快取時的預設值：

    $value = Cache::get('key', function () {
        return DB::table(...)->get();
    });

#### 確認項目存在

`has` 方法可以用來檢查一個項目是否存在於快取中。如果項目的值是 `null` 或是 `false`，那麼將會回傳 `false`：

    if (Cache::has('key')) {
        //
    }

#### 遞增或遞減值

`increment` 和 `decrement` 方法都可以用來調整快取中的整數項目值，這兩個方法都可以選擇性的傳入第二個參數，用來指示要遞增或遞減多少：

    Cache::increment('key');
    Cache::increment('key', $amount);
    Cache::decrement('key');
    Cache::decrement('key', $amount);

#### 取出或更新

有時候，你可能會想從快取中取出一個項目，但也想在取出的項目不存在時存入一個預設值。例如，你可能會想從快取中取出所有使用者，或者當找不到使用者時，從資料庫中將這些使用者取出並放入快取中。你可以透過使用 `Cache::remember` 方法：

    $value = Cache::remember('users', $minutes, function () {
        return DB::table('users')->get();
    });

如果那個項目不存在快取中，則傳遞給 `remember` 方法的`閉包`將會被執行，而且閉包的執行結果將會被存放在快取中。

你也可以使用 `rememberForever` 方法去取得快取中的項目，並且將這個項目永遠的儲存：

    $value = Cache::rememberForever('users', function() {
        return DB::table('users')->get();
    });

#### 取出與刪除

如果你需要從快取中取出一個項目並刪除它，你可以使用 `pull` 方法。與 `get` 相似，如果物件不存在快取中，`pull` 方法將會回傳 `null`：

    $value = Cache::pull('key');

<a name="storing-items-in-the-cache"></a>
### 存放項目到快取中

你可能會在 `Cache` facade 中使用 `put` 方法來存放項目到快取中。當你將一個項目放進快取時，你需要指定「幾分鐘」給將要存放的值：

    Cache::put('key', 'value', $minutes);

如果不指定過期的分鐘數，你也可以傳遞一個 PHP 的 `DateTime` 實例來表示該快取項目過期的時間點：

    $expiresAt = Carbon::now()->addMinutes(10);

    Cache::put('key', 'value', $expiresAt);

#### 項目不存在時才新增

`add` 方法只會把還不存在快取中的項目放入快取。如果成功存放到快取，會回傳 `true`，否回傳 `false`：

    Cache::add('key', 'value', $minutes);

#### 永久地儲存項目

`forever` 方法可以用來存放永久的項目到快取中。但是因為這些項目並不會過期，所以必須被手動刪除，這可以透過 `forget` 方法達成：

    Cache::forever('key', 'value');

> {tip} 如果你是使用 Memcached 驅動，永久儲存的項目會在到達大小限制時被刪除。

<a name="removing-items-from-the-cache"></a>
### 刪除快取中的項目

你可能會使用 `forget` 方法移除在快取中的一個項目。

    Cache::forget('key');

你也可以使用 `flush` 方法來清除所有項目。

    Cache::flush();

> {note} 在清除所有快取時，將會直接清除快取中的所有項目，與你設定的任何前輟字串都沒有關係。如果你有與其他應用程式共用快取中的項目，請特別注意到這點。

<a name="the-cache-helper"></a>
### 快取輔助函式

除了使用 `Cache` facade 或是 [cache contract](/laravel_tw/docs/5.5/contracts) 之外，你還可以使用全域的 `cache` 函式來從快取中取得或是儲存資料。當 `cache` 函式被呼叫並且只有一個字串的參數時，將會傳回快取中這個項目的值：

    $value = cache('key');

如果你對這個函式提供了一組鍵和值的陣列以及過期的分鐘數，則會把這個項目存進快取中，並設定為你指定的到期時間：

    cache(['key' => 'value'], $minutes);

    cache(['key' => 'value'], Carbon::now()->addSeconds(10));

當你在測試中呼叫這個全域的 `cache` 輔助函式，你可以使用 `Cache::shouldReceive` 方法就像你在[測試一個 facade](/laravel_tw/docs/5.5/mocking#mocking-facades) 一樣。

<a name="cache-tags"></a>
## 快取標籤

> {note} 快取標籤並不支援 `file` 及 `database` 驅動。此外，當使用多個標籤以及將快取儲存成「永久」時，使用像是 memcached 這樣性能較好的驅動，可以自動清除舊的歷史記錄。

<a name="storing-tagged-cache-items"></a>
### 寫入被標記的快取項目

快取標籤允許你在快取中標記關聯的項目，並清空所有已分配指定標籤的快取值。你可以透過傳遞一組標籤名稱的有序陣列，以存取被標記的快取。舉例來說，讓我們存取一個被標記的快取並 `put` 值給它：

    Cache::tags(['people', 'artists'])->put('John', $john, $minutes);

    Cache::tags(['people', 'authors'])->put('Anne', $anne, $minutes);

<a name="accessing-tagged-cache-items"></a>
### 取得被標記的快取項目

若要取得一個被標記的快取項目，只要傳遞一樣的標籤有序列表至 `tags` 方法，然後傳遞你想取得的項目給 `get` 方法：

    $john = Cache::tags(['people', 'artists'])->get('John');

    $anne = Cache::tags(['people', 'authors'])->get('Anne');

<a name="removing-tagged-cache-items"></a>
### 刪除被標記的快取項目

你可以清空已分配單一標籤或是一組標籤列表中的所有項目。例如，下方的語法會將被標記 `people`、`authors`，或兩者的快取給移除。所以，`Anne` 與 `John` 都從快取中被移除：

    Cache::tags(['people', 'authors'])->flush();

相反的，下方的語法只會刪除被標記為 `authors` 的快取，所以 `Anne` 會被移除，但 `John` 不會：

	Cache::tags('authors')->flush();

<a name="adding-custom-cache-drivers"></a>
## 加入客製化的快取驅動

<a name="writing-the-driver"></a>
### 撰寫一個驅動

為了建立一個客製化的快取驅動，首先我們要實作一個 `Illuminate\Contracts\Cache\Store` [contract](/laravel_tw/docs/5.5/contracts)。因此，一個 `MongoDB` 快取的實作會看起來像這樣：

    <?php

    namespace App\Extensions;

    use Illuminate\Contracts\Cache\Store;

    class MongoStore implements Store
    {
        public function get($key) {}
        public function many(array $keys);
        public function put($key, $value, $minutes) {}
        public function putMany(array $values, $minutes);
        public function increment($key, $value = 1) {}
        public function decrement($key, $value = 1) {}
        public function forever($key, $value) {}
        public function forget($key) {}
        public function flush() {}
        public function getPrefix() {}
    }

我們只需要透過一個 MongoDB 的連線來實作這些方法。那麼要如何實作這些方法呢？讓我們看一下 Laravel 框架中 `Illuminate\Cache\MemcachedStore` 的原始程式碼。一旦我們完成實作之後，我們就可以接著完成註冊我們的客製化驅動：

    Cache::extend('mongo', function ($app) {
        return Cache::repository(new MongoStore);
    });

> {tip} 如果你不知道要將你的客製化快取驅動程式碼放置在何處，你可以在你的 `app` 目錄下建立一個 `Extension` 的命名空間。但是請記住，Laravel 沒有硬性規定的應用程式結構，你可以依照你的喜好任意組織你的應用程式。

<a name="registering-the-driver"></a>
### 註冊快取驅動

為了註冊一個客製化的快取驅動，我們將會使用到 `Cache` facade 的 `extend` 方法。呼叫 `Cache::extend` 的工作可以在新加入的 Laravel 應用程式中預設的 `App\Providers\AppServiceProvider` 的 boot 方法中完成，又或者你可以建立你自己的服務提供者來管理擴充功能（只是請別忘了在 `config/app.php` 中的服務提供者陣列註冊這個提供者）：

    <?php

    namespace App\Providers;

    use App\Extensions\MongoStore;
    use Illuminate\Support\Facades\Cache;
    use Illuminate\Support\ServiceProvider;

    class CacheServiceProvider extends ServiceProvider
    {
        /**
         * 執行註冊後的啟動服務。
         *
         * @return void
         */
        public function boot()
        {
            Cache::extend('mongo', function ($app) {
                return Cache::repository(new MongoStore);
            });
        }

        /**
         * 在容器中註冊綁定。
         *
         * @return void
         */
        public function register()
        {
            //
        }
    }

第一個傳遞給 `extend` 方法的參數是驅動的名稱，這個名稱要與你在 `config/cache.php` 設定檔中，`driver` 選項指定的名稱相同，第二個參數是一個應該要回傳一個 `Illuminate\Cache\Repository` 實例的閉包，這個閉包會被傳入一個 `$app` 實例，這個實例是屬於[服務容器](/laravel_tw/docs/5.5/container)。

一旦你的擴充功能完成，你只需要簡單的更新 `config/cache.php` 設定檔中的 `driver` 選項為你的擴充功能名稱即可。

<a name="events"></a>
## 快取事件

如果要在每次操作快取時執行一段程式碼，你可以監聽由快取所觸發的[事件](/laravel_tw/docs/5.5/events)。一般來說，你必須將事件監聽器放置在 `EventServiceProvider` 的 `boot` 方法中：

    /**
     * 應用程式的事件監聽器列表。
     *
     * @var array
     */
    protected $listen = [
        'Illuminate\Cache\Events\CacheHit' => [
            'App\Listeners\LogCacheHit',
        ],

        'Illuminate\Cache\Events\CacheMissed' => [
            'App\Listeners\LogCacheMissed',
        ],

        'Illuminate\Cache\Events\KeyForgotten' => [
            'App\Listeners\LogKeyForgotten',
        ],

        'Illuminate\Cache\Events\KeyWritten' => [
            'App\Listeners\LogKeyWritten',
        ],
    ];
