# 資料庫：分頁

- [介紹](#introduction)
- [基本用法](#basic-usage)
    - [對查詢建構器的結果分頁](#paginating-query-builder-results)
    - [對 Eloquent 模型分頁](#paginating-eloquent-results)
    - [手動建立一個分頁](#manually-creating-a-paginator)
- [顯示分頁結果](#displaying-pagination-results)
    - [將結果轉換成 JSON](#converting-results-to-json)
- [自訂分頁視圖](#customizing-the-pagination-view)
- [分頁器實例方法](#paginator-instance-methods)

<a name="introduction"></a>
## 介紹

在其他框架中，分頁是非常讓人苦惱的。Laravel 的分頁器是與[查詢建構器](/laravel_tw/docs/5.5/queries) 和 [Eloquent ORM](/laravel_tw/docs/5.5/eloquent) 一起被整合的，並且提供方便且易於使用的資料庫結果，還能馬上使用。分頁器產生的 HTML 可相容於 [Bootstrap CSS 框架](https://getbootstrap.com/).

<a name="basic-usage"></a>
## 基本用法

<a name="paginating-query-builder-results"></a>
### 對查詢建構器的結果分頁

這裡有幾種方式可以分頁項目。最簡單的方式是在[查詢建構器](/laravel_tw/docs/5.5/queries) 或者在 [Eloquent 查詢](/laravel_tw/docs/5.5/eloquent)上使用 `paginate` 方法。`paginate` 方法會依據目前使用者正在看的頁面自動設定正確的分頁數量與偏移數。預設情況下，目前頁數會透過 HTTP 請求中的 `page` 查詢字串參數的值來檢測。當然，Laravel 會自動檢測這個值，並且會自動插入分頁器產生的連結中。

在這個範例中，在 `paginate` 方法傳入唯一的參數，即是你想要在「每一頁」顯示的資料數量。在這個案例中，讓我們指定想要每一頁都顯示 `15` 筆資料：

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Support\Facades\DB;
    use App\Http\Controllers\Controller;

    class UserController extends Controller
    {
        /**
         * 顯示應用程式的所有使用者。
         *
         * @return Response
         */
        public function index()
        {
            $users = DB::table('users')->paginate(15);

            return view('user.index', ['users' => $users]);
        }
    }

> {note} 目前， Laravel 的分頁無法有效操作含有 `groupBy` 語句。如果你需要對使用 `groupBy` 的結果做分頁，建議你查詢資料庫後再手動製作分頁。

#### 「簡易分頁」

如果在你的視圖只需要顯示簡單的「下一步」和「上一步」連結，你可以使用 `simplePaginate` 方法來執行更高效能的查詢。在渲染視圖時，如果你不在需要為每個頁碼顯示連結，那麼這會有助於大型資料庫的效能：

    $users = DB::table('users')->simplePaginate(15);

<a name="paginating-eloquent-results"></a>
### 對 Eloquent 模型分頁

你也可以將 [Eloquent](/laravel_tw/docs/5.5/eloquent) 查詢分頁。在這個範例中，我們會將 `User` 模型分成每頁最多只有 `15` 項目。如你所見，該語法幾乎與查詢建構器的結果一樣：

    $users = App\User::paginate(15);

當然，你可以在設定查詢的其他搜尋條件之前呼叫 `paginate` 方法，像是使用 `where` 子句：

    $users = User::where('votes', '>', 100)->paginate(15);

在對 Eloquent 模型分頁的時候，你也可以使用 `simplePaginate` 方法：

    $users = User::where('votes', '>', 100)->simplePaginate(15);

<a name="manually-creating-a-paginator"></a>
### 手動建立一個分頁

有時你可能希望手動建立一個分頁實例，然後將項目陣列傳入給它。你可以根據你的實際需要來建立 `Illuminate\Pagination\Paginator` 或 `Illuminate\Pagination\LengthAwarePaginator` 其中一個實例來實現。

`Paginator` 類別不需要知道資料的總筆數。然而，也因為如此，它也無法提供取得最後一頁的方法。`LengthAwarePaginator` 與 `Paginator` 的參數幾乎相同；但是它需要資料的總筆數。

換句話說， `Paginator` 對應於在查詢建構器和 Eloquent 上的 `simplePaginate` 方法，`LengthAwarePaginator` 對應於 `paginate` 方法。

> {note}當手動建立一個分頁器實例時，你應該手動「切割」傳遞給分頁器的陣列。如果你不確定如何做到這一點，請查閱 PHP 的 [array_slice](https://secure.php.net/manual/en/function.array-slice.php) 函式。

<a name="displaying-pagination-results"></a>
## 顯示分頁結果

在呼叫 `paginate` 方法時，你會接收 `Illuminate\Pagination\LengthAwarePaginator` 實例。在呼叫 `simplePaginate` 方法時，你會接收 `Illuminate\Pagination\Paginator` 實例。這些對象提供幾種方法用來描述結果集。除了這些輔助方法，分頁器的實例也是個疊代器，並且可以像陣列一樣使用迴圈取值。所以，你一旦取得該結果，你就可以顯示該結果並使用 [Blade](/laravel_tw/docs/5.5/blade) 渲染頁面連結：

    <div class="container">
        @foreach ($users as $user)
            {% raw %} {{ $user->name }} {% endraw %}
        @endforeach
    </div>

    {% raw %} {{ $users->links() }} {% endraw %}

`links` 方法會在結果中渲染該連結到其餘頁面。這些每一個連結都已經具有正確的 `page` 查詢字串變數。請記得，使用 `link` 方法產生的 HTML 會相容於 [Bootstrap CSS 框架](https://getbootstrap.com).

#### 自訂分頁的 URI

`withPath` 方法可以讓你當透過分頁器產生連結時，使用自訂的 URI。例如，如果你想要分頁器去產生像是 `http://example.com/custom/url?page=N` 這樣的連結，你應該將 `custom/url` 傳入 `withPath` 方法：

    Route::get('users', function () {
        $users = App\User::paginate(15);

        $users->withPath('custom/url');

        //
    });

#### 附加到分頁連結

你可以使用 `appends` 方法來將查詢字串附加到分頁連結上。例如，要附加 `sort=votes` 到每一個分頁連結上，你應該照著以下範例呼叫 `appends`：

    {% raw %} {{ $users->appends(['sort' => 'votes'])->links() }} {% endraw %}

如果你希望附加一段「雜湊片段」到分頁器的連結，你可以使用 `fragment` 方法。例如，要附加 `#foo` 到每個分頁連結的最後一段，請跟著下列範例一樣呼叫 `fragment` 方法：

    {% raw %} {{ $users->fragment('foo')->links() }} {% endraw %}

<a name="converting-results-to-json"></a>
### 將結果轉換成 JSON

Laravel 分頁器結果類別實作了 `Illuminate\Contracts\Support\Jsonable` 介面的 `toJson` 方法，因此你可以非常容易地將分頁結果轉換成 JSON。你也可以從路由或控制器操作中簡單的回傳分頁器實例，並轉換成 JSON：

    Route::get('users', function () {
        return App\User::paginate();
    });

分頁器的 JSON 將包括分頁相關的資訊，如 `total` ， `current_page` ， `last_page` ，等等。該實例資料可透過 JSON 陣列中的 `data` 鍵中取得。下方是從路由回傳的分頁器實例轉換成 JSON 的一個例子：

    {
       "total": 50,
       "per_page": 15,
       "current_page": 1,
       "last_page": 4,
       "first_page_url": "http://laravel.app?page=1",
       "last_page_url": "http://laravel.app?page=4",
       "next_page_url": "http://laravel.app?page=2",
       "prev_page_url": null,
       "path": "http://laravel.app",
       "from": 1,
       "to": 15,
       "data":[
            {
                // Result Object
            },
            {
                // Result Object
            }
       ]
    }

<a name="customizing-the-pagination-view"></a>
## 自訂分頁視圖

渲染預設的視圖所顯示的分頁連結可相容於 Bootstrap CSS 框架。然而，如果你不是使用 Bootstrap，你依然可以自由的定義自己的視圖來選染這些連結。在分頁器實例上呼叫 `links` 方法時，視圖名稱將作為傳入該方法的第一個參數：

    {% raw %} {{ $paginator->links('view.name') }} {% endraw %}

    // 傳入資料到該視圖...
    {% raw %} {{ $paginator->links('view.name', ['foo' => 'bar']) }} {% endraw %}

然而，自訂分頁視圖的最容易方式是使用 `vendor:publish` 指令將它們導入到 `resources/views/vendor` 目錄中：

    php artisan vendor:publish --tag=laravel-pagination

這個指令會放置該視圖到 `resources/views/vendor/pagination` 目錄。在這個目錄中的 `default.blade.php` 檔案會對應於預設的分頁視圖。只要編輯這個檔案，就能修改分頁的 HTML。

<a name="paginator-instance-methods"></a>
## 分頁器實例方法

每個分頁器實例透過以下方法來提供額外的分頁資訊：

- `$results->count()`
- `$results->currentPage()`
- `$results->firstItem()`
- `$results->hasMorePages()`
- `$results->lastItem()`
- `$results->lastPage() (Not available when using simplePaginate)`
- `$results->nextPageUrl()`
- `$results->perPage()`
- `$results->previousPageUrl()`
- `$results->total() (Not available when using simplePaginate)`
- `$results->url($page)`
