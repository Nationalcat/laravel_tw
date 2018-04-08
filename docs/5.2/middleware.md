---
layout: post
title: middleware
tag: 5.2
---
# HTTP 中介層

- [簡介](#introduction)
- [定義中介層](#defining-middleware)
- [註冊中介層](#registering-middleware)
    - [全域中介層](#global-middleware)
    - [為路由指派中介層](#assigning-middleware-to-routes)
    - [中介層群組](#middleware-groups)
- [中介層參數](#middleware-parameters)
- [終止的中介層](#terminable-middleware)

<a name="introduction"></a>
## 簡介

HTTP 中介層提供一個方便的機制來過濾進入應用程式的 HTTP 請求。例如，Laravel 本身使用中介層來檢驗使用者身份驗證。如果使用者未經過身份驗證，中介層會將用戶導向登入頁面，反之，當用戶通過身份驗證，中介層將會同意此請求繼續往前進。

當然，除了身份驗證之外，中介層也可以被用來執行各式各樣的任務，CORS 中介層負責替所有即將離開程式的回應加入適當的標頭。而日誌中介層可以記錄所有傳入應用程式的請求。

Laravel 框架已經內建一些中介層，包括維護、身份驗證、CSRF 保護，等等。所有的中介層都放在 `app/Http/Middleware` 目錄內。

<a name="defining-middleware"></a>
## 定義中介層

要建立一個新的中介層，可以使用 `make:middleware` 這個 Artisan 指令：

    php artisan make:middleware OldMiddleware

此指令將會在 `app/Http/Middleware` 目錄內建立一個名稱為 `OldMiddleware` 的類別。在這個中介層內我們只允許請求內的 `age` 變數大於 200 的才能存取路由，否則，我們會將用戶重新導向「home」這個 URI。

    <?php

    namespace App\Http\Middleware;

    use Closure;

    class OldMiddleware
    {
        /**
         * 執行請求過濾器。
         *
         * @param  \Illuminate\Http\Request  $request
         * @param  \Closure  $next
         * @return mixed
         */
        public function handle($request, Closure $next)
        {
            if ($request->input('age') <= 200) {
                return redirect('home');
            }

            return $next($request);
        }

    }

如你所見，若是 `age` 小於 `200`，中介層將會回傳 HTTP 重導給用戶端；否則，請求將會進一步傳遞到應用程式。只需呼叫帶有 `$request` 的 `$next` 方法，即可將請求傳遞到更深層的應用程式(允許「通過」中介層)。

HTTP 請求在實際碰觸到應用程式之前，最好是可以層層通過許多中介層，每一層都可以對請求進行檢查，甚至是完全拒絕請求。

### *前*與*後*中介層

無論中介層會在請求前執行還是會在請求後才執行，這都取決於中介層本身。例如，這個中介層會在應用程式處理請求**前**執行一些任務：

    <?php

    namespace App\Http\Middleware;

    use Closure;

    class BeforeMiddleware
    {
        public function handle($request, Closure $next)
        {
            // 執行動作

            return $next($request);
        }
    }

這個中介層則會在應用程式處理請求**後**執行它的任務：

    <?php

    namespace App\Http\Middleware;

    use Closure;

    class AfterMiddleware
    {
        public function handle($request, Closure $next)
        {
            $response = $next($request);

            // 執行動作

            return $response;
        }
    }

<a name="registering-middleware"></a>
## 註冊中介層

<a name="global-middleware"></a>
### 全域中介層

若是希望每個 HTTP 請求都經過一個中介層，只要將中介層的類別加入到 `app/Http/Kernel.php` 的 `$middleware` 屬性清單列表中。

<a name="assigning-middleware-to-routes"></a>
### 為路由指派中介層

如果你要指派中介層給特定的路由，你得先將中介層在 `app/Http/Kernel.php` 設定一個好記的鍵，預設情況下，這個檔案內的 `$routeMiddleware` 屬性已包含了 Laravel 目前設定的中介層，你只需要在清單列表中加上一組自訂的鍵即可。

    // 在 App\Http\Kernel 類別內...

    protected $routeMiddleware = [
        'auth' => \App\Http\Middleware\Authenticate::class,
        'auth.basic' => \Illuminate\Auth\Middleware\AuthenticateWithBasicAuth::class,
        'guest' => \App\Http\Middleware\RedirectIfAuthenticated::class,
        'throttle' => \Illuminate\Routing\Middleware\ThrottleRequests::class,
    ];

中介層一旦在 HTTP 核心檔案內被定義，你即可在路由選項內使用 `middleware` 鍵來指派：

    Route::get('admin/profile', ['middleware' => 'auth', function () {
        //
    }]);

使用一組陣列為路由指派多個中介層：

    Route::get('/', ['middleware' => ['first', 'second'], function () {
        //
    }]);

除了使用陣列之外，你也可以在路由的定義之後鏈結 `middleware` 方法：

    Route::get('/', function () {
        //
    }])->middleware(['first', 'second']);

<a name="middleware-groups"></a>
### 中介層群組

有時你可能想將多個中介層組成單一的鍵，讓它可以簡單的指派給路由。你可以使用 HTTP 核心的 `$middlewareGroups` 屬性做到。

Laravel 內建了 `web` 及 `api` 中介層群組，包含了你想套用在 web UI 與 API 路由常用的中介層：

    /**
     * 應用程式的路由中介層群組。
     *
     * @var array
     */
    protected $middlewareGroups = [
        'web' => [
            \App\Http\Middleware\EncryptCookies::class,
            \Illuminate\Cookie\Middleware\AddQueuedCookiesToResponse::class,
            \Illuminate\Session\Middleware\StartSession::class,
            \Illuminate\View\Middleware\ShareErrorsFromSession::class,
            \App\Http\Middleware\VerifyCsrfToken::class,
        ],

        'api' => [
            'throttle:60,1',
            'auth:api',
        ],
    ];

中介層群組可以使用與單一中介層一樣的語法指派給路由及控制器行為。同樣的，中介層群組只是讓一次指派多個中介層置路由更方便：

    Route::group(['middleware' => ['web']], function () {
        //
    });

<a name="middleware-parameters"></a>
## 中介層參數

中介層也可以額外接受自訂參數，例如，如果應用程式要在執行特定操作之前，檢查通過驗證的使用者是否具備該操作的「角色」，可以建立 `RoleMiddleware` 來接收角色名稱作為額外的參數。

附加的中介層參數將會在 `$next` 參數之後被傳入中介層：

    <?php

    namespace App\Http\Middleware;

    use Closure;

    class RoleMiddleware
    {
        /**
         * 執行請求過濾
         *
         * @param  \Illuminate\Http\Request  $request
         * @param  \Closure  $next
         * @param  string  $role
         * @return mixed
         */
        public function handle($request, Closure $next, $role)
        {
            if (! $request->user()->hasRole($role)) {
                // Redirect...
            }

            return $next($request);
        }

    }

在路由中可使用冒號 `:` 來區隔中介層名稱與指派參數，多筆參數可使用逗號作為分隔：

    Route::put('post/{id}', ['middleware' => 'role:editor', function ($id) {
        //
    }]);

<a name="terminable-middleware"></a>
## 終止的中介層

有些時候中介層需要在 HTTP 回應已被傳送到用戶端之後才執行，例如，Laravel 內建的「session」中介層，儲存 session 資料是在回應已被傳送到用戶端_之後_才執行。為了做到這一點，你需要在中介層增加 `terminable` 方法定義中介層為「終止的」。

    <?php

    namespace Illuminate\Session\Middleware;

    use Closure;

    class StartSession
    {
        public function handle($request, Closure $next)
        {
            return $next($request);
        }

        public function terminate($request, $response)
        {
            // 儲存 session 資料...
        }
    }

`terminate` 方法必須接收請求及回應。一旦定義了終止的中介層，你需要將它增加到 HTTP 核心檔案的全域中介層清單列表中。

當在你的中介層呼叫 `terminate` 方法時，Laravel 會從[服務容器](/laravel_tw/docs/5.2/container)解析一個全新的中介層實例。如果你希望在 `handle` 及 `terminate` 方法被呼叫時使用一致的中介層實例，只需在容器中使用容器的 `singleton` 方法註冊中介層。
