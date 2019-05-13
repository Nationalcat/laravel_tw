---
layout: post
title: scout
tag: 5.5
---
# Laravel Scout

- [介紹](#introduction)
- [安裝](#installation)
    - [隊列](#queueing)
    - [驅動需求](#driver-prerequisites)
- [設定](#configuration)
    - [設定模型索引](#configuring-model-indexes)
    - [設定可被搜尋的資料](#configuring-searchable-data)
- [索引](#indexing)
    - [批量導入](#batch-import)
    - [新增記錄](#adding-records)
    - [更新記錄](#updating-records)
    - [移除紀錄](#removing-records)
    - [暫停索引](#pausing-indexing)
- [搜尋](#searching)
    - [Where 子句](#where-clauses)
    - [分頁](#pagination)
- [自訂搜尋引擎](#custom-engines)

<a name="introduction"></a>
## 介紹

Laravel Scout 為了新增 [Eloquent 模型](/laravel_tw/docs/5.5/eloquent) 的全文搜尋而提供一個既簡單又基於驅動的解法。使用模型的 observer，Scout 會自動維持同步你的搜尋索引與 Eloquent 記錄。

目前，Scout 內建了一個 [Algolia](https://www.algolia.com/) 驅動。然而，撰寫自訂驅動並不難，你可以自由的使用自己的搜尋實作來擴充 Scout。

<a name="installation"></a>
## 安裝

首先，請透過 Composer 套件管理器來安裝 Scout：

    composer require laravel/scout

安裝 Scout 之後，你應該使用 Artisan 的 `vendor:publish` 指令來發佈 Scout 的設定。這個指令會將 `scout.php` 設定檔發佈到 `config` 目錄中：

    php artisan vendor:publish --provider="Laravel\Scout\ScoutServiceProvider"

最後是將 `Laravel\Scout\Searchable` trait 新增到你想要進行的搜尋模型中。這個 trait 會去註冊一個模型 observer 來維持模型與搜尋驅動的同步：

    <?php

    namespace App;

    use Laravel\Scout\Searchable;
    use Illuminate\Database\Eloquent\Model;

    class Post extends Model
    {
        use Searchable;
    }

<a name="queueing"></a>
### 隊列

雖然不強制要求使用 Scout，你應該在使用該函式庫之前，認真的考慮來設定[隊列驅動](/laravel_tw/docs/5.5/queues)。執行隊列器可以讓 Scout 將所有模型資訊同步到搜尋索引的操作放入隊列中，進而為你的應用程式網頁介面提供更好的回應時間。

你一旦設定了一個隊列驅動，請在 `config/scout.php` 設定檔中將 `queue` 選項設定為 `true`：

    'queue' => true,

<a name="driver-prerequisites"></a>
### 驅動需求

#### Algolia

當你使用 Algolia 驅動時，你應該在 `config/scout.php` 設定檔中設定你的 Algolia `id` 和 `secret` 憑證。一旦設定好你的憑證，你還需要透過 Composer 套件管理器來關莊 Algolia PHP SDK：

    composer require algolia/algoliasearch-client-php

<a name="configuration"></a>
## 設定

<a name="configuring-model-indexes"></a>
### 設定模型索引

每個 Eloquent 模型都與給定的搜尋「索引」同步，該索引包含了該模型所有可被搜尋的記錄。換句話說，你能將每個索引設想成 MySQL 資料表。預設每個模型會被保存到與模型的慣例命名的「資料表」。通常，這會是模型名稱的複數形式。然而，你可以在模型上覆寫 `searchableAs` 方法來自由的自訂模型的索引：

    <?php

    namespace App;

    use Laravel\Scout\Searchable;
    use Illuminate\Database\Eloquent\Model;

    class Post extends Model
    {
        use Searchable;

        /**
         * 取得模型的索引名稱。
         *
         * @return string
         */
        public function searchableAs()
        {
            return 'posts_index';
        }
    }

<a name="configuring-searchable-data"></a>
### 設定可被搜尋的資料

預設整個給定模型的 `toArray` 形式會被保存到搜尋索引中。如果你想要自訂被用來與搜尋索引同步的資料，可以去覆寫模型上的 `toSearchableArray` 方法：

    <?php

    namespace App;

    use Laravel\Scout\Searchable;
    use Illuminate\Database\Eloquent\Model;

    class Post extends Model
    {
        use Searchable;

        /**
         * 取得模型可被索引資料的陣列。
         *
         * @return array
         */
        public function toSearchableArray()
        {
            $array = $this->toArray();

            // 自訂陣列...

            return $array;
        }
    }

<a name="indexing"></a>
## 索引

<a name="batch-import"></a>
### 批量導入

如果你將 Scount 安裝在現有的專案中，你就可以準備將要用到的資料庫記錄導入搜尋驅動中。Scout 提供了一個 Artisan 的 `import` 指令，可以讓你用來將所有現有的記錄導入到搜尋所以中：

    php artisan scout:import "App\Post"

<a name="adding-records"></a>
### 新增記錄

你一旦將 `Laravel\Scout\Searchable` trait 新增到模型，你唯一要做的是 `save` 一個模型實例，並讓它自動新增到搜尋索引中。如果你已經將 Scout 設定為[要使用隊列](#queueing)，那麼這個操作將由隊列器在背景中執行：

    $order = new App\Order;

    // ...

    $order->save();

#### 透過查詢來新增

如果你想要透過 Eloquent 查詢來新增一個模型集合到搜尋索引中，你可以把 `searchable` 方法鏈結到 Eloquent 查詢上。`searchable` 方法會[分塊查詢結果](/laravel_tw/docs/5.5/eloquent#chunking-results)並新增記錄到搜尋索引上。再說一次，如果你有設定過 Scout 隊列設定，隊列器會在背景中新增所有的分塊：

    // 透過 Eloquent 查詢來新增...
    App\Order::where('price', '>', 100)->searchable();

    // 你也可以透過關聯來新增記錄...
    $user->orders()->searchable();

    // 你也可以透過控制器來新增記錄...
    $orders->searchable();

`searchable` 方法可被當作「更新或寫入」的噌做。換句話說，如果模型記錄已經在你的索引中，它就會被更新。如果不存在於搜尋索引中，則將它新增到索引中。

<a name="updating-records"></a>
### 更新記錄

要更新一個可被搜尋的模型，你只需要去更新模型實例的屬性，並將模型 `save` 到資料庫。Scout 會自動的保存你對搜尋索引的變更：

    $order = App\Order::find(1);

    // 更新訂單...

    $order->save();

你也可以在 Eloquent 查詢上使用 `searchable` 方法來更新模型的集合。如果該模型不存在於你的搜尋索引中，就建立它們：

    // 透過 Eloquent 查詢來更新...
    App\Order::where('price', '>', 100)->searchable();

    // 你也可以透過關聯去更新...
    $user->orders()->searchable();

    // 你也可以透過控制器去更新...
    $orders->searchable();

<a name="removing-records"></a>
### 移除紀錄

要從索引中移除一筆記錄，只要從資料庫中 `delete` 該模型。這個移除形式甚至相容於[軟刪除](/laravel_tw/docs/5.5/eloquent#soft-deleting)模型：

    $order = App\Order::find(1);

    $order->delete();

如果你不想要在刪除該記錄之前來接受模型，你可以在 Eloquent 查詢實例或集合上使用 `unsearchable` 方法：

    // 透過 Eloquent 查詢來移除...
    App\Order::where('price', '>', 100)->unsearchable();

    // 你也可以透過關聯去刪除You may also remove via relationships...
    $user->orders()->unsearchable();

    // 你也可以透過控制器去刪除...
    $orders->unsearchable();

<a name="pausing-indexing"></a>
### 暫停索引

有時你可能需要在模型上執行一系列的 Eloquent 操作，且不要把模型資料同步到搜尋索引中。你可以使用 `withoutSyncingToSearch` 方法來做到這一點。這個方法接受一個會立刻執行的回乎。在回呼中發生的任何模型操作都不會被同步到模型的索引：

    App\Order::withoutSyncingToSearch(function () {
        // 執行模型行為...
    });

<a name="searching"></a>
## 搜尋

你可以使用 `search` 方法開始搜尋模型。`search` 方法接受一個被用來搜尋模型的字串。然後，你應該將 `get` 方法鏈結到搜尋查詢上，並與接收到的 Eloquent 模型和給定搜尋查詢相匹配：

    $orders = App\Order::search('Star Trek')->get();

由於 Scout 搜尋會回傳一個 Eloquent 模型的集合，所以你甚至可以直接從路由或控制器中回傳結果，並自動轉換成 JSON：

    use Illuminate\Http\Request;

    Route::get('/search', function (Request $request) {
        return App\Order::search($request->search)->get();
    });

如果你想要在它們被轉換成 Eloquent 模型之前得到原生的結果，請使用 `raw()` 方法：

    $orders = App\Order::search('Star Trek')->raw();

搜尋查詢通常會在模型的 [`searchableAs`](#configuring-model-indexes) 方法指定的索引上執行。然而，你可以使用 `within` 方法來指定自訂的索引來取代原本的搜尋：

    $orders = App\Order::search('Star Trek')
        ->within('tv_shows_popularity_desc')
        ->get();

<a name="where-clauses"></a>
### Where 子句

Scout 可以讓你新增簡單的「where」子句到搜尋查詢中。目前這些方法只支援基本的數字相等式檢查，主要用於對使用者 ID 作有範圍的搜尋查詢。因為搜尋索引不是關聯資料庫，所以目前無法支援更高階的「where」子句：

    $orders = App\Order::search('Star Trek')->where('user_id', 1)->get();

<a name="pagination"></a>
### 分頁

除了接收模型集合外，你可以使用 `paginate` 方法對搜尋結果進行分頁。這個方法會回傳一個 `Paginator` 實例，就像你[平常對 Eloquent 查詢進行分頁](/laravel_tw/docs/5.5/pagination)一樣：

    $orders = App\Order::search('Star Trek')->paginate();

你可以將數量作為傳入 `paginate` 方法的第一個參數，並用來指定每個頁面需要接收模型的數量：

    $orders = App\Order::search('Star Trek')->paginate(15);

一旦你接受到結果，就可以使用 [Blade](/laravel_tw/docs/5.5/blade) 來顯示結果並渲染到頁面連結上，就像你平常使用 Eloquent 查詢一樣：

    <div class="container">
        @foreach ($orders as $order)
            {% raw %} {{ $order->price }} {% endraw %}
        @endforeach
    </div>

    {% raw %} {{ $orders->links() }} {% endraw %}

<a name="custom-engines"></a>
## 自訂搜尋引擎

#### 撰寫搜尋引擎

如果內建的 Scout 搜尋引擎沒一個符合你的需求，你可以自行定義引擎，並將它註冊到 Scout。你的引擎應該去繼承 `Laravel\Scout\Engines\Engine` 抽象類別。這個抽象類別包含了五個自訂引擎必須實作的方法：

    use Laravel\Scout\Builder;

    abstract public function update($models);
    abstract public function delete($models);
    abstract public function search(Builder $builder);
    abstract public function paginate(Builder $builder, $perPage, $page);
    abstract public function map($results, $model);

你可能會在 `Laravel\Scout\Engines\AlgoliaEngine` 類別上發現它有助於參考這些方法的實作。這個類別會提供你一個很好的起點，協助你學習如何在自己的引擎中實作這些方法。

#### 註冊引擎

一旦你撰寫了自訂的引擎，就可以使用 Scout 引擎管理器的 `extend` 方法在 Scout 上註冊它。你應該從 `AppServiceProvider` 的 `boot` 方法或任何其他被應用程式使用的服務提供者呼叫 `extend` 方法。例如，如果你撰寫了一個 `MySqlSearchEngine`，你可以想這樣註冊它：

    use Laravel\Scout\EngineManager;

    /**
     * 啟動任何應用程式服務。
     *
     * @return void
     */
    public function boot()
    {
        resolve(EngineManager::class)->extend('mysql', function () {
            return new MySqlSearchEngine;
        });
    }

一旦你的引擎被註冊，你就可以在 `config/scout.php` 設定檔中指定它做為預設的 Scout `driver`：

    'driver' => 'mysql',
