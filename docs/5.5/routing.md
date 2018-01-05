---
layout: post
title: routing
---
# 路由

- [基本路由](#basic-routing)
    - [重導路由](#redirect-routes)
    - [視圖路由](#view-routes)
- [路由參數](#route-parameters)
    - [必要參數](#required-parameters)
    - [可選參數](#parameters-optional-parameters)
    - [正規表示式限制](#parameters-regular-expression-constraints)
- [命名路由](#named-routes)
- [路由群組](#route-groups)
    - [中介層](#route-group-middleware)
    - [命名空間](#route-group-namespaces)
    - [子網域路由](#route-group-sub-domain-routing)
    - [路由前綴](#route-group-prefixes)
- [路由模型綁定](#route-model-binding)
    - [隱式綁定](#implicit-binding)
    - [顯式綁定](#explicit-binding)
- [表單欺騙方法](#form-method-spoofing)
- [訪問目前路由](#accessing-the-current-route)

<a name="basic-routing"></a>
## 基本路由

最基本的 Laravel 路由單純地接受一個 URI 和一個`閉包`，提供一個非常簡單且直覺的方式來定義路由：

    Route::get('foo', function () {
        return 'Hello World';
    });

#### 預設路由檔案

`routes` 目錄中的路由檔案定義了所有的 Laravel 路由。這些檔案會自動被框架載入。`routes/web.php` 定義網頁介面的路由。這些路由被分配到 `web` 中介層群組，提供像是 session 狀態和 CSRF 保護的特性。`routes/api.php` 中的路由是無狀態的，且被分配到 `api` 中介層群組。

對大部分的應用程式來說，你會先由在 `routes/web.php` 檔案中定義路由開始。`routes/web.php` 中定義的路由可以透過在瀏覽器中輸入已定義的路由 URL 來訪問。例如，你可以在瀏覽器中瀏覽 `http://your-app.dev/user` 來訪問以下路由：

    Route::get('/user', 'UsersController@index');

`routes/api.php` 檔案中定義的路由會透過 `RouteServiceProvider` 被巢狀在一個路由群組中。群組中的路由會自動被加上 `/api` 前綴，因此你不需要手動在檔案中的每個路由做設定。可以藉由修改 `RouteServiceProvider` 類別來調整前綴及其他路由群組選項。

#### 可用的路由器方法

路由器能讓你註冊回應任何 HTTP 動詞的路由：

    Route::get($uri, $callback);
    Route::post($uri, $callback);
    Route::put($uri, $callback);
    Route::patch($uri, $callback);
    Route::delete($uri, $callback);
    Route::options($uri, $callback);

有時候你可能需要註冊一個回應多種 HTTP 動詞的路由。你可以使用 `match` 方法做到。甚至可以透過 `any` 方法來註冊回應所有 HTTP 動詞的路由：

    Route::match(['get', 'post'], '/', function () {
        //
    });

    Route::any('foo', function () {
        //
    });

#### CSRF 保護

任何指向 `web` 路由檔案中定義的 `POST`、`PUT` 或 `DELETE` 路由的 HTML 表單都應該包含一個 CSRF token 欄位。否則請求會被拒絕。詳細 CSRF 保護的說明請參閱 [CSRF 文件](/laravel_tw/docs/5.5/csrf)：

    <form method="POST" action="/profile">
        {% raw %} {{ csrf_field() }} {% endraw %}
        ...
    </form>

<a name="redirect-routes"></a>
### 重導路由

如果要定義重導至另一個 URI 的路由，可以使用 `Route::redirect` 方法。此方法提供一個方便的快捷方式，所以你不需要定義ㄧ個完整的路由或控制器來執行一個單純的重導：

    Route::redirect('/here', '/there', 301);

<a name="view-routes"></a>
### 視圖路由

如果路由只需要回傳一個視圖，可以使用 `Route::view` 方法。如同 `redirect` 方法，此方法提供一個方便的快捷方式，因此不須要定義一個完整的路由或控制器。`view` 方法的第一個參數是 URI，第二個參數是視圖名稱。此外，也可以在可選的第三個參數傳入提供給視圖的資料陣列。

    Route::view('/welcome', 'welcome');

    Route::view('/welcome', 'welcome', ['name' => 'Taylor']);

<a name="route-parameters"></a>
## 路由參數

<a name="required-parameters"></a>
### 必要參數

有時候你可能需要從 URI 路由中取得一些字段。例如，你可能需要從 URL 取得使用者的 ID。你可以透過定義路由參數來取得：

    Route::get('user/{id}', function ($id) {
        return 'User '.$id;
    });

你可以依照路由需要，定義任何數量的路由參數：

    Route::get('posts/{post}/comments/{comment}', function ($postId, $commentId) {
        //
    });

路由的參數都會被放在 `{}` 大括號內，只由字母組成，且不包含 `-` 字元。使用 `_` 來取代 `-`。路由參數根據它們的順序被注入到路由回呼或控制器中 - 回呼或控制器中的參數名稱不會有任何影響。

<a name="parameters-optional-parameters"></a>
### 選擇性參數

有時候你可能需要指定路由參數，但是讓路由參數的存在是選擇性的。你可以藉由在參數名稱後面加上 `?` 達成。記得給該路由的對應參數一個預設值：

    Route::get('user/{name?}', function ($name = null) {
        return $name;
    });

    Route::get('user/{name?}', function ($name = 'John') {
        return $name;
    });

<a name="parameters-regular-expression-constraints"></a>
### 正規表示式限制

你可以在一個路由實例使用 `where` 方法來限制路由參數的格式。`where` 方法接受參數的名稱和限制參數格式的正規表示式：

    Route::get('user/{name}', function ($name) {
        //
    })->where('name', '[A-Za-z]+');

    Route::get('user/{id}', function ($id) {
        //
    })->where('id', '[0-9]+');

    Route::get('user/{id}/{name}', function ($id, $name) {
        //
    })->where(['id' => '[0-9]+', 'name' => '[a-z]+']);

<a name="parameters-global-constraints"></a>
#### 全域限制

如果你想要一個路由參數總是透過指定的正規表示式限制，可以使用 `pattern` 方法。在 `RouteServiceProvider` 中的 `boot` 方法來定義這些格式：

    /**
     * 定義你的模型綁定、格式過濾器等。
     *
     * @return void
     */
    public function boot()
    {
        Route::pattern('id', '[0-9]+');

        parent::boot();
    }

一旦定義好格式後，它將自動應用於使用該參數名稱的所有路由：

    Route::get('user/{id}', function ($id) {
        // {id} 為數字時才會執行...
    });

<a name="named-routes"></a>
## 命名路由

命名路由讓你更方便為特定路由產生 URL 或重導。可以藉由鏈結 `name` 方法來為路由定義加上名稱：

    Route::get('user/profile', function () {
        //
    })->name('profile');

也可以為控制器行為指定名稱：

    Route::get('user/profile', 'UserController@showProfile')->name('profile');

#### 產生命名路由的 URL

指定名稱給路由後，便可以在使用全域的 `route` 方法產生 URL 或重導時使用路由名稱。

    // 產生 URL...
    $url = route('profile');

    // 產生重導...
    return redirect()->route('profile');

如果命名路由有定義參數，可以把參數傳給 `route` 方法的第二個參數。提供的參數會自動被插入到 URL 中正確的位置：

    Route::get('user/{id}/profile', function ($id) {
        //
    })->name('profile');

    $url = route('profile', ['id' => 1]);

#### 檢查目前路由

如果你想要確定目前的請求是否有被路由到指定的命名路由，可以使用 Route 實例的 `named` 方法。例如，可以在路由中介層中檢查目前的路由名稱：

    /**
     * 處理傳入的請求。
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \Closure  $next
     * @return mixed
     */
    public function handle($request, Closure $next)
    {
        if ($request->route()->named('profile')) {
            //
        }

        return $next($request);
    }

<a name="route-groups"></a>
## 路由群組

路由群組允許你共用路由屬性，像是中介層、命名空間，你可以利用路由群組套用這些屬性到多個路由，而不需在每個路由都設定一次。用陣列來指定共用屬性，當作 `Route::group` 方法的第一個參數：

<a name="route-group-middleware"></a>
### 中介層

要指定中介層到所有群組內的路由，你可以在定義群組前使用 `middleware` 方法。中介層將會依照在陣列中的順序執行：

    Route::middleware(['first', 'second'])->group(function () {
        Route::get('/', function () {
            // 使用 first 和 second 中介層
        });

        Route::get('user/profile', function () {
            // 使用 first 和 second 中介層
        });
    });

<a name="route-group-namespaces"></a>
### 命名空間

另一個常見的用法是利用 `namespace` 方法來指定相同的 PHP 命名空間中給一群控制器。

    Route::namespace('Admin')->group(function () {
        // 「App\Http\Controllers\Admin」 命名空間下的控制器
    });

記得，預設 `RouteServiceProvider` 會在命名空間群組內導入路由檔案，讓你不用指定完整的 `App\Http\Controllers` 命名空間前綴就能註冊控制器路由。所以，我們只需要指定在基底 `App\Http\Controllers` 命名空間之後的命名。

<a name="route-group-sub-domain-routing"></a>
### 子網域路由

路由群組也可以用來處理子網域路由。子網域可能會像是路由 URI 指定的路由參數，可以在路由或控制器中取得一部份的子網域來使用。在定義群組前呼叫 `domain` 方法來指定子網域：

    Route::domain('{account}.myapp.com')->group(function () {
        Route::get('user/{id}', function ($account, $id) {
            //
        });
    });

<a name="route-group-prefixes"></a>
### 路由前綴

`prefix` 方法可以用來為群組中的每個路由加上 URI 前綴。例如，你可能想要在群組中所有的路由 URI 加上 `admin` 前綴：

    Route::prefix('admin')->group(function () {
        Route::get('users', function () {
            // 符合 「/admin/users」 URL
        });
    });

<a name="route-model-binding"></a>
## 路由模型綁定

當注入模型的 ID 到路由或控制器行為時，你將會經常查詢來取得與該 ID 對應的模型。Laravel 路由模型綁定提供方便的方式來自動注入類別實例至路由中。例如，除了注入一個使用者的 ID，你可以注入與給定 ID 相符的完整 `User` 模型實例。

<a name="implicit-binding"></a>
### 隱式綁定

Laravel 會自動解析路由或控制器行為中變數名稱與路由片段名稱相符，並由型別提示所定義的 Eloquent 模型。例如：

    Route::get('api/users/{user}', function (App\User $user) {
        return $user->email;
    });

#### 自定鍵名

由於 `$user` 變數的型別提示為 `App\User` Eloquent 模型且與 URI 的 `{user}` 片段相符，Laravel 會自動注入與請求 URI 的 ID 值對應的模型實例。如果資料庫中找不到符合的模型實例，會自動產生一個 404 HTTP 回應。

如果你想讓模型綁定取得模型類別時使用 `id` 以外的資料庫欄位，你可以覆寫 Eloquent 模型的 `getRouteKeyName` 方法：

    /**
     * 取得路由的模型鍵值。
     *
     * @return string
     */
    public function getRouteKeyName()
    {
        return 'slug';
    }

<a name="explicit-binding"></a>
### 顯式綁定

要註冊一個顯式綁定，可使用路由器的 `model` 方法為給定參數指定類別。你必須在 `RouteServiceProvider` 的 `boot` 方法中定義顯式模型綁定：

    public function boot()
    {
        parent::boot();

        Route::model('user', App\User::class);
    }

接著，定義包含 `{user}` 參數的路由：

    Route::get('profile/{user}', function (App\User $user) {
        //
    });

因為我們已經綁定所有的 `{user}` 參數至 `App\User` 模型，`User` 實例會被注入至該路由。例如，一個至 `profile/1` 的請求會注入 ID 為 `1` 的 User 實例。

如果資料庫中找不到符合的模型實例，會自動產生一個 404 HTTP 回應。

#### 自訂解析邏輯

如果你希望使用你自己的解析邏輯，可以使用 `Route::bind` 方法。你傳遞至 `bind` 方法的`閉包`會取得 URI 的部分值，且應該回傳你想注入至路由的類別實例：

    public function boot()
    {
        parent::boot();

        Route::bind('user', function ($value) {
            return App\User::where('name', $value)->first() ?? abort(404);
        });
    }

<a name="form-method-spoofing"></a>
## 表單欺騙方法

HTML 表單沒有支援 `PUT`、`PATCH` 或 `DELETE` 動作。所以在定義由 HTML 表單所呼叫的 `PUT`、`PATCH` 或 `DELETE` 路由時，你會需要在表單中增加隱藏的 `_method` 欄位。隨著 `_method` 欄位送出的值將被視為 HTTP 請求方法使用：

    <form action="/foo/bar" method="POST">
        <input type="hidden" name="_method" value="PUT">
        <input type="hidden" name="_token" value="{% raw %} {{ csrf_token() }} {% endraw %}">
    </form>

也可以用 `method_field` 輔助函式來產生 `_method` 輸入欄位：

    {% raw %} {{ method_field('PUT') }} {% endraw %}

<a name="accessing-the-current-route"></a>
## 訪問目前路由

你可以用 `Route` facade 的 `current`、`currentRouteName` 和 `currentRouteAction` 方法來取得有關處理傳入請求的路由的資訊：

    $route = Route::current();

    $name = Route::currentRouteName();

    $action = Route::currentRouteAction();

請參考 [Route facade 的基礎類別](https://laravel.com/api/{% raw %} {{version}} {% endraw %}/Illuminate/Routing/Router.html)和[路由實例](https://laravel.com/api/{% raw %} {{version}} {% endraw %}/Illuminate/Routing/Route.html)這兩份 API 文件來詳閱可用的方法。
