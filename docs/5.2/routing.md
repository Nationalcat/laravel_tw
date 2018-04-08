---
layout: post
title: routing
tag: 5.2
---
# HTTP 路由

- [基本路由](#basic-routing)
- [路由參數](#route-parameters)
    - [必要參數](#required-parameters)
    - [選擇性參數](#parameters-optional-parameters)
- [命名路由](#named-routes)
- [路由群組](#route-groups)
    - [中介層](#route-group-middleware)
    - [命名空間](#route-group-namespaces)
    - [子網域路由](#route-group-sub-domain-routing)
    - [路由前綴](#route-group-prefixes)
- [CSRF 保護](#csrf-protection)
    - [簡介](#csrf-introduction)
    - [例外 URIs](#csrf-excluding-uris)
    - [X-CSRF-Token](#csrf-x-csrf-token)
    - [X-XSRF-Token](#csrf-x-xsrf-token)
- [路由模型綁定](#route-model-binding)
- [表單方法欺騙](#form-method-spoofing)

<a name="basic-routing"></a>
## 基本路由

你會在 `app/Http/routes.php` 檔案中為應用程式定義所有的路由，它會被 `App\Providers\RouteServiceProvider` 類別載入。而最基本的 Laravel 路由僅接收一個 URI 及一個`閉包`：

    Route::get('foo', function () {
        return 'Hello World';
    });

    Route::post('foo', function () {
        //
    });

預設情況下，`routes.php` 檔案包含了一個路由及一個套用了 `web` 中介層群組的[路由群組](#route-groups)，讓你將所有路由放置於此。這個中介層群組為路由提供了 session 狀態及 CSRF 保護。**一般來說，你會將大部分的路由放置於此群組中。**

#### 可用的路由器方法

路由器能讓你註冊回應任何 HTTP 動詞的路由：

    Route::get($uri, $callback);
    Route::post($uri, $callback);
    Route::put($uri, $callback);
    Route::patch($uri, $callback);
    Route::delete($uri, $callback);
    Route::options($uri, $callback);

有時候你可能需要讓註冊一個路由可以應對到多個 HTTP 動詞。你可以使用 `match` 方法做到。或者甚至可以透過 `any` 方法來使用註冊路由並回應所有的 HTTP 動詞：

    Route::match(['get', 'post'], '/', function () {
        //
    });

    Route::any('foo', function () {
        //
    });

<a name="route-parameters"></a>
## 路由參數

<a name="required-parameters"></a>
### 基礎路由參數

有時候你可能需要在你的 URI 路由中，取得一些參數。例如，你可能需要從 URL 取得使用者的 ID。你可以透過定義路由參數來取得：

    Route::get('user/{id}', function ($id) {
        return 'User '.$id;
    });

你可以依照路由需要，定義任何數量的路由參數：

    Route::get('posts/{post}/comments/{comment}', function ($postId, $commentId) {
        //
    });

路由的參數都會被放在「大括號」內。當執行路由時，參數會透過路由`閉包`來傳遞。

> **注意：**路由參數不能包含 `-` 字元。請使用底線 (`_`) 。

<a name="parameters-optional-parameters"></a>
### 選擇性路由參數

有時候你可能需要指定路由參數，但是讓路由參數的存在是選擇性的。你可以藉由在參數名稱後面加上 `?` 達成。記得給該路由的對應參數一個預設值：

    Route::get('user/{name?}', function ($name = null) {
        return $name;
    });

    Route::get('user/{name?}', function ($name = 'John') {
        return $name;
    });

<a name="named-routes"></a>
## 命名路由

命名路由讓你更方便為特定路由產生 URLs 或進行重導。你可以在定義路由時使用陣列的 `as` 鍵為路由指定名稱：

    Route::get('profile', ['as' => 'profile', function () {
        //
    }]);

你還可以指定路由名稱到你的控制器操作：

    Route::get('profile', [
        'as' => 'profile', 'uses' => 'UserController@showProfile'
    ]);

除了在路由的陣列定義中指定路由名稱外，你也可以在路由定義後方串連 `name` 方法：

    Route::get('user/profile', 'UserController@showProfile')->name('profile');

#### 路由群組和命名路由

假設使用[路由群組](#route-groups)，你可以指定一個 `as` 關鍵字在路由群組的屬性陣列中，來指定路由群組中共同的前綴：

    Route::group(['as' => 'admin::'], function () {
        Route::get('dashboard', ['as' => 'dashboard', function () {
            // 路由名稱為「admin::dashboard」
        }]);
    });

#### 對命名路由產生 URLs

一旦你在給定的路由中分配了名稱，你可以在透過全域 `route` 函式時，使用路由名稱產生 URLs 或是重導：

    // 產生 URLs...
    $url = route('profile');

    // 產生重導...
    return redirect()->route('profile');

若指定路由時有定義參數，可以在 `route` 函式傳入第二個參數。給定的參數將自動加入到 URL 中正確的位置：

    Route::get('user/{id}/profile', ['as' => 'profile', function ($id) {
        //
    }]);

    $url = route('profile', ['id' => 1]);

<a name="route-groups"></a>
## 路由群組

路由群組允許你共用路由屬性，像是中介層、命名空間，你可以利用路由群組套用這些屬性到多個路由，而不需在每個路由都設定一次。共用屬性被指定為陣列格式，當作 `Route::group` 方法的第一個參數：

為了暸解更多路由群組相關內容，我們會逐步介紹幾個功能及常見的使用範例。

<a name="route-group-middleware"></a>
### 中介層

要指定中介層到所有群組內的路由，你可以在群組屬性陣列裡使用 `middleware` 參數。中介層將會依照你在列表內指定的順序執行：

    Route::group(['middleware' => 'auth'], function () {
        Route::get('/', function ()    {
            // 使用 Auth 中介層
        });

        Route::get('user/profile', function () {
            // 使用 Auth 中介層
        });
    });

<a name="route-group-namespaces"></a>
### 命名空間

另一個常見的用例是，指定相同的 PHP 命名空間中給一群控制器。你可以使用 `namespace` 參數指定群組內所有控制器的命名空間前綴：

    Route::group(['namespace' => 'Admin'], function()
    {
        // 控制器在「App\Http\Controllers\Admin」命名空間

        Route::group(['namespace' => 'User'], function() {
            // 控制器在「App\Http\Controllers\Admin\User」命名空間
        });
    });

記得，預設 `RouteServiceProvider` 會在命名空間群組內導入你的 `routes.php` 檔案，讓你不用指定完整的 `App\Http\Controllers` 命名空間前綴就能註冊控制器路由。所以，我們只需要指定在基底 `App\Http\Controllers` 根命名空間之後的命名。

<a name="route-group-sub-domain-routing"></a>
### 子網域路由

路由群組也可以用來處理萬用字元的子網域。子網域可以像路由 URIs 分配路由參數，讓你在路由或控制器取得子網域參數。可以使用路由群組屬性陣列上的 `domain` 指定子網域變數名稱：

    Route::group(['domain' => '{account}.myapp.com'], function () {
        Route::get('user/{id}', function ($account, $id) {
            //
        });
    });

<a name="route-group-prefixes"></a>
### 路由前綴

`prefix` 群組屬性可以用於為群組中每個路由加上給定的 URI 前綴。舉個例子，你可能想要為路由群組中的每個路由加上 `admin` 的 URIs 前綴：

    Route::group(['prefix' => 'admin'], function () {
        Route::get('users', function ()    {
            // 符合「/admin/users」URL
        });
    });

你也可以使用 `prefix` 參數去指定路由群組中共用的參數：

    Route::group(['prefix' => 'accounts/{account_id}'], function () {
        Route::get('detail', function ($accountId)    {
            // 符合 accounts/{account_id}/detail URL
        });
    });

<a name="csrf-protection"></a>
## CSRF 保護

<a name="csrf-introduction"></a>
### 介紹

Laravel 提供簡單的方法保護你的應用程式不受到[跨網站請求偽造](http://en.wikipedia.org/wiki/Cross-site_request_forgery)（CSRF）攻擊。跨網站請求偽造是一種惡意的攻擊，藉以透過經過身份驗證的使用者身份執行未經授權的命令。

Laravel 會自動產生一個 CSRF「token」給每個活動使用者由應用程式管理的 session。該 token 用來驗證使用者為實際發出請求至應用程式的使用者。

在應用程式中定義 HTML 表單的時候，表單裡應該包含一個隱藏的 CSRF token 欄位，防護 CSRF 的中介層才能驗證請求。要產生一個包含 CSRF token 的隱藏輸入欄位 `_token`，可以使用 `csrf_field` 輔助函式：

    // 原生 PHP
    <?php echo csrf_field(); ?>

    // Blade 模板語法
    {% raw %} {{ csrf_field() }} {% endraw %}

`csrf_field` 輔助函式會產生以下的 HTML：

    <input type="hidden" name="_token" value="<?php echo csrf_token(); ?>">

你不需要手動驗證 POST、PUT 或 DELETE 請求的 CSRF token。包含在 `web` 中介層群組內的 `VerifyCsrfToken` [中介層](/laravel_tw/docs/5.2/middleware) 將自動驗證請求與 session 中的 token 是否相符。

<a name="csrf-excluding-uris"></a>
### 不受 CSRF 保護的 URIs

有時候你可能會希望一組 URIs 不要被 CSRF 保護。例如，你如果使用 [Stripe](https://stripe.com) 處理付款，並且利用他們的 webhook 系統，你需要從 Laravel CSRF 保護中，排除 webhook 的處理路由。

你可以透過在 `web` 中介層群組之外定義路由來排除 URIs，它被包含在預設的 `routes.php` 檔案中，或者可以在 `VerifyCsrfToken` 中介層中的 `$except` 屬性增加 URIs：

    <?php

    namespace App\Http\Middleware;

    use Illuminate\Foundation\Http\Middleware\VerifyCsrfToken as BaseVerifier;

    class VerifyCsrfToken extends BaseVerifier
    {
        /**
         * 這些 URIs 應該從 CSRF 驗證排除。
         *
         * @var array
         */
        protected $except = [
            'stripe/*',
        ];
    }

<a name="csrf-x-csrf-token"></a>
### X-CSRF-TOKEN

除了檢查當作 POST 參數的 CSRF token 之外，在 Laravel `VerifyCsrfToken` 中介層也會確認請求標頭中的 `X-CSRF-TOKEN`。例如，你可以將其儲存在 meta 標籤中：

    <meta name="csrf-token" content="{% raw %} {{ csrf_token() }} {% endraw %}">

一旦你建立了 `meta` 標籤，你可以使用 jQuery 之類的函式庫將 token 加入到所有的請求標頭。基於 AJAX 的應用，提供了簡單、方便的 CSRF 保護：

    $.ajaxSetup({
            headers: {
                'X-CSRF-TOKEN': $('meta[name="csrf-token"]').attr('content')
            }
    });

<a name="csrf-x-xsrf-token"></a>
### X-XSRF-TOKEN

Laravel 也會在 `XSRF-TOKEN` cookie 中儲存 CSRF token。你也可以使用 cookie 的值來設定 `X-XSRF-TOKEN` 請求標頭。一些 JavaScript 框架會自動幫你處理，例如：Angular。你很少會需要手動去設定這個值。

<a name="route-model-binding"></a>
## 路由模型綁定

Laravel 路由模型綁定提供了一個方便的方式來注入類別實例至路由中。例如，除了注入一個使用者的 ID，你可以注入與給定 ID 相符的完整 `User` 類別實例。

### 隱式綁定

Laravel 會自動解析路由或控制器行為中變數名稱與路由片段名稱相符，並由型別提示所定義的 Eloquent 模型。例如：

    Route::get('api/users/{user}', function (App\User $user) {
        return $user->email;
    });

在這個例子中，因為對路由 URI 的 `{user}` 片段所符合 `$user` 變數的 Eloquent 型別提示，所以 Laravel 會自動注入與請求 URI 的 ID 值對應的模型實例。

如果符合的模型實例不存在於資料庫中，就會自動產生一個 404 HTTP 回應。

#### 自定鍵名

如果你想讓隱式模型綁定取得模型時使用 `id` 以外的資料庫欄位，你可以覆寫 Eloquent 模型的 `getRouteKeyName` 方法：

    /**
     * 取得模型在路由的鍵。
     *
     * @return string
     */
    public function getRouteKeyName()
    {
        return 'slug';
    }

### 顯式綁定

要註冊一個顯式綁定，可以使用路由器的 `model` 方法為給定參數指定類別。你必須在 `RouteServiceProvider::boot` 方法中定義你的模型綁定：

#### 綁定參數至模型

    public function boot(Router $router)
    {
        parent::boot($router);

        $router->model('user', 'App\User');
    }

接著，定義包含 `{user}` 參數的路由：

    $router->get('profile/{user}', function(App\User $user) {
        //
    });

因為我們已經綁定 `{user}` 參數至 `App\User` 模型，所以 `User` 實例會被注入至該路由。所以，舉個例子，一個至 `profile/1` 的請求會注入 ID 為 1 的 `User` 實例。

如果符合的模型實例不存在於資料庫中，就會自動產生一個 404 HTTP 回應。

#### 自定解析邏輯

如果你希望使用你自己的解析邏輯，那麼你必須使用 `Route::bind` 方法。你傳遞至 `bind` 方法的閉包會取得 URI 的部分值，且必須回傳你想注入至路由的類別實例：

    $router->bind('user', function($value) {
        return App\User::where('name', $value)->first();
    });

#### 自定「不存在」行為

如果你希望指定你自己的「不存在」行為，只要傳遞一個閉包作為 `model` 方法的第三個參數：

    $router->model('user', 'App\User', function() {
        throw new NotFoundHttpException;
    });

<a name="form-method-spoofing"></a>
## 表單方法欺騙

HTML 表單沒有支援 `PUT`、`PATCH` 或 `DELETE` 動作。所以在定義 `PUT`、`PATCH` 或 `DELETE` 路由，並在 HTML 表單中被呼叫的時候，你將需要在表單中增加隱藏的 `_method` 欄位。隨著 `_method` 欄位送出的值將被視為 HTTP 請求方法使用：

    <form action="/foo/bar" method="POST">
        <input type="hidden" name="_method" value="PUT">
        <input type="hidden" name="_token" value="{% raw %} {{ csrf_token() }} {% endraw %}">
    </form>

要產生隱藏的輸入欄位 `_method`，你也可以使用 `methid_field` 輔助函式：

    <?php echo method_field('PUT'); ?>

當然，可以使用 Blade [模板引擎](/laravel_tw/docs/5.2/blade)：

    {% raw %} {{ method_field('PUT') }} {% endraw %}
