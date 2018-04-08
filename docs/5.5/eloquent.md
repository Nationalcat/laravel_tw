---
layout: post
title: eloquent
tag: 5.5
---
# Eloquent：入門

- [介紹](#introduction)
- [定義模型](#defining-models)
    - [Eloquent 模型慣例](#eloquent-model-conventions)
- [取得模型](#retrieving-models)
    - [集合](#collections)
    - [分塊結果](#chunking-results)
- [取得單一模型或 Aggregate](#retrieving-single-models)
    - [取得 Aggregate](#retrieving-aggregates)
- [插入與更新模型](#inserting-and-updating-models)
    - [插入](#inserts)
    - [更新](#updates)
    - [批量賦值](#mass-assignment)
    - [其他建立方法](#other-creation-methods)
- [刪除模型](#deleting-models)
    - [軟刪除](#soft-deleting)
    - [查詢被軟刪除的模型](#querying-soft-deleted-models)
- [查詢 Scope](#query-scopes)
    - [全域的 Scope](#global-scopes)
    - [局部的 Scope](#local-scopes)
- [事件](#events)
    - [Observer](#observers)

<a name="introduction"></a>
## 介紹

Laravel 的 Eloquent ORM 提供了漂亮、簡潔的 ActiveRecord 實作來和資料庫互動。每個資料庫表有一個對應的「模型」可以用來跟資料表互動。你可以透過模型查詢資料表內的資料，以及新增記錄到資料表中。

在開始之前，一定要在 `config/database.php` 中設定一個資料庫連接。更多資料庫的設定資訊，請查看[資料庫設定](/laravel_tw/docs/5.5/database#configuration)。

<a name="defining-models"></a>
## 定義模型

開始之前，讓我們先建立一個 Eloquent 模型。模型通常放在 `app` 目錄，不過你可以自由地把他們放在任何可以透過你的 `composer.json` 自動載入的地方。所有的 Eloquent 模型都繼承 `Illuminate\Database\Eloquent\Model` 類別。

建立模型實例的最簡單的方法是使用 [Artisan 指令](/laravel_tw/docs/5.5/artisan)的 `make:model`：

    php artisan make:model User

假設當你產生一個模型時，想要產生一個[資料庫遷移](/laravel_tw/docs/5.5/schema#database-migrations)，可以使用 `--migration` 或 `-m` 選項：

    php artisan make:model User --migration

    php artisan make:model User -m

<a name="eloquent-model-conventions"></a>
### Eloquent 模型慣例

現在，讓我們來看一個 `Flight` 模型的範例，我們將會用它來從 `flights` 資料表取回與儲存資訊：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Flight extends Model
    {
        //
    }

#### 資料表名稱

請注意，我們並沒有告訴 Eloquent `Flight` 模型該使用哪一個資料表。依照慣例，除非明確地指定其他名稱，不然類別的小寫、底線、複數形式會拿來當作資料表的表單名稱。所以，這個案例中，Eloquent 將會假設 `Flight` 模型儲存記錄在 `flights` 資料表。你可以在模型上定義一個 `table` 屬性，用來指定自訂的資料表：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Flight extends Model
    {
        /**
         * 與模型關聯的資料表。
         *
         * @var string
         */
        protected $table = 'my_flights';
    }

#### 主鍵

Eloquent 也會假設每個資料表有一個主鍵欄位叫做 `id`。你可以定義一個 `$primaryKey` 屬性來覆寫這個慣例。

此外，Eloquent 會假設主鍵是一個遞增的整數值，這表示預設的主鍵位自動轉換成 `int`。如果你希望使用非遞增或非數字的主鍵，務必把模型上的 `public $incrementing` 屬性設定為 `false`。如果你的主鍵不是一個整數，你應該將模型上的 `protected $keyType` 屬性設定為 `string`。

#### 時間戳記

預設的 Eloquent 會預期你的資料表會有 `created_at` 和 `updated_at` 欄位。如果你不希望 Eloquent 自動管理這些欄位，請將模型上的 `$timestamps` 屬性設定為 `false`：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Flight extends Model
    {
        /**
         * 說明模型是否應該被戳記時間。
         *
         * @var bool
         */
        public $timestamps = false;
    }

如果你需要客製化你的時間戳記格式，在你的模型內設定 `$dateFormat` 屬性。這個屬性決定日期如何在資料庫中儲存，以及當模型被序列化成陣列或是 JSON 時的格式：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Flight extends Model
    {
        /**
         * 模型的日期欄位儲存格式。
         *
         * @var string
         */
        protected $dateFormat = 'U';
    }

如果你需要自訂欄位名稱，並用來儲存時間戳記，你可以在模型中設定 `CREATED_AT` 和 `UPDATED_AT` 常數：

    <?php

    class Flight extends Model
    {
        const CREATED_AT = 'creation_date';
        const UPDATED_AT = 'last_update';
    }

#### 資料庫連線

預設所有 Eloquent 模型會使用應用程式預設的資料庫連線設定。如果你想為模型指定不同的連線，請使用 `$connection` 屬性：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Flight extends Model
    {
        /**
         * 為模型選擇連線名稱。
         *
         * @var string
         */
        protected $connection = 'connection-name';
    }

<a name="retrieving-models"></a>
## 取得模型

一旦你建立了一個模型並且將模型[關聯到資料表](/laravel_tw/docs/5.5/schema)，你就可以從資料庫中取得資料。把每個 Eloquent 模型想像成強大的[查詢建構器](/laravel_tw/docs/5.5/queries)，讓你可以流暢地查詢與模型關聯的資料表。例如：

    <?php

    use App\Flight;

    $flights = App\Flight::all();

    foreach ($flights as $flight) {
        echo $flight->name;
    }

#### 新增額外的限制

Eloquent 的 `all` 方法會回傳在模型資料表中所有的結果。由於每個 Eloquent 模型可以當作一個[查詢建構器](/laravel_tw/docs/5.5/queries)，所以你可以在查詢中增加規則，然後透過 `get` 方法來取得結果：

    $flights = App\Flight::where('active', 1)
                   ->orderBy('name', 'desc')
                   ->take(10)
                   ->get();

> {tip} 由於 Eloquent 模型是查詢建構器，應該檢閱所有[查詢建構器](/laravel_tw/docs/5.5/queries)可用的方法。你可以在你的 Eloquent 查詢中使用這其中的任何方法。

<a name="collections"></a>
### 集合

像是 `all` 和 `get` 能夠取得多個結果的 Eloquent 方法，會回傳 `Illuminate\Database\Eloquent\Collection` 實例。`Collection` 類別為處理你的 Eloquent 結果提供[各種有用的方法](/laravel_tw/docs/5.5/eloquent-collections#available-methods)：

    $flights = $flights->reject(function ($flight) {
        return $flight->cancelled;
    });

當然，你也可以像陣列一樣簡單地遍歷集合：

    foreach ($flights as $flight) {
        echo $flight->name;
    }

<a name="chunking-results"></a>
### 分塊結果

如果你需要處理上千筆 Eloquent 查詢結果，可以使用 `chunk` 命令。`chunk` 方法將會取得一個 Eloquent 模型的「分塊」，將它們送到給定的 `閉包 (Closure)` 進行處理。當你在處理大量的結果時，使用 `chunk` 方法可以節省記憶體：

    Flight::chunk(200, function ($flights) {
        foreach ($flights as $flight) {
            //
        }
    });

傳遞到方法的第一個參數是表示你希望每次「分塊」要接收的資料數量。閉包則作為第二個參數傳遞，它將會在每次從資料取出分塊時被呼叫。資料庫查詢會將執行接收到的每個記錄塊傳入閉包中。

#### 使用指標

`cursor` 方法可以讓你使用指標來搜索資料庫記錄，並只會執行一次查詢。在處理大量資料時，使用 `cursor` 方法可以大幅減少記憶體的使用量：

    foreach (Flight::where('foo', 'bar')->cursor() as $flight) {
        //
    }

<a name="retrieving-single-models"></a>
## 取得單一模型或 Aggregate

當然，除了取得給定資料表的所有記錄，你還可以使用 `find` 或 `first` 來取得單一記錄。這些方法會回傳單一模型實例，而非回傳模型的集合：

    // 透過主鍵取得模型...
    $flight = App\Flight::find(1);

    // 取得符合查詢條件的第一個模型...
    $flight = App\Flight::where('active', 1)->first();

你也可以使用主鍵陣列作為參數來呼叫 `find` 方法，這會回傳符合記錄的集合：

    $flights = App\Flight::find([1, 2, 3]);

#### Not Found 拋出例外

有時你可能希望因為找不到模型而拋出例外。這在路由或控制器中特別有用。`findOrFail` 和 `firstOrFail` 方法會取得查詢的第一個結果。然而，如果未能找到結果，`Illuminate\Database\Eloquent\ModelNotFoundException` 會拋出例外：

    $model = App\Flight::findOrFail(1);

    $model = App\Flight::where('legs', '>', 100)->firstOrFail();

如果沒有捕獲例外，則會自動發送 `404` HTTP 會應給使用者。在使用這些方法時，不必仔細撰寫檢查要回傳的 `404` 回應：

    Route::get('/api/flights/{id}', function ($id) {
        return App\Flight::findOrFail($id);
    });

<a name="retrieving-aggregates"></a>
### 取得 Aggregate

你也可以使用[查詢建構器](/laravel_tw/docs/5.5/queries)提供的 `count`、`sum`、`max` 和其他 [aggregate 方法](/laravel_tw/docs/5.5/queries#aggregates)。這些方法會回傳確切的純量值，而不是完整的模型實例：

    $count = App\Flight::where('active', 1)->count();

    $max = App\Flight::where('active', 1)->max('price');

<a name="inserting-and-updating-models"></a>
## 插入與更新模型

<a name="inserts"></a>
### 插入

要在資料庫建立一筆新紀錄，只要建立一個新模型實例，並在模型上設定屬性，接著呼叫 `save` 方法：

    <?php

    namespace App\Http\Controllers;

    use App\Flight;
    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;

    class FlightController extends Controller
    {
        /**
         * 建立新 flight 實例。
         *
         * @param  Request  $request
         * @return Response
         */
        public function store(Request $request)
        {
            // 驗證請求...

            $flight = new Flight;

            $flight->name = $request->name;

            $flight->save();
        }
    }

在這個範例中，我們把進來的 HTTP 請求的 `name` 參數簡單地指定給 `App\Flight` 模型實例的 `name` 屬性。當我們呼叫 `save` 方法，就會新增一筆記錄到資料庫中。當 `save` 方法被呼叫時，`created_at` 以及 `updated_at` 時間戳記將會自動被設定，所以不需要手動去設定它們。

<a name="updates"></a>
### 更新

`save` 方法也可以用於更新資料庫中已經存在的模型。要更新模型，你必須先取回模型，設定任何你希望更新的屬性，接著呼叫 `save` 方法。同樣的，`updated_at` 時間戳記將會自動被更新，所以不需要手動設定它的值：

    $flight = App\Flight::find(1);

    $flight->name = 'New Flight Name';

    $flight->save();

#### 批量更新

也可以針對符合給定查詢的任意數量模型執行更新。在這個範例中，所有 `active` 並且 `destination` 為 `San Diego` 的航班，將會被標記為延遲：

    App\Flight::where('active', 1)
              ->where('destination', 'San Diego')
              ->update(['delayed' => 1]);

`update` 方法預期收到一個欄位與值成對的陣列，來代表應該被更新的欄位。

> {note} 透過 Eloquent 來批量更新時，更新後的模型將不觸發 `saved` 和 `updated` 模型事件。這是因為發出批量更新時，實際上並不會取得模型。

<a name="mass-assignment"></a>
### 批量賦值

你也可以在使用 `create` 方法來儲存一個新的模型。被插入的模型實例會從該方法回傳給你。然而，在這之前，你將需要在你的模型上指定一個 `fillable` 或是 `guarded` 屬性，所有的 Eloquent 模型預設會針對批量賦值作保護。

當使用者透過請求傳入一個非預期的 HTTP 參數時，該參數會非預期竄改資料庫中的欄位，發生批量賦值的漏洞。例如，一個惡意的使用者透過 HTTP 請求發送一個 `is_admin` 參數，接著被傳送到你模型的 `create` 方法，讓使用者可以把自己提升為一位管理員。

所以，你應該開始定義想要被批量賦值的模型屬性。你可以在模型上使用 `$fillable` 屬性來達到這點。例如讓我們實際去做 `Flight` 模型的 `name` 屬性的批量賦值：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Flight extends Model
    {
        /**
         * 可被批量賦值的屬性。
         *
         * @var array
         */
        protected $fillable = ['name'];
    }

一旦我們已經設定屬性為可以被批量賦值的，我們可以使用 `create` 方法來新增一筆新記錄到資料庫。`create` 方法回傳已經被儲存的模型實例：

    $flight = App\Flight::create(['name' => 'Flight 10']);

如果你已經有一個模型實例，你可以使用 `fill` 方法來填充屬性陣列：

    $flight->fill(['name' => 'Flight 22']);

#### 屬性的白名單

`$fillable` 作為一個可以被批量賦值的屬性的「白名單」，然而你也可以選擇使用 `$guarded`。`$guarded` 屬性應該包含一個屬性的陣列，是你不想要被批量賦值的。所有不在陣列裡面的其他屬性將會是可以被批量賦值的。所以，`$guarded` 的功能像是一個「黑名單」。當然，你應該使用 `$fillable` 或 `$guarded` - 而不是兩者。在下列範例中，**除了 `price`** 的所有屬性都可被批量賦值：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Flight extends Model
    {
        /**
         * 不可被批量賦值的屬性。
         *
         * @var array
         */
        protected $guarded = ['price'];
    }

如果你想要所有屬性都能被批量賦值，你可以定義 `$guarded` 屬性為空陣列：

    /**
     * 不可被批量賦值的屬性。
     *
     * @var array
     */
    protected $guarded = [];

<a name="other-creation-methods"></a>
### 其他建立方法

#### `firstOrCreate` 和 `firstOrNew`

你還可以使用其他兩種方法透過批量賦值屬性來建立模型：`firstOrCreate` 和 `firstOrNew`。`firstOrCreate` 方法會嘗試使用給定的欄位與值來找資料庫記錄。如果在資料庫中無法找到該模型，會從屬性的第一個參數以及那些可選的參數第二個參數中插入一筆記錄。

`firstOrNew` 方法類似 `firstOrCreate`，會嘗試在資料庫查詢符合給定屬性的記錄。然而，如果沒找到模型，將會回傳一個新模型。該注意 `firstOrNew` 回傳的模型還未存到資料庫，你還需要手動呼叫 `save` 來儲存它：

    // 依名稱取得航班，或因為不存在而建立它...
    $flight = App\Flight::firstOrCreate(['name' => 'Flight 10']);

    // 依名稱取得航班，或建立該名稱與延遲的屬性...
    $flight = App\Flight::firstOrCreate(
        ['name' => 'Flight 10'], ['delayed' => 1]
    );

    // 依名稱取得航班，或實例...
    $flight = App\Flight::firstOrNew(['name' => 'Flight 10']);

    // 依名稱取得航班，或實例的名稱和延遲的屬性...
    $flight = App\Flight::firstOrNew(
        ['name' => 'Flight 10'], ['delayed' => 1]
    );

#### `updateOrCreate`

你還可能遇到想要去更新已存在的模型或者如果模型不存在去新增的情況。Laravel 提供 `updateOrCreate` 方法，只需要一步即可做到。`updateOrCreate` 類似 `firstOrCreate` 方法儲存模型，所以這裡不需要呼叫 `save()`：

    // 如果是從奧克蘭到聖地牙哥的航班，將價格訂為 99 美金。
    // 如果沒有符合的模型，就建立一個。
    $flight = App\Flight::updateOrCreate(
        ['departure' => 'Oakland', 'destination' => 'San Diego'],
        ['price' => 99]
    );

<a name="deleting-models"></a>
## 刪除模型

在模型實例上呼叫 `delete` 方法來刪除一個模型：

    $flight = App\Flight::find(1);

    $flight->delete();

#### 透過主鍵刪除存在的模型

在上述範例中，我們會在呼叫 `delete` 方法前，從資料庫取得模型。然而，如果你知道該模型的主鍵，你可以沒有取得模型就刪除它。呼叫 `destroy` 方法來達成：

    App\Flight::destroy(1);

    App\Flight::destroy([1, 2, 3]);

    App\Flight::destroy(1, 2, 3);

#### 透過查詢來刪除模型

當然，你也可以在一組模型上執行刪除的語法。在這個範例中，我們會刪除所有被標記無效的航班。批量刪除類似批量更新，不會觸發刪除模型的任何模型事件：

    $deletedRows = App\Flight::where('active', 0)->delete();

> {note} 透過 Eloquent 執行批量刪除語法，`deleting` 和 `deleted` 模型不會觸發刪除模型的事件。這是因為在執行刪除語法時，並未實際取得模型。

<a name="soft-deleting"></a>
### 軟刪除

除了從資料庫確實的移除記錄，Eloquent 也能「軟刪除」模型。當模型被軟刪除時，它們不會真的從你的資料庫中移除，`deleted_at` 屬性是在模型上設定並寫入到資料庫。如果模型有不能是 null 的 `deleted_at` 值，該模型將會被軟刪除。要為模型啟用軟刪除，請在模型上使用 `Illuminate\Database\Eloquent\SoftDeletes` trait 並新增 `deleted_at` 欄位到你的 `$dates` 屬性：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;
    use Illuminate\Database\Eloquent\SoftDeletes;

    class Flight extends Model
    {
        use SoftDeletes;

        /**
         * 該屬性會變更為日期。
         *
         * @var array
         */
        protected $dates = ['deleted_at'];
    }

當然，你應該新增 `deleted_at` 欄位到你的資料表。Laravel [schema 建構器](/laravel_tw/docs/5.5/migrations)具有一個輔助函式來建立這個欄位：

    Schema::table('flights', function ($table) {
        $table->softDeletes();
    });

現在，當你在模型上呼叫 `delete` 方法時，`deleted_at` 欄位會設定當前日期與時間。還有，查詢有使用軟刪除的模型時，被軟刪除的模型會自動從所有查詢結果中排除。

如果要確定給定的模型實例是否被軟刪除，請使用 `trashed` 方法：

    if ($flight->trashed()) {
        //
    }

<a name="querying-soft-deleted-models"></a>
### 查詢被軟刪除的模型

#### 包含被軟刪除的模型

如上所述，被軟刪除的模型會自動從查詢結果中排除。然而，你可以在查詢上使用 `withTrashed` 方法，強制被軟刪除的模型出現在結果中：

    $flights = App\Flight::withTrashed()
                    ->where('account_id', 1)
                    ->get();

`withTrashed` 方法也可以被用在 [Eloquent 的關聯](/laravel_tw/docs/5.5/eloquent-relationships)查詢上：

    $flight->history()->withTrashed()->get();

#### 只取得被軟刪除的模型

`onlyTrashed` 方法會**只有**取得被軟刪除的模型：

    $flights = App\Flight::onlyTrashed()
                    ->where('airline_id', 1)
                    ->get();

#### 恢復被軟刪除的模型

有時你可能希望「取消刪除」被軟刪除的模型。要把被軟刪除的資料恢復到一般狀態，請在模型實例上使用 `restore` 方法：

    $flight->restore();

你也可以在查詢中使用 `restore` 方法來快速恢復多筆資料。就像其他「批量」操作，這不會在恢復資料的時候觸發任何模型實例：

    App\Flight::withTrashed()
            ->where('airline_id', 1)
            ->restore();

像是 `withTrashed` 方法，`restore` 方法也可被用在 [Eloquent 關聯](/laravel_tw/docs/5.5/eloquent-relationships)上：

    $flight->history()->restore();

#### 永久刪除模型

有時你可能需要真的從資料庫中刪除一筆資料。使用 `forceDelete` 方法可以從資料庫中永久移除一筆被軟刪除的資料：

    // 強制刪除單一個模型實例...
    $flight->forceDelete();

    // 強制刪除所有關聯模型...
    $flight->history()->forceDelete();

<a name="query-scopes"></a>
## 查詢 Scope

<a name="global-scopes"></a>
### 全域的 Scope

全域的 Scope 可以讓你為給定模型新增條件到所有查詢。Laravel 自己的[軟刪除](#soft-deleting)功能利用全域的 Scope 能從資料庫中只取出「未刪除」模型。撰寫自己全域的 Scope 能提供更方便且簡單的方式來確保給定模型的每個查詢都能受到一定的限制。

#### 撰寫全域的 Scope

撰寫全域的 scope 是相當簡單的事情。定義實作 `Illuminate\Database\Eloquent\Scope` 介面的類別。這個介面會要求你實作 `apply` 方法，`apply` 方法可以根據需要來新增 `where` 來查詢：

    <?php

    namespace App\Scopes;

    use Illuminate\Database\Eloquent\Scope;
    use Illuminate\Database\Eloquent\Model;
    use Illuminate\Database\Eloquent\Builder;

    class AgeScope implements Scope
    {
        /**
         * 將 Scope 應用於給定的 Eloquent 查詢建構器。
         *
         * @param  \Illuminate\Database\Eloquent\Builder  $builder
         * @param  \Illuminate\Database\Eloquent\Model  $model
         * @return void
         */
        public function apply(Builder $builder, Model $model)
        {
            $builder->where('age', '>', 200);
        }
    }

> {tip} 如果你的全域的 Scope 正在新增欄位到查詢的 select 子句，你應該使用 `addSelect` 方法，而非 `select`。這可避免不小心換掉現有的查詢 select 子句。

#### 應用在全域的 Scope

要指派全域的 Scope 到模型，你應該覆寫給定模型的 `boot` 方法並使用 `addGlobalScope` 方法：

    <?php

    namespace App;

    use App\Scopes\AgeScope;
    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * 該模型的「啟動」方法。
         *
         * @return void
         */
        protected static function boot()
        {
            parent::boot();

            static::addGlobalScope(new AgeScope);
        }
    }

新增了 Scope 之後，`User::all()` 查詢會產生以下的 SQL：

    select * from `users` where `age` > 200

#### 匿名全域的 Scope

Eloquent 也可以讓你使用閉包來定義全域的，這對於簡單的作用域內相當的有用，但是不保證在獨立的類別內：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;
    use Illuminate\Database\Eloquent\Builder;

    class User extends Model
    {
        /**
         * 該模型的「啟動」方法。
         *
         * @return void
         */
        protected static function boot()
        {
            parent::boot();

            static::addGlobalScope('age', function (Builder $builder) {
                $builder->where('age', '>', 200);
            });
        }
    }

#### 移除全域的 Scope

如果你想要為給定的查詢移除全域的 Scope，你可以使用 `withoutGlobalScope` 方法。該方法接受全域的 Scope 的類別名稱作為唯一參數：

    User::withoutGlobalScope(AgeScope::class)->get();

如果你想要移除幾個或甚至所有的全域的 Scope，你可以使用 `withoutGlobalScopes` 方法：

    // 移除所有的全域的 Scope...
    User::withoutGlobalScopes()->get();

    // 移除某些全域的 Scope...
    User::withoutGlobalScopes([
        FirstScope::class, SecondScope::class
    ])->get();

<a name="local-scopes"></a>
### 局部的 Scope

局部的 Scope 讓你定義限制的共用集合，它可以輕鬆地在你的應用程式重複使用。例如，你可能需要頻繁地取得所有被認為是「受歡迎的」使用者。要定義的 Scope，必須簡單地在 Eloquent 模型方法前面加上前綴 `scope`

Scope 總是會回傳一個查詢建構器的實例：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * 只查詢受歡迎使用者的 Scope。
         *
         * @param \Illuminate\Database\Eloquent\Builder $query
         * @return \Illuminate\Database\Eloquent\Builder
         */
        public function scopePopular($query)
        {
            return $query->where('votes', '>', 100);
        }

        /**
         * 只查詢活躍使用者的 Scope。
         *
         * @param \Illuminate\Database\Eloquent\Builder $query
         * @return \Illuminate\Database\Eloquent\Builder
         */
        public function scopeActive($query)
        {
            return $query->where('active', 1);
        }
    }

#### 利用局部的 Scope

Scope 一旦被定義，你可以在查詢模型時呼叫 scope 的方法。然而，你並不需要在呼叫該方法時引入 `scope` 前綴。你甚至能鏈結呼叫各種 Scope，例如：

    $users = App\User::popular()->active()->orderBy('created_at')->get();

#### 動態 Scope

有時你可能希望去定義一個接受參數的 Scope。開始之前，只要新增額外參數到你的 Scope。Scope 參數應該在 `$query` 參數之後被定義：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * 查詢只有引入給定類型使用者的 Scope。
         *
         * @param \Illuminate\Database\Eloquent\Builder $query
         * @param mixed $type
         * @return \Illuminate\Database\Eloquent\Builder
         */
        public function scopeOfType($query, $type)
        {
            return $query->where('type', $type);
        }
    }

現在，你可以在呼叫 scope 的時候傳入該參數：

    $users = App\User::ofType('admin')->get();

<a name="events"></a>
## 事件

Eloquent 模型可以讓你觸發下列模型的生命週期幾個時間點的事件：`retrieved`、`creating`、`created`、`updating`、`updated`、`saving`、`saved`、`deleting`、`deleted`、`restoring`、`restored`。事件可以讓你每次在資料庫中儲存或更新指定的模型類別時，能夠輕易的執行程式碼。

從資料庫中取得已存在的資料時，會觸發 `retrieved` 事件。當新的資料被第一次儲存時，會觸發 `creating` 和 `created` 事件。如果資料已經存在於資料庫，並呼叫 `save`，就會觸發 `updating` 和 `updated` 事件。然而，這兩種案例都會觸發 `saving` 和 `saved` 事件。

開始之前，在你的 Eloquent 模型上定義 `$dispatchesEvents` 屬性，它會映射 Eloquent 模型的生命週期到你的[事件類別](/laravel_tw/docs/5.5/events):

    <?php

    namespace App;

    use App\Events\UserSaved;
    use App\Events\UserDeleted;
    use Illuminate\Notifications\Notifiable;
    use Illuminate\Foundation\Auth\User as Authenticatable;

    class User extends Authenticatable
    {
        use Notifiable;

        /**
         * 為模型事件映射。
         *
         * @var array
         */
        protected $dispatchesEvents = [
            'saved' => UserSaved::class,
            'deleted' => UserDeleted::class,
        ];
    }

<a name="observers"></a>
### Observer

如果你在給定模型上監聽多個事件，你可以使用 Observer 來組織你的所有監聽器到單一個類別。Observer 類別有個方法名稱，會反射你想監聽的 Eloquent 事件。這些方法中的每一個都會接收模型作為它們的參數。Laravel 預設並沒有 Observer 的目錄，不過你可以建立任何你想要的目錄來放置 Observer 類別：

    <?php

    namespace App\Observers;

    use App\User;

    class UserObserver
    {
        /**
         * 監聽使用者被建立事件。
         *
         * @param  \App\User  $user
         * @return void
         */
        public function created(User $user)
        {
            //
        }

        /**
         * 監聽使用者正在刪除事件。
         *
         * @param   \App\User  $user
         * @return void
         */
        public function deleting(User $user)
        {
            //
        }
    }

要註冊一個 observer，在你想觀察的模型上使用 observe 方法。你可以在一個服務提供者中使用 `boot` 方法來註冊 Observer。在這個範例中，我們會在 `AppServiceProvider` 中註冊 Observer：

    <?php

    namespace App\Providers;

    use App\User;
    use App\Observers\UserObserver;
    use Illuminate\Support\ServiceProvider;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * 啟動任何應用程式服務。
         *
         * @return void
         */
        public function boot()
        {
            User::observe(UserObserver::class);
        }

        /**
         * 註冊服務提供者。
         *
         * @return void
         */
        public function register()
        {
            //
        }
    }
