---
layout: post
title: authentication
---
# 認證

- [簡介](#introduction)
    - [資料庫注意事項](#introduction-database-considerations)
- [認證快速入門](#authentication-quickstart)
    - [路由](#included-routing)
    - [視圖](#included-views)
    - [認證](#included-authenticating)
    - [取得已認證之使用者](#retrieving-the-authenticated-user)
    - [保護路由](#protecting-routes)
    - [登入限流](#login-throttling)
- [手動認證使用者](#authenticating-users)
    - [記住使用者](#remembering-users)
    - [其它認證方法](#other-authentication-methods)
- [HTTP 基礎認證](#http-basic-authentication)
    - [無狀態 HTTP 基礎認證](#stateless-http-basic-authentication)
- [社群認證](https://github.com/laravel/socialite)
- [增加自定義 Guard](#adding-custom-guards)
- [增加自定義 User Providers](#adding-custom-user-providers)
    - [The User Provider Contract](#the-user-provider-contract)
    - [The Authenticatable Contract](#the-authenticatable-contract)
- [事件](#events)

<a name="introduction"></a>
## 簡介

> {tip} **想要快速起步？**在一個全新的 Laravel 應用中執行 `php artisan make:auth` 和 `php artisan migrate`  指令，接著可用瀏覽器開啟 `http://your-app.dev/register` 或其他在程式中定義的 url。這兩個簡單的指令就可以搭建好整個認證系統的骨架！

Laravel 中實現使用者認證非常簡單。實際上，幾乎所有東西都已經為你配置好了。配置檔案位於 `config/auth.php`，其中包含了用於調整認證服務行為的、標註好註釋的選項配置。

在其核心程式碼中，Laravel 的認證元件由「guards」和「providers」組成，Guard 定義了使用者在每個請求中如何被認證，例如，Laravel `session` guard 狀態的維護是使用 session 儲存和 cookie。

Provider 定義如何從持久化儲存中獲取使用者資訊，Laravel 底層支援透過 Eloquent 和資料庫查詢建構器兩種方式來獲取使用者，如果需要的話，你還可自由地定義額外的 Provider。

如果看到這些名詞覺得很困惑，大可不必太過擔心，因為對絕大多數應用程式而言，只需使用預設認證配置即可，不需要做什麼改動。

<a name="introduction-database-considerations"></a>
### 資料庫注意事項

預設的 Laravel 在 `app` 資料夾中會含有 `App\User` [Eloquent 模型](/laravel_tw/docs/5.3/eloquent)。這個模型將使用預設的 Eloquent 認證來驅動。如果你的應用程式沒有使用 Eloquent，請選擇使用 Laravel 查詢建構器的 `database` 認證驅動。

為 `App\User` 模型建立資料庫表結構時，確認密碼欄位最少需 60 字元長。保持欄位原定的 255 字元長是個好選擇。

同時，`users` 資料表中必須含有 nullable 、100 字元長的 `remember_token` 欄位。當用戶登入應用程式並勾選「記住我」時，這個欄位將會被用來儲存「記住我」session 的 token。

<a name="authentication-quickstart"></a>
## 認證快速入門

Laravel 內建有數個認證控制器，它們被放置在 `App\Http\Controllers\Auth` 命名空間內，`RegisterController` 處理使用者註冊，`LoginController` 處理使用者認證，`ForgotPasswordController` 處理重置密碼的 e-mail 連結，`ResetPasswordController` 包含重置密碼的邏輯。每個控制器都使用 trait 來包含必要的方法。對於大部分應用程式而言，並不需要修改這些控制器。
<a name="included-routing"></a>
### 路由

Laravel 提供一個快速的指令來幫你建立所有認證所需的路由及視圖，指令如下：

    php artisan make:auth

這個指令應該只被用在全新的應用程式。這個指令將會安裝註冊和登入視圖，以及所有認證相關的路由。這個指令也會產生一個 `HomeController`，這個控制器負責處理登入後的應用程式導覽頁面。然而，你可以自由地根據你的應用程式所需來自訂或移除這些控制器。

<a name="included-views"></a>
### 視圖

在上面的段落有提到，`php artisan make:auth` 這個指令會建立所有你在認證相關的視圖，並且放到 `resources/views/auth` 中。

`make:auth` 這個指令也會同時建立一個 `resources/views/layouts` 資料夾，內含一個預設的版面設計給你的應用程式使用。所有預設的視圖都是使用 Bootstrap CSS 框架，你可以自由地依你所需來客製化它們。

<a name="included-authenticating"></a>
### 認證

現在你已經為認證控制器設定好路由及視圖，可以開始讓新使用者註冊並來登入應用程式了。你只要簡單地在瀏覽器存取定義好的路由，認證控制器已經（透過他們各自的 traits）包含認證現有使用者和儲存新使用者的邏輯了。

#### 客製化路徑

當使用者成功的認證後，他們將被重導到 `/home` URI。你可以透過修改 `LoginController`、`RegisterController` 和 `ResetPasswordController` 的 `redirectTo` 屬性自訂認證成功後自動重導的 URI：

    protected $redirectTo = '/';

當一個使用者登入認證失敗後，預設將會自動重導回登入表單頁面。
假如重導路徑需要客製化邏輯，可以定義 `redirectTo` 方法，而不是 `redirectTo` 屬性：

    protected function redirectTo()
    {
        //
    }

> {tip} `redirectTo` 方法比 `redirectTo` 屬性有更高的優先權。

#### 客製化使用者名稱

Laravel 預設使用 `email` 欄位來認證，如果要客製此欄位，可以在 `LoginController` 中定義 `username` 方法：

    public function username()
    {
        return 'username';
    }

#### 客製化 Guard 

你還可以自定實作使用者認證和註冊的「guard」。開始之前，需要在 `LoginController`、`RegisterController` 和 `ResetPasswordController` 中定義 `guard` 方法，該方法需要傳回一個 guard 實例：

    use Illuminate\Support\Facades\Auth;

    protected function guard()
    {
        return Auth::guard('guard-name');
    }

#### 客製化驗證（Validation）/ 儲存

要修改新使用者註冊所必需的表單欄位，或者自定新使用者欄位如何儲存到資料庫，可以修改 `RegisterController` 類別。該類別負責為應用程式驗證輸入欄位和建立新使用者。

`RegisterController` 的 `validator` 方法包含了新使用者的驗證規則，可以依需求自由地修改該方法。

`RegisterController` 的 `create` 方法負責使用 [Eloquent ORM](/laravel_tw/docs/5.3/eloquent) 在資料庫中建立新的 `App\User` 記錄，也可以基於自己的資料庫需求自由地修改該方法。

<a name="retrieving-the-authenticated-user"></a>
### 取得已認證之使用者

你可以透過 `Auth` facade 來存取認證的使用者：

    use Illuminate\Support\Facades\Auth;

    // 取得目前的已認證使用者...
    $user = Auth::user();

    // 取得目前的已認證使用者 ID...
    $id = Auth::id();

也有另外一種方法可以存取認證過的使用者，就是透過 `Illuminate\Http\Request` 實例。請記得，型別提示的類別將會被自動注入：

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;

    class ProfileController extends Controller
    {
        /**
         * 更新使用者的資料
         *
         * @param  Request  $request
         * @return Response
         */
        public function update(Request $request)
        {
            // $request->user() 回傳已認證之使用者...
        }
    }

#### 檢查使用者是否登入

為了確認使用者是否已經登入，你可以使用 `Auth` facade 的 `check` 方法，如果使用者被認證過，將會回傳 `true`：

    use Illuminate\Support\Facades\Auth;

    if (Auth::check()) {
        // The user is logged in...
    }

> {tip} 儘管可以使用 `check` 方法來檢查使用者是否登入，在允許該使用者存取特定的路由或控制器之前，可以使用中介層來檢查使用者是否認證過。想得到更多資訊，請閱讀[保護路由](/laravel_tw/docs/5.3/authentication#protecting-routes)的文件。

<a name="protecting-routes"></a>
### 保護路由

[路由中介層](/laravel_tw/docs/5.3/middleware)用於限認證過的使用者存取指定路由，Laravel 提供了 `auth` 中介層來達到這個目的，而這個中介層被定義在 `Illuminate\Auth\Middleware\Authenticate` 中，它已經被註冊到 HTTP 核心中，你只需要將它附加到路由定義中：

    Route::get('profile', function () {
        // 只有認證過的使用者能進來這裡...
    })->middleware('auth');

當然，如果你正在使用[控制器類別](/laravel_tw/docs/5.3/controllers)，你可以在建構子中呼叫 middleware 方法，而不是在路由中直接定義它：

    public function __construct()
    {
        $this->middleware('auth');
    }

#### 指定 Guard

新增 `auth` 中介層到路由後，還需要指定使用哪個 guard 來處理認證。指定的 guard 對應配置檔案 `auth.php` 中 `guards` 陣列的某個鍵名：

    public function __construct()
    {
        $this->middleware('auth:api');
    }

<a name="login-throttling"></a>
### 登入限流

Laravel 內建的 `LoginController` 類別提供 `Illuminate\Foundation\Auth\ThrottlesLogins` trait 允許你在應用程式中限制登入次數。預設情況下，如果使用者在進行幾次嘗試後仍不能提供正確的憑證，將在一分鐘內無法進行登入。這個限制會特別針對使用者的名稱 / e-mail 和他們的 IP 地址。

<a name="authenticating-users"></a>
## 手動認證使用者

當然，不一定要使用 Laravel 內建的認證控制器。如果選擇刪除這些控制器，可以直接呼叫 Laravel 的認證類別來管理使用者認證。不用擔心，很簡單。

我們可以利用 `Auth` [facade](/laravel_tw/docs/5.3/facades) 來存取 Laravel 的認證服務，因此需確認在類別的頂部匯入 `Auth` facade。接下來讓我們看一下 Auth 的 `attempt` 方法：

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Support\Facades\Auth;

    class LoginController extends Controller
    {
        /**
         * 處理認證
         *
         * @return Response
         */
        public function authenticate()
        {
            if (Auth::attempt(['email' => $email, 'password' => $password])) {
                // 認證通過...
                return redirect()->intended('dashboard');
            }
        }
    }

`attempt` 方法會接受一個陣列來作為第一個參數，這個陣列的值會用來找尋資料庫裡的使用者資料，所以在上面的範例中，使用者會藉由 `email` 欄位取得，如果使用者被找到了，資料庫裡經過雜湊的密碼將會與陣列中雜湊的 `password` 值做比對，如果兩個雜湊密碼相符的話，會開啟一個通過認證的 session 給使用者。

如果認證成功，`attempt` 方法將會回傳 `true`，反之則為 `false`。

重導器上的 `intended` 方法將會重導使用者回原本想要進入的頁面，也可以傳入一個備用 URI 至這個方法，以避免要導回的頁面無法使用。

#### 指定額外條件

也可以加入除了使用者的電子信箱及密碼的額外條件至認證查詢。例如，我們要確認使用者是否被標記為「active」：

    if (Auth::attempt(['email' => $email, 'password' => $password, 'active' => 1])) {
        // 這個使用者是存在且有效的，沒有被停權
    }

> {note} 在這些例子中，`email` 不是一個必要的選項，它只是被用來當作範例。你可以使用資料庫中任何等同於「使用者名稱」的欄位。

#### 存取特定的 Guard 實例

可以透過 `Auth` facade 的 `guard` 方法來指定使用特定的 guard 實例。這允許你在管理應用程式的不同部份時，使用完全不同的認證模組或使用者資料表。

`guard` 方法所指定的 guard 名稱必須對應到 `auth.php` 設定檔中 guards 陣列的其中一個鍵名：

    if (Auth::guard('admin')->attempt($credentials)) {
        //
    }

#### 登出

為了讓使用者登出，你可以使用 `Auth` facade 的 `logout` 方法。這個方法會清除使用者在 session 中所有認證相關的資料：

    Auth::logout();

<a name="remembering-users"></a>
### 記住使用者

如果你想要提供「記住我」的功能，你需要傳入一個布林值到 `attempt` 方法的第二個參數，這會永久保持使用者的 session 直到使用者手動登出。你的 `users` 資料表一定要包含一個 `remember_token` 欄位，這是用來儲存「記住我」的 token。

    if (Auth::attempt(['email' => $email, 'password' => $password], $remember)) {
        // 這個使用者被記住了...
    }

> {tip} 如果使用 Laravel 內建的 `LoginController`，「記住我」的邏輯已經透過 traits 實作完成。

可以使用 `viaRemember` 方法來檢查這個使用者是否使用「記住我」cookie 來做認證：

    if (Auth::viaRemember()) {
        //
    }

<a name="other-authentication-methods"></a>
### 其它認證方法

#### 用使用者實例做認證

如果你需要使用存在的使用者實例來登入，你需要呼叫 `login` 方法，並傳入使用者實例，這個物件必須實作 `Illuminate\Contracts\Auth\Authenticatable` [contract](/laravel_tw/docs/5.3/contracts)。當然，內建的 `App/User` 模型已經實作了這個介面：

    Auth::login($user);

    // 登入並且「記住」使用者...
    Auth::login($user, true);

當然，你也可以指定 guard 實例：

    Auth::guard('admin')->login($user);

#### 用使用者 ID 做認證

如果你需要使用使用者的 ID 來登入，你需要使用 `loginUsingId` 方法，這個方法只接受要登入的使用者的主鍵：

    Auth::loginUsingId(1);

    // 登入並且「記住」使用者...
    Auth::loginUsingId(1, true);

#### 一次性的使用者認證

你可以使用 `once` 方法來只針對一次的請求來認證使用者，沒有任何的 session 或 cookie 會被使用，這個對於建議無狀態的 API 非常的有用：

    if (Auth::once($credentials)) {
        //
    }

<a name="http-basic-authentication"></a>
## HTTP 基礎認證

[HTTP 基礎認證](https://en.wikipedia.org/wiki/Basic_access_authentication) 提供一個快速的方法來認證使用者，不需要任何「登入」頁面。開始之前，先增加 `auth.basic` [中介層](/laravel_tw/docs/5.3/middleware)到你的路由，`auth.basic` 中介層已經被包含在 Laravel 框架中，所以你不需要定義它：

    Route::get('profile', function () {
        // 只有認證過的使用者可進入...
    })->middleware('auth.basic');

一旦中介層被增加到路由上，當使用瀏覽器進入這個路由時，會自動提示你需要提供憑證。預設上，`auth.basic` 中介層將會使用使用者的 `email` 欄位當作「使用者名稱」。

#### FastCGI 的注意事項

如果是正在使用 FastCGI，HTTP 基礎認證可能無法正常運作，需要將下面這幾行設定加入 `.htaccess` 檔案中：

    RewriteCond %{HTTP:Authorization} ^(.+)$
    RewriteRule .* - [E=HTTP_AUTHORIZATION:%{HTTP:Authorization}]

<a name="stateless-http-basic-authentication"></a>
### 無狀態 HTTP 基礎認證

你可以使用 HTTP 基礎認證而不用在 session 中設定使用者認證用的 cookie，這個功能對 API 認證來說非常有用。為了達到這個目的，[定義一個中介層](/laravel_tw/docs/5.3/middleware)並呼叫 `onceBasic` 方法。如果沒有任何回應從 `onceBasic` 方法回傳的話，這個請求就會進一步傳進應用程式中：

    <?php

    namespace Illuminate\Auth\Middleware;

    use Illuminate\Support\Facades\Auth;

    class AuthenticateOnceWithBasicAuth
    {
        /**
         * 處理請求
         *
         * @param  \Illuminate\Http\Request  $request
         * @param  \Closure  $next
         * @return mixed
         */
        public function handle($request, $next)
        {
            return Auth::onceBasic() ?: $next($request);
        }

    }

接著，[註冊這個路由中介層](/laravel_tw/docs/5.3/middleware#registering-middleware)，然後將它增加在一個路由上：

    Route::get('api/user', function () {
        // 只有認證使用者能進入...
    })->middleware('auth.basic.once');

<a name="adding-custom-guards"></a>
## 增加自定義 Guard

可以使用 `Auth` 的 `extend` 方法來自定認證 Guard，你需要在[服務提供者](/laravel_tw/docs/5.3/providers)中的 `provider`放置此程式碼呼叫。由於 Laravel 已經置於 `AuthServiceProvider`，可以把程式碼放入這個提供者：

    <?php

    namespace App\Providers;

    use App\Services\Auth\JwtGuard;
    use Illuminate\Support\Facades\Auth;
    use Illuminate\Foundation\Support\Providers\AuthServiceProvider as ServiceProvider;

    class AuthServiceProvider extends ServiceProvider
    {
        /**
         * 註冊應用程式的任意認證/授權服務
         *
         * @return void
         */
        public function boot()
        {
            $this->registerPolicies();

            Auth::extend('jwt', function ($app, $name, array $config) {
                // 回傳 Illuminate\Contracts\Auth\Guard 實例...

                return new JwtGuard(Auth::createUserProvider($config['provider']));
            });
        }
    }

正如上面的程式碼所示，`extend` 方法傳參進去的回呼需要傳回 `Illuminate\Contracts\Auth\Guard` 的實例，這個介面有幾個方法需要實作。一旦定義好 Guard 以後，可在 `auth.php` 設定檔中使用 `guards` 配置：

    'guards' => [
        'api' => [
            'driver' => 'jwt',
            'provider' => 'users',
        ],
    ],

<a name="adding-custom-user-providers"></a>
## 增加自定義 User Provider

如果沒有使用傳統的關連式資料庫儲存使用者資訊，則需要使用自己的認證 user provider 來擴充 Laravel。我們使用 `Auth` facade 上的 `provider` 方法自定 user provider：

    <?php

    namespace App\Providers;

    use Illuminate\Support\Facades\Auth;
    use App\Extensions\RiakUserProvider;
    use Illuminate\Support\ServiceProvider;

    class AuthServiceProvider extends ServiceProvider
    {
        /**
         * 註冊應用程式的任意認證/授權服務
         *
         * @return void
         */
        public function boot()
        {
            $this->registerPolicies();

            Auth::provider('riak', function ($app, array $config) {
                // 回傳 Illuminate\Contracts\Auth\UserProvider 實例...

                return new RiakUserProvider($app->make('riak.connection'));
            });
        }
    }

透過 `provider` 方法註冊 provider 後，你可以在設定檔案 `auth.php` 中切換到新的 user provider。首先，在該設定檔定義一個 `provider` 的 user 驅動方式：

    'providers' => [
        'users' => [
            'driver' => 'riak',
        ],
    ],

然後，可以在你的 `guards` 設定中使用這個提供者：

    'guards' => [
        'web' => [
            'driver' => 'session',
            'provider' => 'users',
        ],
    ],

<a name="the-user-provider-contract"></a>
### The User Provider Contract

`Illuminate\Contracts\Auth\UserProvider` 的實作只負責獲取 `Illuminate\Contracts\Auth\Authenticatable` 的一個實作， 且不受限於永久儲存系統，例如 MySQL, Riak 等等。這兩個介面允許 Laravel 認證機制繼續作用，而不用管使用者如何儲存或是使用什麼樣型別的類別實現它。

讓我們來看看 `Illuminate\Contracts\Auth\UserProvider` contract:

    <?php

    namespace Illuminate\Contracts\Auth;

    interface UserProvider {

        public function retrieveById($identifier);
        public function retrieveByToken($identifier, $token);
        public function updateRememberToken(Authenticatable $user, $token);
        public function retrieveByCredentials(array $credentials);
        public function validateCredentials(Authenticatable $user, array $credentials);

    }

`retrieveById` 函式通常傳入一個代表使用者的值，例如 MySQL 中遞增的 ID。透過 ID 匹配的方法來取出和回傳 `Authenticatable` 的實作。

`retrieveByToken` 函式藉由使用者唯一的 `$identifier` 和「記住我」`$token` 來取得使用者。如同之前的方法，應該回傳 `Authenticatable` 的實作。

`updateRememberToken` 方法使用新的 `$token` 更新了 `$user` 的 `remember_token` 欄位。這個新的 token 可以是全新的 token（嘗試使用「記住我」登入成功時），或是 null（當用戶登出時）。

`retrieveByCredentials` 方法獲取了從 `Auth::attempt` 方法傳送過來的憑證陣列（當想要登入時）。這個方法應該要 「查詢」所使用的持久化儲存系統來匹配這些憑證。通常，這個方法會執行一個帶著「where」`$credentials['username']` 條件的查詢。這個方法接著需要回傳一個 `Authenticatable` 的一個實作。**此方法不應該企圖做任何密碼驗證或認證操作。**

`validateCredentials` 方法需要比較 `$user` 和 `$credentials` 來認證這個使用者。例如，這個方法可能會比較 `Hash::check` 後的 `$user->getAuthPassword()` 值及 `$credentials['password']` 值。這個方法驗證密碼的有效性並應該只回傳布林值。

<a name="the-authenticatable-contract"></a>
### The Authenticatable Contract

剛剛我們已經探討了 `UserProvider` 的每個方法，讓我們看一下 `Authenticatable` contract。還記得吧，UserProvider 需要從 `retrieveById` 和 `retrieveByCredentials` 方法來回傳 Authenticatable 介面的實作：

    <?php

    namespace Illuminate\Contracts\Auth;

    interface Authenticatable {

        public function getAuthIdentifierName();
        public function getAuthIdentifier();
        public function getAuthPassword();
        public function getRememberToken();
        public function setRememberToken($value);
        public function getRememberTokenName();

    }

這個介面很簡單。`getAuthIdentifierName` 方法需要回傳使用者的「主鍵」欄位名。`getAuthIdentifier` 方法需要回傳使用者的「主鍵」。在 MySQL 中，指的是自動遞增主鍵。而 `getAuthPassword` 應該要回傳使用者雜湊後的密碼。這個介面允許認證系統和任何使用者類別運作，不管使用何種 ORM 或儲存抽象層。預設情況下，Laravel 的 `app` 資料夾中會包含 `User` 類別來實作此介面，所以你可以觀察這個類別以作為實作的例子。

<a name="events"></a>
## 事件

Laravel 提供了在認證過程中的各種[事件](/laravel_tw/docs/5.3/events)。你可以在 `EventServiceProvider` 為這些事件附加監聽器：

    /**
     * The event listener mappings for the application.
     *
     * @var array
     */
    protected $listen = [
        'Illuminate\Auth\Events\Registered' => [
            'App\Listeners\LogRegisteredUser',
        ],

        'Illuminate\Auth\Events\Attempting' => [
            'App\Listeners\LogAuthenticationAttempt',
        ],

        'Illuminate\Auth\Events\Authenticated' => [
            'App\Listeners\LogAuthenticated',
        ],

        'Illuminate\Auth\Events\Login' => [
            'App\Listeners\LogSuccessfulLogin',
        ],

        'Illuminate\Auth\Events\Failed' => [
            'App\Listeners\LogFailedLogin',
        ],

        'Illuminate\Auth\Events\Logout' => [
            'App\Listeners\LogSuccessfulLogout',
        ],

        'Illuminate\Auth\Events\Lockout' => [
            'App\Listeners\LogLockout',
        ],
    ];
