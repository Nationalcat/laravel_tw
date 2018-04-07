---
layout: post
title: authentication
---
# 認證

- [介紹](#introduction)
    - [資料庫注意事項](#introduction-database-considerations)
- [認證快速入門](#authentication-quickstart)
    - [路由](#included-routing)
    - [視圖](#included-views)
    - [認證](#included-authenticating)
    - [取得已認證之使用者](#retrieving-the-authenticated-user)
    - [保護路由](#protecting-routes)
    - [認證限制](#authentication-throttling)
- [手動認證使用者](#authenticating-users)
    - [記住使用者](#remembering-users)
    - [其他認證方法](#other-authentication-methods)
- [HTTP 基礎認證](#http-basic-authentication)
    - [無狀態 HTTP 基礎認證](#stateless-http-basic-authentication)
- [重設密碼](#resetting-passwords)
    - [資料庫注意事項](#resetting-database)
    - [路由](#resetting-routing)
    - [視圖](#resetting-views)
    - [重設密碼後](#after-resetting-passwords)
- [社群認證](#social-authentication)
- [新增客製化守衛](#adding-custom-guards)
- [新增客製化使用者提供者](#adding-custom-user-providers)
- [事件](#events)

<a name="introduction"></a>
## 介紹

Laravel 讓實作認證變得非常簡單。事實上，幾乎所有東西都是可以藉由設定來直接使用。認證設定檔被放在 `config/auth.php`，其中包含了幾個有良好文件的選項，以此來調整認證服務的行為。

在 Laravel 的核心中，身份驗證工具是由「守衛」和「提供者」所組成。守衛定義了在每個請求中，如何與使用者進行身份驗證。舉例來說，Laravel 內建的一個 `session` 守衛會使用 session 儲存器和 cookies 來維護驗證狀態。然後另一個 `token` 守衛會使用每個請求所傳遞的「API token」來認證使用者。

提供者定義了如何從你的永久性儲存區取得使用者資料。Laravel 內建支援使用 Eloquent 和資料庫查詢產生器來取得使用者資料。然而，你可以自由地根據你的應用程式需要來定義額外的提供者。

如果看到這裡你還覺得一片混亂的話，不用太緊張！大多數的應用程式大概都不需要去修改這預設的驗證設定。

<a name="introduction-database-considerations"></a>
### 資料庫注意事項

預設的 Laravel 會在你的 `app` 資料夾中內建一個 `App\User` [Eloquent 模型](/laravel_tw/docs/5.2/eloquent)。這個模型預設使用 Eloquent 認證驅動。如果你的應用程式沒有使用 Eloquent，你可以改用 `database` 這個使用了 Laravel 查詢生成器的認證驅動。

為 `App\User` 模型建立資料庫結構時，請確保密碼欄位最少有 60 字元長。

同時，你需要確認你的 `users` (或是相同意義的) 資料表含有 nullable、100 字元長的 `remember_token` 欄位，這個欄位將會被你的應用程式用來儲存「記住我」 session 的標記。只要在資料庫遷移檔中使用 `$table->rememberToken()` 即可輕鬆加入這個欄位。

<a name="authentication-quickstart"></a>
## 認證快速入門

Laravel 內建兩個認證控制器，它們被放置在 `App\Http\Controllers\Auth` 命名空間，`AuthController` 處理使用者註冊及認證，而 `PasswordController` 負責處理重置使用者忘記的密碼。這些控制器使用了 trait 來包含所需要的方法，對於大多數的應用程式而言，你並不需要修改這些控制器。

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

現在你已經為認證控制器設定好了路由及視圖，你可以開始讓你的新使用者註冊並認證來登入你的應用程式了。你只要簡單地在瀏覽器存取你定義的路由，認證控制器早已（透過他們各自的 traits）包含了處理認證現有使用者，及儲存新使用者在資料庫的邏輯了。

#### 路徑客製化

當使用者成功的認證後，他們將被重導到 `/home` URI。你可以透過修改 `AuthController` 的 `redirectPath` 屬性自訂認證成功後自動重導的 URI：

    protected $redirectPath = '/dashboard';

當使用者認證失敗，將會被重導到 `/login` URI。你可以設定 `AuthController` 的 `loginPath` 屬性來自訂認證失敗後的自動重導位置：

    protected $loginPath = '/login';

`loginPath` 屬性並不會改變當使用者存取受保護的路由時所自動重導的路徑。該路徑是由 `App\Http\Middleware\Authenticate` 中介層的 `handle` 方法所控制。

#### 資料驗證及儲存的客製化

如果你想要修改一個新使用者註冊時需填寫的表單欄位，或是自訂將使用者的記錄新增到資料庫的方式，你可以修改 `AuthController` 類別，這個類別負責驗證和建立新的使用者。

`AuthController` 的 `validator` 方法包含了對於新使用者的驗證規則。你可以隨意的修改這個方法。

`AuthController` 的 `create` 方法負責使用 [Eloquent ORM](/laravel_tw/docs/5.2/eloquent) 創造新的 `App\User` 記錄到你的資料庫。你可以根據需求任意修改這個方法來符合你資料庫的需求。

<a name="retrieving-the-authenticated-user"></a>
### 取得已認證之使用者

你可以透過 `Auth` facade 來存取認證的使用者。

    $user = Auth::user();

也有另外一種方法可以存取認證過的使用者，就是透過 `Illuminate\Http\Request` 實例：請記得，型別提示的類別將會被自動注入到你控制器的方法中。

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;
    use Illuminate\Routing\Controller;

    class ProfileController extends Controller
    {
        /**
         * 更新使用者的資料
         *
         * @param  Request  $request
         * @return Response
         */
        public function updateProfile(Request $request)
        {
            if ($request->user()) {
                // $request->user() returns an instance of the authenticated user...
            }
        }
    }

#### 確認現在的使用者是否認證過

為了確認使用者是否已經登入，你可以使用 `Auth` facade 的 `check` 方法，如果使用者被認證過，將會回傳 `true`：

    if (Auth::check()) {
        // 這個使用者已經登入...
    }

不過，在允許該使用者存取特定的路由或控制器之前，你可以使用中介層來確認這個使用者是否認證過。想得到更多資訊，請閱讀[保護路由](/laravel_tw/docs/5.2/authentication#protecting-routes)的文件。

<a name="protecting-routes"></a>
### 保護路由

[路由中介層](/laravel_tw/docs/5.2/middleware)使用於限定認證過的使用者存取指定的路由，Laravel 提供了 `auth` 中介層來達到這個目的，而這個中介層被定義在 `app\Http\Middleware\Authenticate.php` 中，你只需要將它附加到路由定義中：

    // 使用路由閉包...

    Route::get('profile', ['middleware' => 'auth', function() {
        // 只有認證過的使用者能進來這裡...
    }]);

    // 使用控制器...

    Route::get('profile', [
        'middleware' => 'auth',
        'uses' => 'ProfileController@show'
    ]);

當然，如果你正在使用[控制器類別](/laravel_tw/docs/5.2/controllers)，你可以在建構子中呼叫 `middleware` 方法，而不是在路由中直接定義它：

    public function __construct()
    {
        $this->middleware('auth');
    }

### 指定認證所用的守衛

當你將 `auth` 這個中介層附加到一個路由時，你也可以指定要哪個守衛應該被使用來處理這次認證：

    Route::get('profile', [
        'middleware' => 'auth:api',
        'uses' => 'ProfileController@show'
    ]);

以上所指定的守衛名稱必須對應到 `auth.php` 設定檔中 `guards` 陣列的其中一個鍵名。

<a name="authentication-throttling"></a>
### 認證限制

如果你使用 Laravel's 內建的 `AuthController` 類別，可以透過 `Illuminate\Foundation\Auth\ThrottlesLogins` trait 在你的應用程式限制登入次數。預設情況下，如果使用者在幾次嘗試後仍不能提供正確的憑證，將在一分鐘內無法進行登入。這個限制會特別針對使用者的用戶名稱 / 郵件地址和他們的 IP 位址：

    <?php

    namespace App\Http\Controllers\Auth;

    use App\User;
    use Validator;
    use App\Http\Controllers\Controller;
    use Illuminate\Foundation\Auth\ThrottlesLogins;
    use Illuminate\Foundation\Auth\AuthenticatesAndRegistersUsers;

    class AuthController extends Controller
    {
        use AuthenticatesAndRegistersUsers, ThrottlesLogins;

        // 省略 AuthController 類別中的其餘程式碼...
    }

<a name="authenticating-users"></a>
## 手動認證使用者

當然，你不一定要使用 Laravel 內建的認證控制器，如果你選擇刪除這些控制器，你將需要自己處理使用者認證，並使用 Laravel 的認證類別。不用怕，這不會很難！

我們將透過 `Auth` [facade](/laravel_tw/docs/5.2/facades) 存取 Laravel 的認證服務，所以我們需要確認是否在類別的最上面引入 `Auth` facade，接下來讓我們看一下 `attempt` 方法：

    <?php

    namespace App\Http\Controllers;

    use Auth;
    use Illuminate\Routing\Controller;

    class AuthController extends Controller
    {
        /**
         * Handle an authentication attempt.
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

`attempt` 方法的第一個參數接受一個鍵值對陣列，在這個陣列的值會用來找尋資料庫裡的使用者資料，所以在上面的範例中，使用者會藉由 `email` 欄位取得，如果使用者被找到了，資料庫裡經過雜湊的密碼將會與陣列中雜湊的 `password` 值做比對，如果兩個雜湊密碼相符的話，會開啟一個通過認證的 session 給使用者。

如果認證成功，`attempt` 方法將會回傳 `true`，反之則為 `false`。

重導器上的 `intended` 方法將會重導使用者回原本想要進入的頁面，也可以傳入一個備用 URI 至這個方法，以避免要導回的頁面無法使用。

#### 指定額外的條件

如果你希望，你也可以加入除了使用者的電子信箱及密碼的額外條件至認證查詢。例如，我們要確認使用者是否被標記為「active」：

    if (Auth::attempt(['email' => $email, 'password' => $password, 'active' => 1])) {
        // 這個使用者是存在且有效的，沒有被停權
    }

> **注意：**在這些例子中，`email` 不是一個必要的選項，它只是被用來當作範例。你可以使用資料庫中任何等同於「使用者名稱」的欄位。

#### 存取特定的守衛實例

你可以在使用 `Auth` facade 時使用 `guard` 方法來指定守衛實例。這允許你在管理應用程式的不同部份時，使用完全不同的認證模組或是使用者的資料表。

`guard` 方法所指定的守衛名稱必須對應到 `auth.php` 設定檔中 `guards` 陣列的其中一個鍵名。

    if (Auth::guard('admin')->attempt($credentials)) {
        //
    }

#### 登出

為了讓使用者登出，你可以使用 `Auth` facade 的 `logout` 方法。這個方法會清除使用者在 session 中所有認證相關的資料：

    Auth::logout();

<a name="remembering-users"></a>
## 記住使用者

如果你想要提供「記住我」的功能，你需要傳入一個布林值到 `attempt` 方法的第二個參數，這會永久保持使用者的 session 直到使用者手動登出。你的 `users` 資料表一定要包含一個 `remember_token` 欄位，這是用來儲存「記住我」的標記。

    if (Auth::attempt(['email' => $email, 'password' => $password], $remember)) {
        // The user is being remembered...
    }

如果你是被記住的使用者，你可以使用 `viaRemember` 方法來確認這個使用者是否使用「記住我」 cookie 來做認證：

    if (Auth::viaRemember()) {
        //
    }

<a name="other-authentication-methods"></a>
### 其他認證方法

#### 用使用者實例做認證

如果你需要使用存在的使用者實例來登入，你需要呼叫 `login` 方法，並傳入使用實例，這個物件必須實作 `Illuminate\Contracts\Auth\Authenticatable` [contract](/laravel_tw/docs/5.2/contracts)。當然，內建的 `App/User` 模型已經實作了這個介面：

    Auth::login($user);

#### 用使用者 ID 做認證

如果你需要使用使用者的 ID 來登入，你需要使用 `loginUsingId` 方法，這個方法只接受要登入的使用者的主鍵：

    Auth::loginUsingId(1);

#### 一次性的使用者認證

你可以使用 `once` 方法來只針對一次的請求來認證使用者，沒有任何的 session 或 cookie 會被使用，這個對於建議無狀態的 API 非常的有用。`once` 方法跟 `attempt` 方法擁有同樣的傳入參數：

    if (Auth::once($credentials)) {
        //
    }

<a name="http-basic-authentication"></a>
## HTTP 基礎認證

[HTTP 基礎認證](http://en.wikipedia.org/wiki/Basic_access_authentication)提供一個快速的方法來認證使用者，不需要任何「登入」頁面。想要使用這功能，要先增加 `auth.basic` [中介層](/laravel_tw/docs/5.2/middleware)到你的路由上，`auth.basic` 中介層已經被內建在 Laravel 框架中，所以你不需要定義它：

    Route::get('profile', ['middleware' => 'auth.basic', function() {
        // 只有被認證過的使用者才可以進入...
    }]);

一旦中介層被增加到路由上，當使用瀏覽器進入這個路由時，會自動提示你需要提供憑證。預設上，`auth.basic` 中介層將會使用使用者的 `email` 欄位當作「使用者名稱」。

#### FastCGI 的注意事項

如果是正在使用 FastCGI，HTTP 基礎認證可能無法正常運作，你需要將下面這幾行設定加入你 `.htaccess` 檔案中：

    RewriteCond %{HTTP:Authorization} ^(.+)$
    RewriteRule .* - [E=HTTP_AUTHORIZATION:%{HTTP:Authorization}]

<a name="stateless-http-basic-authentication"></a>
### 無狀態 HTTP 基礎認證

你可以使用 HTTP 基礎認證而不用在 session 中設定使用者認證用的 cookie，這個功能對 API 認證來說非常有用。為了達到這個目的，[定義一個中介層](/laravel_tw/docs/5.2/middleware)並這個中介層會呼叫 `onceBasic` 方法。如果沒有任何回應從 `onceBasic` 方法返回的話，這個請求就會進一步傳進應用程式中：

    <?php

    namespace Illuminate\Auth\Middleware;

    use Auth;
    use Closure;

    class AuthenticateOnceWithBasicAuth
    {
        /**
         * 處理請求
         *
         * @param  \Illuminate\Http\Request  $request
         * @param  \Closure  $next
         * @return mixed
         */
        public function handle($request, Closure $next)
        {
            return Auth::onceBasic() ?: $next($request);
        }

    }

接著，[註冊這個路由中介層](/laravel_tw/docs/5.2/middleware#registering-middleware)，然後將它增加在一個路由上：

    Route::get('api/user', ['middleware' => 'auth.basic.once', function() {
        // 只有認證過的使用者可以進入...
    }]);

<a name="resetting-passwords"></a>
## 重設密碼

<a name="resetting-database"></a>
### 重設資料庫

大部分的網頁應用程式都會提供一些方式讓使用者來重設他們忘記的密碼。Laravel 提供便利的方法來傳送密碼提示和實作密碼重設，而不是強迫你重新實作這些功能。

開始之前，請先確認你的 `App\User` 模型實作了 `Illuminate\Contracts\Auth\CanResetPassword` contract。當然，原有的 `App\User` 早已實作了這個介面，並且使用 `Illuminate\Auth\Passwords\CanResetPassword` trait 引入實作這個介面所需要的方法。

#### 產生重置標記的資料表遷移檔

接下來，必須要創建一個資料表來儲存密碼的重置標記，而這個資料表的遷移已經包含在 Laravel 中了，就被放在 `database/migrations` 資料夾裡。所以，你要做的就是做一次遷移：

    php artisan migrate

<a name="resetting-routing"></a>
### 路由

Laravel 內建了含有所有重置使用者密碼邏輯的 `Auth\PasswordController`。所有重設密碼功能所需的路由都可以透過 `make:auth` Artisan 指令自動產生：

    php artisan make:auth

<a name="resetting-views"></a>
### 視圖

你只要執行 `make:auth` 以後，Laravel 將會幫你產生所有重設密碼所需的視圖，這些視圖都會被放在 `resources/views/auth/passwords` 內，你可以自由地修改它們以符合你的需求。

<a name="after-resetting-passwords"></a>
### 重設密碼後

一旦你定義了路由跟視圖來重置使用者的密碼，你只需要在瀏覽器中打開 `/password/reset` 來存取這個路由。Laravel 中的 `PasswordController` 已經包含了傳送密碼重置連結的電子郵件，及更新密碼到資料庫的邏輯。

密碼重置以後，這個使用者會自動登入，並重導到 `/home`。你可以自己客製化重導的目標，只要定義 `PasswordController` 的 `redirectTo` 屬性：

    protected $redirectTo = '/dashboard';

> **注意：**預設的密碼重置標記會在一個小時後過期，你可以透過更改 `config/auth.php` 的 `expire` 選項來修改這個設定。

<a name="social-authentication"></a>
## 社群認證

除了傳統的表單認證，Laravel 同樣提供了簡單方便的方法來認證 OAuth 提供者，這個方法使用了 [Laravel Socialite](https://github.com/laravel/socialite)。Socialite 目前支援 Facebook、Twitter、LinkedIn、Google、GitHub 跟 Bitbucket。

開始使用 Socialite 前，新增依賴套件至你的 `composer.json`：

    composer require laravel/socialite

### 設定

安裝 Socialite 之後，到 `config/app.php` 設定檔中註冊 `Laravel\Socialite\SocialiteServiceProvider`：

    'providers' => [
        // 其他服務提供者...

        Laravel\Socialite\SocialiteServiceProvider::class,
    ],

同樣的，新增 `Socialite` facade 到 `app` 設定檔的 `aliases` 陣列中：

    'Socialite' => Laravel\Socialite\Facades\Socialite::class,

你將需要新增憑證來使用 OAuth 服務，這些憑證需要被放在 `config/services.php` 設定檔，並根據你應用程式的需求，增加 `facebook`、`twitter`、`linkedin`、`google`、`github` 或 `bitbucket` 的鍵，例如：

    'github' => [
        'client_id' => 'your-github-app-id',
        'client_secret' => 'your-github-app-secret',
        'redirect' => 'http://your-callback-url',
    ],

### 基礎應用

接下來，你已經準備好開始認證使用者了！你將需要兩個路由: 一個用來重導使用者到 OAuth 提供者，另一個在認證後接收提供者的回呼。我們將會藉由 `Socialite` [facade](/laravel_tw/docs/5.2/facades) 存取 Socialite：

    <?php

    namespace App\Http\Controllers;

    use Socialite;
    use Illuminate\Routing\Controller;

    class AuthController extends Controller
    {
        /**
         * 重導使用者到 GitHub 認證頁。
         *
         * @return Response
         */
        public function redirectToProvider()
        {
            return Socialite::driver('github')->redirect();
        }

        /**
         * 從 Github 得到使用者資訊
         *
         * @return Response
         */
        public function handleProviderCallback()
        {
            $user = Socialite::driver('github')->user();

            // $user->token;
        }
    }

`redirect` 方法會負責處理傳送使用者到 OAuth 提供者，而 `user` 方法會從提供者返回的請求來取得使用者資訊。在重導使用者之前，你也可以使用 `scopes` 方法來設定請求的 「作用域」。這個方法將覆寫所有已經存在的作用域：

    return Socialite::driver('github')
                ->scopes(['scope1', 'scope2'])->redirect();

當然，你需要定義路由到你的控制器方法：

    Route::get('auth/github', 'Auth\AuthController@redirectToProvider');
    Route::get('auth/github/callback', 'Auth\AuthController@handleProviderCallback');

一些 OAuth 提供者支援在重導的請求中自訂參數。若要在請求中加入任何自訂參數，只要呼叫 `with` 方法並帶上一個關聯陣列：

    return Socialite::driver('google')
                ->with(['hd' => 'example.com'])->redirect();

#### 取得使用者細節

一旦你有了使用者的實例，你可以獲取使用者更細節的資訊:

    $user = Socialite::driver('github')->user();

    // OAuth Two 提供者
    $token = $user->token;

    // OAuth One 提供者
    $token = $user->token;
    $tokenSecret = $user->tokenSecret;

    // 所有提供者
    $user->getId();
    $user->getNickname();
    $user->getName();
    $user->getEmail();
    $user->getAvatar();

<a name="adding-custom-guards"></a>
## 新增客製化的認證驅動

你可以使用 `Auth` facade 中的 `extend` 方法來定義屬於你自己的身份認證守衛。你應該把這段程式碼放在[服務提供者](/laravel_tw/docs/5.2/providers)的 `provider` 中：

    <?php

    namespace App\Providers;

    use Auth;
    use App\Services\Auth\JwtGuard;
    use Illuminate\Support\ServiceProvider;

    class AuthServiceProvider extends ServiceProvider
    {
        /**
         * 在服務啟動後進行註冊
         *
         * @return void
         */
        public function boot()
        {
            Auth::extend('jwt', function($app, $name, array $config) {
                // 回傳 Illuminate\Contracts\Auth\Guard 的實例...

                return new JwtGuard(Auth::createProvider($config['provider']));
            });
        }

        /**
         * 在容器中註冊綁定項目
         *
         * @return void
         */
        public function register()
        {
            //
        }
    }

你可以看到上面的範例中，傳遞給 `extend` 方法的回呼函式必須要回傳 `Illuminate\Contracts\Auth\Guard` 的實例。你需要實作這個介面中定義的所有方法來定義一個客製化的守衛。

一旦你客製化的守衛被定義好之後，你可以在你的 guards 設定檔中使用你的守衛：

    'guards' => [
        'api' => [
            'driver' => 'jwt',
            'provider' => 'users',
        ],
    ],

<a name="adding-custom-user-providers"></a>
## 新增客製化使用者提供者

如果你不是使用傳統的關聯式資料庫來儲存使用者，你將需要擴充 Laravel 來新增你自己的認證使用者提供者。我們將使用 `Auth` facade 的 `provider` 方法來定義客製化使用者提供者。你應該把這段程式碼放置在[服務提供者](/laravel_tw/docs/5.2/providers)的 `provider` 中：

    <?php

    namespace App\Providers;

    use Auth;
    use App\Extensions\RiakUserProvider;
    use Illuminate\Support\ServiceProvider;

    class AuthServiceProvider extends ServiceProvider
    {
        /**
         * 在服務啟動後進行註冊
         *
         * @return void
         */
        public function boot()
        {
            Auth::provider('riak', function($app, array $config) {
                // 回傳一個 Illuminate\Contracts\Auth\UserProvider 的實例...
                return new RiakUserProvider($app['riak.connection']);
            });
        }

        /**
         * 在容器中註冊綁定項目
         *
         * @return void
         */
        public function register()
        {
            //
        }
    }

在你使用了 `provider` 註冊了服務提供者後，你就可以在 `config/auth.php` 設定檔中改用你新的使用者提供者。首先，定義一個使用你的新驅動的 `provider`：

    'providers' => [
        'users' => [
            'driver' => 'riak',
        ],
    ],

然後，你可以在 `guards` 設定中使用這個服務提供者。

    'guards' => [
        'web' => [
            'driver' => 'session',
            'provider' => 'users',
        ],
    ],

### 使用者提供者 Contract

`Illuminate\Contracts\Auth\UserProvider` 的實例只負責取得 `Illuminate\Contracts\Auth\Authenticatable` 的實例，且不受限於永久儲存系統，例如 MySQL, Riak 等等。這兩個介面允許 Laravel 認證機制不用管使用者是如何儲存或是該用什麼類型的類別來代表它。

讓我們來看看 `Illuminate\Contracts\Auth\UserProvider` contract：

    <?php

    namespace Illuminate\Contracts\Auth;

    interface UserProvider {

        public function retrieveById($identifier);
        public function retrieveByToken($identifier, $token);
        public function updateRememberToken(Authenticatable $user, $token);
        public function retrieveByCredentials(array $credentials);
        public function validateCredentials(Authenticatable $user, array $credentials);

    }

`retrieveById` 函式通常接收一個代表使用者的值，例如 MySQL 中自動增加的 ID。這個方法應該要取得並回傳匹配這個 ID 的 `Authenticatable` 的實例。

`retrieveByToken` 函式藉由使用者獨特的 `$identifier` 和儲存在 `remember_token` 欄位的「記住我」`$token` 來檢索使用者。如同之前的方法，`Authenticatable` 的實例應該被回傳。

`updateRememberToken` 方法使用新的 `$token` 更新了 `$user` 的 `remember_token` 欄位。這個新的標記可以是全新的標記（當使用「記住我」嘗試登入成功時），或是 null（當使用者登出時）。

`retrieveByCredentials` 方法接收在登入應用程式時傳遞給 `Auth::attempt` 方法的憑證陣列。這個方法應該要從底層的永久式儲存系統中「查詢」匹配這個憑證的使用者。通常，這個方法會利用 `$credentials['username']` 來執行一個帶著「where」條件的查詢。接著這個方法需要回傳一個 `UserInterface` 的實例。**這個方法不應該企圖做任何密碼的驗證或是認證。**

`validateCredentials` 方法應該要拿 `$credentials` 與 `$user` 進行核對來認證這個使用者。例如，這個方法可能會比較 `$user->getAuthPassword()` 字串及 `Hash::make` 後的 `$credentials['password']`。這個方法應該只驗證使用者的憑證並回傳一個布林值。

### The Authenticatable Contract

現在我們已經介紹了 `UserProvider` 的每個方法，讓我們看一下 `Authenticatable` contract。再次提醒，`UserProvider` 的 `retrieveById` 和 `retrieveByCredentials` 方法需要回傳 `Authenticatable` 的實例：

    <?php

    namespace Illuminate\Contracts\Auth;

    interface Authenticatable {

        public function getAuthIdentifier();
        public function getAuthPassword();
        public function getRememberToken();
        public function setRememberToken($value);
        public function getRememberTokenName();

    }

這個介面很簡單。`getAuthIdentifier` 方法需要回傳使用者的「主鍵」。在 MySQL，這個主鍵是指自動增加的主鍵。而 `getAuthPassword` 應該要回傳使用者雜湊後的密碼。這個介面允許認證系統與任何使用者類別合作，不用管你在使用何種 ORM 或是儲存抽象層。Laravel 的 `app` 資料夾中內建了實作這個介面的 `User` 類別，所以你可以觀察這個類別作為實作的範例。

<a name="events"></a>
## 事件

Laravel 提供了在認證過程中的各種[事件](/laravel_tw/docs/5.2/events)。你可以在 `EventServiceProvider` 為這些事件附加監聽器：

    /**
     * 為你的應用程式註冊任何事件。
     *
     * @var array
     */
    protected $listen = [
        'Illuminate\Auth\Events\Attempting' => [
            'App\Listeners\LogAuthenticationAttempt',
        ],

        'Illuminate\Auth\Events\Login' => [
            'App\Listeners\LogSuccessfulLogin',
        ],

        'Illuminate\Auth\Events\Logout' => [
            'App\Listeners\LogSuccessfulLogout',
        ],
    ];
