---
layout: post
title: middleware
tag: 5.5
---
# 中介層

- [介紹](#introduction)
- [定義中介層](#defining-middleware)
- [註冊中介層](#registering-middleware)
    - [全域中介層](#global-middleware)
    - [分配中介層給路由](#assigning-middleware-to-routes)
    - [中介層群組](#middleware-groups)
- [中介層參數](#middleware-parameters)
- [終止的中介層](#terminable-middleware)

<a name="introduction"></a>
## 介紹

中介層提供了一種方便的機制來過濾進入應用程式的 HTTP 請求。例如 Laravel 其中有一個中介層是被用於驗證應用程式的使用者是否有通過認證。如果使用者沒通過認證，中介層就會將使用者重導到登入頁面。但是，如果使用者通過認證的話，中介層就可以讓使用者更近一步的進入應用程式。

當然，除了可以驗證，還能撰寫額外的中介層來執行各種任務。CORS 中介層能去負責將正確的標頭新增到即將離開應用程式的所有回應中。日誌中介層能將所有傳入的請求記錄到應用程式中。

在 Laravel 框架中，還包含了幾個中介層，其中有用於認證和 CSRF 保護的中介層。這些所有的中介層都放置於 `app/Http/Middleware` 目錄中。

<a name="defining-middleware"></a>
## 定義中介層

請使用 Artsian 的 `make:middleware` 指令來建立新的中介層：

    php artisan make:middleware CheckAge

這個指令會在 `app/Http/Middleware` 目錄中放置剛建立的 `CheckAge` 類別。在這個中介層中，我們只會讓有滿足於 `age` 大於 200 條件的請求能去存取該路由。否則，我們會將使用者重導回 `home` URL。

    <?php

    namespace App\Http\Middleware;

    use Closure;

    class CheckAge
    {
        /**
         * 處理傳進來的請求。
         *
         * @param  \Illuminate\Http\Request  $request
         * @param  \Closure  $next
         * @return mixed
         */
        public function handle($request, Closure $next)
        {
            if ($request->age <= 200) {
                return redirect('home');
            }

            return $next($request);
        }
    }

正如你所見，如果給定的 `age` 小於等於 `200`，該中介層會回傳 HTTP 重導給客戶端。否則，該請求會被更近一步的傳入應用程式。要將請求更深入的傳入應用程式（允許讓中介層「傳入」），只要呼叫帶有 `$request` 的 `$next` 回呼。

在 HTTP 請求要接觸應用程式之前，最好是可以讓請求先層層通過中介層這些關卡。每一層關卡都能對請求進行檢查，甚至完全拒絕它。

### 中介層之前與之後

無論中介層想要在請求之前還是之後執行才執行，這都取決於中介層本身。例如，以下中介層會在應用程式處理請求**之前**執行一些任務：

    <?php

    namespace App\Http\Middleware;

    use Closure;

    class BeforeMiddleware
    {
        public function handle($request, Closure $next)
        {
            // 執行操作

            return $next($request);
        }
    }

然而，這個中介層會在應用程式處理請求**之後**才執行任務：

    <?php

    namespace App\Http\Middleware;

    use Closure;

    class AfterMiddleware
    {
        public function handle($request, Closure $next)
        {
            $response = $next($request);

            // 執行操作

            return $response;
        }
    }

<a name="registering-middleware"></a>
## 註冊中介層

<a name="global-middleware"></a>
### 全域中介層

如果你想要在應用程式的每個 HTTP 請求期間執行一個中介層，只需要在 `app/Http/Kernel.php` 類別的 `$middleware` 屬性中列出中介層類別。

<a name="assigning-middleware-to-routes"></a>
### 分配中介層給路由

如果你想要分配中介層到特定路由，你首先在 `app/Http/Kernel.php` 檔案中分配該中介層的鍵。預設這個類別的 `$routeMiddleware` 屬性會包含 Laravel 中所有的中介層。若要新增自己的中介層，只需要將它附加到這個清單上，並為這個中介層分配一個你選擇的鍵。例如：

    // 在 App\Http\Kernel 類別中...

    protected $routeMiddleware = [
        'auth' => \Illuminate\Auth\Middleware\Authenticate::class,
        'auth.basic' => \Illuminate\Auth\Middleware\AuthenticateWithBasicAuth::class,
        'bindings' => \Illuminate\Routing\Middleware\SubstituteBindings::class,
        'can' => \Illuminate\Auth\Middleware\Authorize::class,
        'guest' => \App\Http\Middleware\RedirectIfAuthenticated::class,
        'throttle' => \Illuminate\Routing\Middleware\ThrottleRequests::class,
    ];

一旦在 HTTP 核心中定義了中介層，你就能使用 `middleware` 方法將中介層指派到路由上：

    Route::get('admin/profile', function () {
        //
    })->middleware('auth');

你還可以分配多個中介層到路由：

    Route::get('/', function () {
        //
    })->middleware('first', 'second');

在分配中介層時，你也可以傳入完整的類別名稱：

    use App\Http\Middleware\CheckAge;

    Route::get('admin/profile', function () {
        //
    })->middleware(CheckAge::class);

<a name="middleware-groups"></a>
### 中介層群組

有時你可能想要將幾個中介層匯集到同一個鍵，以便之後能夠輕鬆的將它分配給路由。你可以使用 HTTP 核心的 `$middlewareGroups` 屬性來做到。

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
            \Illuminate\Routing\Middleware\SubstituteBindings::class,
        ],

        'api' => [
            'throttle:60,1',
            'auth:api',
        ],
    ];

中介層群組可以使用與單一中介層一樣的語法指派給路由及控制器行為。同樣的，中介層群組只是讓一次指派多個中介層置路由更方便：

    Route::get('/', function () {
        //
    })->middleware('web');

    Route::group(['middleware' => ['web']], function () {
        //
    });

> {tip} `web` 中介層群組會被 `RouteServiceProvider` 自動的應用到 `routes/web.php` 檔案中。

<a name="middleware-parameters"></a>
## 中介層參數

中介層也可以接受額外的參數。例如，如果你的應用程式需要在執行給定行為之前驗證已認證使用者是否具有給定的「角色」，你可以建立一個 `CheckRole` 中介層，該中介層可以接收角色名稱作為附加參數。

附加的中介層參數將會在 `$next` 參數之後被傳入中介層：

    <?php

    namespace App\Http\Middleware;

    use Closure;

    class CheckRole
    {
        /**
         * 處理傳進來的請求。
         *
         * @param  \Illuminate\Http\Request  $request
         * @param  \Closure  $next
         * @param  string  $role
         * @return mixed
         */
        public function handle($request, Closure $next, $role)
        {
            if (! $request->user()->hasRole($role)) {
                // 重導...
            }

            return $next($request);
        }

    }

在路由中可使用冒號 `:` 來區隔中介層名稱與指派參數，多筆參數可使用逗號作為分隔：

    Route::put('post/{id}', function ($id) {
        //
    })->middleware('role:editor');

<a name="terminable-middleware"></a>
## 終止的中介層

在 HTTP 回應發送到瀏覽器後，有時會需要中介層去執行一些任務。例如，Laravel 中內建的「session」中介層會在發送到瀏覽器後將 Session 寫入儲存器中。如果你在中介層上定義了 `terminate` 方法，那麼它會在回應發送瀏覽器之後自動被呼叫。

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
            // 儲存 Session 資料...
        }
    }

`terminate` 方法會同時接收請求和回應。一旦你定義了一個用於終止的中介層，你應該把它新增到 `app/Http/Kernel.php` 檔案中的路由或全域的清單中。

當在你的中介層呼叫 `terminate` 方法時，Laravel 會從[服務容器](/laravel_tw/docs/5.5/container)解析一個全新的中介層實例。如果你希望在 `handle` 及 `terminate` 方法被呼叫時使用一致的中介層實例，只需在容器中使用容器的 `singleton` 方法註冊中介層。
