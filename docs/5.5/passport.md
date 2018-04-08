---
layout: post
title: passport
tag: 5.5
---
# API 認證（Passport）

- [介紹](#introduction)
- [安裝](#installation)
    - [前端快速入門](#frontend-quickstart)
    - [部署 Passport](#deploying-passport)
- [設定](#configuration)
    - [Token 有效期限](#token-lifetimes)
- [發放 Access Token](#issuing-access-tokens)
    - [管理客戶端](#managing-clients)
    - [請求 Token](#requesting-tokens)
    - [更新 Token](#refreshing-tokens)
- [密碼授權 Token](#password-grant-tokens)
    - [建立一個密碼授權的客戶端](#creating-a-password-grant-client)
    - [請求 Token](#requesting-password-grant-tokens)
    - [請求所有範圍](#requesting-all-scopes)
- [隱式授權 Token](#implicit-grant-tokens)
- [客戶端憑證授權 Token](#client-credentials-grant-tokens)
- [個人的 Access Token](#personal-access-tokens)
    - [建立個人的存取客戶端](#creating-a-personal-access-client)
    - [管理個人的 Access Token](#managing-personal-access-tokens)
- [保護路由](#protecting-routes)
    - [透過中介層](#via-middleware)
    - [傳入 Access Token](#passing-the-access-token)
- [Token Scope](#token-scopes)
    - [定義 Scope](#defining-scopes)
    - [將 Scope 分配到 Token](#assigning-scopes-to-tokens)
    - [檢查 Scope](#checking-scopes)
- [搭配 JavaScript 使用你的 API](#consuming-your-api-with-javascript)
- [事件](#events)
- [測試](#testing)

<a name="introduction"></a>
## 介紹

Laravel 已經讓傳統登入表單來更容易地進行認證，但是 API 認證的部份呢？API 通常使用 Token 來認證使用者，並不是維護在請求之間的 Session 狀態。Laravel 使用 Laravel Passport 來輕易的做到 API 認證，Passport 可以在幾分鐘內為你的 Laravel 應用程式提供一個完整的 OAuth2 伺服器實作。Passport 是建構在 Alex Bilbie 維護的 [League OAuth2 server](https://github.com/thephpleague/oauth2-server)的基礎上。

> {note} 本文件已假設你已經了解 OAuth2 了。如果你對 OAuth2 不是很了解，請在繼續閱讀之前先熟悉 OAuth2 的一般術語和功能。

<a name="installation"></a>
## 安裝

來開始透過 Composer 套件管理器安裝 Passport：

    composer require laravel/passport

Passport 服務提供者在框架中已註冊好本身的資料庫遷移目錄，所以你應該在遷移資料庫之後註冊這個提供者。Passport 的遷移檔會建立應用程式需要儲存客戶端與 Access Token 的資料表：

    php artisan migrate

> {note} 如果你不想使用 Passport 的預設資料表結構，你應該在 `AppServiceProvider` 的 `register` 方法中呼叫 `Passport::ignoreMigrations` 方法。你可以使用 `php artisan vendor:publish --tag=passport-migrations` 導出該預設資料表結構。

接下來，你應該執行 `passport:install` 指令。這個指令會建立用來產生安全 Access Token 的加密金鑰。此外，該指令會建立用於產生 Access Token 的「個人存取」與「密碼授權」的客戶端：

    php artisan passport:install

執行這個指令後，新增 `Laravel\Passport\HasApiTokens` trait 到你的 `App\User` 。這個 trait 會提供一些輔助方法到你的模型，這可以讓你檢查被認證過的使用者的 Token 與 Scope：

    <?php

    namespace App;

    use Laravel\Passport\HasApiTokens;
    use Illuminate\Notifications\Notifiable;
    use Illuminate\Foundation\Auth\User as Authenticatable;

    class User extends Authenticatable
    {
        use HasApiTokens, Notifiable;
    }

接著，你應該呼叫 `Passport::routes` 方法在你的 `AuthServiceProvider` 中的 `boot` 方法。這個方法會註冊該路由必須要發放 Access Token 和取消存取 Toekn、客戶端和個人存取的 Token：

    <?php

    namespace App\Providers;

    use Laravel\Passport\Passport;
    use Illuminate\Support\Facades\Gate;
    use Illuminate\Foundation\Support\Providers\AuthServiceProvider as ServiceProvider;

    class AuthServiceProvider extends ServiceProvider
    {
        /**
         * 應用程式的 policy 映射。
         *
         * @var array
         */
        protected $policies = [
            'App\Model' => 'App\Policies\ModelPolicy',
        ];

        /**
         * 註冊任何認證與授權的服務。
         *
         * @return void
         */
        public function boot()
        {
            $this->registerPolicies();

            Passport::routes();
        }
    }

最後，在你的 `config/auth.php` 設定檔，你應該將認證 Guard 的 `api` 中的 `driver` 選項設定為 `passport`。這會指示應用程式在正要認證剛傳入的 API 請求時使用 Passport 的 `TokenGuard`：

    'guards' => [
        'web' => [
            'driver' => 'session',
            'provider' => 'users',
        ],

        'api' => [
            'driver' => 'passport',
            'provider' => 'users',
        ],
    ],

<a name="frontend-quickstart"></a>
### 前端快速入門

> {note} 為了要使用 Vue 的 Passport 元件，你必須使用 [Vue](https://vuejs.org) JavaScript 框架。這些元件也用到 Bootstrap CSS 框架。然而，甚至如果你不使用這些工具，這些元件對於實作你自己的前端是具有非常高的參考價值。

Passport 附帶了一個 JSON API，你可以用來讓你的使用者建立客戶端或個人存取的 Token。然而，撰寫前端與這些 API 交換資料是非常浪費時間。所以，Passport 還引入了預先寫好的 [Vue](https://vuejs.org) 元件，你可以用它來作為你實作這些的起點。

要發佈 Vue 的 Passport 元件，使用 Artisan 的 `vendor:publish` 指令：

    php artisan vendor:publish --tag=passport-components

已發佈的元件會放置在 `resources/assets/js/components` 目錄中。元件一旦被發佈，你應該在 `resources/assets/js/app.js` 檔案中註冊它們：

    Vue.component(
        'passport-clients',
        require('./components/passport/Clients.vue')
    );

    Vue.component(
        'passport-authorized-clients',
        require('./components/passport/AuthorizedClients.vue')
    );

    Vue.component(
        'passport-personal-access-tokens',
        require('./components/passport/PersonalAccessTokens.vue')
    );

註冊元件後，請確實執行 `npm run dev` 來重新編譯你的前端資源。一旦重新編譯好你的前端資源，你就可以將元件放到你應用程式的其中一個模板中，接著建立客戶端和個人存取的 Token：

    <passport-clients></passport-clients>
    <passport-authorized-clients></passport-authorized-clients>
    <passport-personal-access-tokens></passport-personal-access-tokens>

<a name="deploying-passport"></a>
### 部署 Passport

當第一次部署 Passport 到你的線上伺服器時，你將需要執行 `passport:keys` 指令。這個指令產生這個指令會為了產生 Access Token 而去產生 Passport 所需要的加密金鑰。該產生的金鑰通常不會存放在版控系統中：

    php artisan passport:keys

<a name="configuration"></a>
## 設定

<a name="token-lifetimes"></a>
### Token 有效期限

預設的 Passport 會發放長期存取的 Token，且永不需要重新再發一次。如果你想要縮短 Token 有效期限的設定。你可以使用 `tokensExpireIn` 和 `refreshTokensExpireIn` 方法。這些方法會從 `AuthServiceProvider` 的 `boot` 方法中被呼叫：

    use Carbon\Carbon;

    /**
     * 註冊任何認證與授權服務。
     *
     * @return void
     */
    public function boot()
    {
        $this->registerPolicies();

        Passport::routes();

        Passport::tokensExpireIn(Carbon::now()->addDays(15));

        Passport::refreshTokensExpireIn(Carbon::now()->addDays(30));
    }

<a name="issuing-access-tokens"></a>
## 發放 Access Token

使用具有授權碼的 OAuth2 是大多數開發者所熟悉的 OAuth2 方式。在使用認證授權碼時，客戶端的應用程式會將使用者重導到你的伺服器，他們將同意或拒絕向客戶端發出 Access Token 的請求。

<a name="managing-clients"></a>
### 管理客戶端

首先，開發者若要建構與 API 交換資料的應用程式，會需要依照自己建立的「客戶端」來註冊應用程式。通常認證會在使用者同意請求之後，提供其應用程式的名稱和應用程式要被重導到的 URL。

#### `passport:client` 指令

建立客戶端的最簡單方式就是使用 Artisan 的 `passport:client` 指令。這個指令可被用於建立自己的客戶端來測試 OAuth2 的功能。當你執行 `client` 指令時，Passport 會提示你更多與客戶端有關的資訊，並提供你客戶端的 ID 和 Secret：

    php artisan passport:client

#### JSON API

由於你的使用者會無法使用 `client` 指令，所以 Passport 提供一個可用於建立客戶端的 JSON API。這個為你省去手動撰寫用於建立、更新和刪除客戶端的控制器的麻煩。

然而，你會需要把 Passport JSON API 與自己前端配對，就為了你的使用者要管理他們客戶端而提供一個儀表板。以下，我們會回顧所有用於管理客戶端的 API 端點。為了方便講解，我們會使用 [Axios](https://github.com/mzabriskie/axios) 來示範如何向該端點發送 HTTP 請求。

> {tip} 如果你不想實作整個客戶端管理前端介面，你能用[前端快速入門](#frontend-quickstart) 在幾分鐘內擁有一套功能的前端介面。

#### `GET /oauth/clients`

這個路由回傳了已認證使用者的所有客戶端。這對於列出所有使用者的客戶端來說是非常的有用，藉此他們可以編輯或刪除它們：

    axios.get('/oauth/clients')
        .then(response => {
            console.log(response.data);
        });

#### `POST /oauth/clients`

這個路由被用於建立新的客戶端。它需要兩種資料：客戶端的 `name` 和 `redirect` URL。`redirect` URL 是在被同意或拒絕授權請求之後，使用者要被重導的位置。

當客戶端被建立時，它會被發一組客戶端 ID 和 Secret。這些值會被用於從你應用程式中請求一組 Access Token 的時候。客戶端的建立路由會回傳新的客戶端實例：

    const data = {
        name: 'Client Name',
        redirect: 'http://example.com/callback'
    };

    axios.post('/oauth/clients', data)
        .then(response => {
            console.log(response.data);
        })
        .catch (response => {
            // 列出回應中的錯誤...
        });

#### `PUT /oauth/clients/{client-id}`

這個路由被用於更新客戶端。它需要兩種資料：客戶端的 `name` 和 `redirect` URL。`redirect` URL 是使用者在被同意或拒絕授權請求之後所要被重導的位置。該路由會回傳已更新過的客戶端實例：

    const data = {
        name: 'New Client Name',
        redirect: 'http://example.com/callback'
    };

    axios.put('/oauth/clients/' + clientId, data)
        .then(response => {
            console.log(response.data);
        })
        .catch (response => {
            // 列出回應中的錯誤...
        });

#### `DELETE /oauth/clients/{client-id}`

這個路由被用於刪除客戶端：

    axios.delete('/oauth/clients/' + clientId)
        .then(response => {
            //
        });

<a name="requesting-tokens"></a>
### 請求 Token

#### 為授權而重導

客戶端一旦被建立，開發者就可以從應用程式中使用他們的客戶端 ID 和 Secret 來請求一個授權碼和 Access Token。首先，正在使用的使用者會向你應用程式 `/oauth/authorize` 路由的發出重導的請求，像是：

    Route::get('/redirect', function () {
        $query = http_build_query([
            'client_id' => 'client-id',
            'redirect_uri' => 'http://example.com/callback',
            'response_type' => 'code',
            'scope' => '',
        ]);

        return redirect('http://your-app.com/oauth/authorize?'.$query);
    });

> {tip} 請記得，`/oauth/authorize` 路由已經在 `Passport::routes` 方法中定義。你不需要手動定義這個路由。

#### 同意請求

當接受到授權請求時，Passport 會自動顯示一個模板給使用者，讓他們同意或拒絕授權請求。如果他們同意該請求，他們將會被重導回正使用的應用程式所指定的 `redirect_uri`。`redirect_uri` 必須在建立客戶端時與指定的 `redirect` URL 匹配。

如果你想要自訂授權同意畫面，你可以使用 Artisan 的 `vendor:publish` 指令來發佈 Passport 的視圖。剛發佈的視圖會放置在 `resources/views/vendor/passport`：

    php artisan vendor:publish --tag=passport-views

#### 將授權碼轉換成 Access Token

如果使用者同意授權請求，他們會被重導回正在使用的應用程式。使用者應該像你的應用程式發出 `POST` 請求來請求一組 Access Token。該請求會包含使用者同意授權請求時由你的應用程式所發的授權碼。在本範例中，我們會使用 Guzzle HTTP 函式庫來進行 `POST` 請求：

    Route::get('/callback', function (Request $request) {
        $http = new GuzzleHttp\Client;

        $response = $http->post('http://your-app.com/oauth/token', [
            'form_params' => [
                'grant_type' => 'authorization_code',
                'client_id' => 'client-id',
                'client_secret' => 'client-secret',
                'redirect_uri' => 'http://example.com/callback',
                'code' => $request->code,
            ],
        ]);

        return json_decode((string) $response->getBody(), true);
    });

`/oauth/token` 路由會回傳一組 JSON 回應，並包含 `access_token`、`refresh_token` 和 `expires_in` 屬性。`expires_in` 屬性會包含 Access Token 過期之前的秒數。

> {tip} 像是 `/oauth/authorize` 路由，`/oauth/token` 路由是由 `Passport::routes` 方法來定義。這裡不需要手動定義這個路由。

<a name="refreshing-tokens"></a>
### 更新 Token

如果你的應用程式發了一組短期的 Access Token，使用者會需要在發 Access Token 時提供新的 Access Token 來更新他們的 Access Token。在本範例中，我們會使用 Guzzle HTTP 函式庫來更新 Token：

    $http = new GuzzleHttp\Client;

    $response = $http->post('http://your-app.com/oauth/token', [
        'form_params' => [
            'grant_type' => 'refresh_token',
            'refresh_token' => 'the-refresh-token',
            'client_id' => 'client-id',
            'client_secret' => 'client-secret',
            'scope' => '',
        ],
    ]);

    return json_decode((string) $response->getBody(), true);

`/oauth/token` 這個路由會回傳一組 JSON 回應，並包含 `access_token`、`refresh_token` 和 `expires_in` 屬性。`expires_in` 屬性包含 Access Token 過期之前的秒數。

<a name="password-grant-tokens"></a>
## 密碼授權 Token

OAuth2 密碼授權可以讓你的其他第一方客戶端可以使用者電子信箱或使用者名稱和密碼來獲得 Access Token，像是行動版應用程式。這可以讓你安全的將 Access Token 發給第一方的客戶端，且不需要你的使用者經過整個 OAuth2 授權碼的重導流程。

<a name="creating-a-password-grant-client"></a>
### 建立一個密碼授權的客戶端

在應用程式能透過密碼授權來發放 Token 之前，你會需要建立一個密碼授權客戶端。你可以使用 `passport:client` 指令並搭配 `--password` 選項來做到。如果你已經執行過 `passport:install` 指令，你就不需要再執行這個指令：

    php artisan passport:client --password

<a name="requesting-password-grant-tokens"></a>
### 請求 Token

你一旦建立了密碼授權客戶端，你可以向使用者的電子信箱和密碼發送一個 `POST` 請求到 `/oauth/token` 路由來請求一組 Access Token。請記得，這個路由已經被 `Passport::routes` 方法註冊，所以這裡不需要再手動定義它。如果請求成功，你會從伺服器的 JSON 回應中收到一組 `access_token` 和 `refresh_token`：

    $http = new GuzzleHttp\Client;

    $response = $http->post('http://your-app.com/oauth/token', [
        'form_params' => [
            'grant_type' => 'password',
            'client_id' => 'client-id',
            'client_secret' => 'client-secret',
            'username' => 'taylor@laravel.com',
            'password' => 'my-password',
            'scope' => '',
        ],
    ]);

    return json_decode((string) $response->getBody(), true);

> {tip} 記得，Access Token 預設的有效期限是長期的。然而，你可以自由的依據需求來[設定 Access Tokne 的最大有效期限](#configuration)。

<a name="requesting-all-scopes"></a>
### 請求所有 Scope

當使用密碼授權的時候，你可能希望為你的應用程式支援所有範圍授權 Token。你可以透過請求 `*` scope 來完成。如果你使用了 `*` 作為 scope 的值，Token 實例上的 `can` 方法就總是會回傳 `true`。這個範圍可以只可以被分配給一個使用 `password` 授權的 Token：

    $response = $http->post('http://your-app.com/oauth/token', [
        'form_params' => [
            'grant_type' => 'password',
            'client_id' => 'client-id',
            'client_secret' => 'client-secret',
            'username' => 'taylor@laravel.com',
            'password' => 'my-password',
            'scope' => '*',
        ],
    ]);

<a name="implicit-grant-tokens"></a>
## 隱式授權 Token

隱式授權類似於授權碼授權。然而，該 Token 回傳給客戶端並不會交換授權碼。這種授權最常被用於無法安全地儲存客戶端憑證的 JavaScript 或行動版應用程式。要啟用該授權，請在你的 `AuthServiceProvider` 中呼叫 `enableImplicitGrant` 方法：

    /**
     * 註冊任何認證與授權服務。
     *
     * @return void
     */
    public function boot()
    {
        $this->registerPolicies();

        Passport::routes();

        Passport::enableImplicitGrant();
    }

授權一旦被啟用，開發者就可以從應用程式中使用它們的客戶端 ID 來請求一組 Access Token。正在使用的應用程式會請求重導到應用程式的 `/oauth/authorize` 路由，就像是：

    Route::get('/redirect', function () {
        $query = http_build_query([
            'client_id' => 'client-id',
            'redirect_uri' => 'http://example.com/callback',
            'response_type' => 'token',
            'scope' => '',
        ]);

        return redirect('http://your-app.com/oauth/authorize?'.$query);
    });

> {tip} 請記得，`/oauth/authorize` 路由已經被 `Passport::routes` 方法所定義。你就不需要再手動定義這個路由。

<a name="client-credentials-grant-tokens"></a>
## 客戶端憑證授權 Token

客戶端憑證授權適用於機器對機器的授權。例如，你可以在 API 執行維護的排程任務中使用此授權。要使用這個方法，首先你需要在 `app/Http/Kernel.php` 的 `routeMiddleware` 中新增中介層：

    use Laravel\Passport\Http\Middleware\CheckClientCredentials;

    protected $routeMiddleware = [
        'client' => CheckClientCredentials::class,
    ];

然後將這個中介層附加到路由上：

    Route::get('/user', function(Request $request) {
        ...
    })->middleware('client');

要接收一個 Token，請向 `oauth/token` 端點發出請求：

    $guzzle = new GuzzleHttp\Client;

    $response = $guzzle->post('http://your-app.com/oauth/token', [
        'form_params' => [
            'grant_type' => 'client_credentials',
            'client_id' => 'client-id',
            'client_secret' => 'client-secret',
            'scope' => 'your-scope',
        ],
    ]);

    return json_decode((string) $response->getBody(), true)['access_token'];

<a name="personal-access-tokens"></a>
## 個人的 Access Token

有些時候，你的使用者可能希望發放 Access Token 給他們自己，而不用透過典型授權碼重導流程。可以讓使用者透過應用程式的 UI 來發放 Token 給他們自己，在一般狀況來說，這有助於對使用者體驗你的 API 或者作為發放 Access Token 的簡單的方式。

> {note} 個人的 Access Token 的有效期限也是長期的。當你使用 `tokensExpireIn` 或 `refreshTokenExpireIn` 方法時，它們的生命週期不會被修改。

<a name="creating-a-personal-access-client"></a>
### 建立個人存取客戶端

在你的應用程式能夠發放個人的 Access Token 之前，你會需要建立個人的存取客戶端。你可以使用 `passport:client` 指令並搭配 `--personal` 選項來做到。如果你已經執行過 `passport:install` 指令，你就不需要再執行這個指令了：

    php artisan passport:client --personal

<a name="managing-personal-access-tokens"></a>
### 管理個人的 Access Token

你一旦建立了個人的存取客戶端，你可以在 `User` 模型實例上使用 `createToken` 方法來為給定使用者發放 Token。`createToken` 方法接受 Token 的名稱作為該方法的第一個參數，並將 [scopes](#token-scopes) 的一組可選陣列作為它的第二個參數：

    $user = App\User::find(1);

    // 建立沒有範圍的 Token...
    $token = $user->createToken('Token Name')->accessToken;

    // 建立帶有範圍的 Token...
    $token = $user->createToken('My Token', ['place-orders'])->accessToken;

#### JSON API

Passport 也為管理 Access Token 引入一組 JSON API。你可以 JSON API 與自己的前端配對，來為你的使用者提供一個管理個人的 Access Token 的儀表板。接下來，我們會回顧所有用於管理 Access Token 的 API 端。為了方便講解，我們會使用 [Axios](https://github.com/mzabriskie/axios) 示範如何該端點發送 HTTP 請求。

> {tip} 如果你不想要實作個人的 Access Token 前端介面，你可以使用[前端快速入門](#frontend-quickstart)中的內容在幾分鐘內擁有完整功能的前端介面。

#### `GET /oauth/scopes`

這個路由回傳所有應用程式的 [scopes](#token-scopes) 定義。你可以使用這個路由來列出使用者可以分配到個人的 Access Token 的範圍：

    axios.get('/oauth/scopes')
        .then(response => {
            console.log(response.data);
        });

#### `GET /oauth/personal-access-tokens`

這個路由回傳已認證使用者所建立的所有個人的 Access Token。這個主要助於列出所有使用者的 Token 來讓他們可以做編輯或刪除：

    axios.get('/oauth/personal-access-tokens')
        .then(response => {
            console.log(response.data);
        });

#### `POST /oauth/personal-access-tokens`

這個路由用來建立新個人的 Access Token。它需要兩種資料：Token 的 `name` 和應該被分配到的 Token `scopes`：

    const data = {
        name: 'Token Name',
        scopes: []
    };

    axios.post('/oauth/personal-access-tokens', data)
        .then(response => {
            console.log(response.data.accessToken);
        })
        .catch (response => {
            // 列出回應上的錯誤...
        });

#### `DELETE /oauth/personal-access-tokens/{token-id}`

這個路由可以被用於刪除個人的 Access Token：

    axios.delete('/oauth/personal-access-tokens/' + tokenId);

<a name="protecting-routes"></a>
## 保護路由

<a name="via-middleware"></a>
### 透過中介層

Passport 引入一個[認證 Guard](/laravel_tw/docs/5.5/authentication#adding-custom-guards)，會去驗證傳入的請求上的 Access Token。你一旦使用 `passport` 驅動來設定 `api` Guard，你只需要在任意路由上指定 `auth:api` 中介層，該中介層會需要一組有效的 Access Token：

    Route::get('/user', function () {
        //
    })->middleware('auth:api');

<a name="passing-the-access-token"></a>
### 傳入 Access Token

當呼叫受 Passport 保護的路由時，應用程式的 API 使用者應該在他們的 `Authorization` 請求標頭中指定他們的 Access Token 作為 `Bearer` Token。例如，在使用 Guzzle HTTP 函式庫時:

    $response = $client->request('GET', '/api/user', [
        'headers' => [
            'Accept' => 'application/json',
            'Authorization' => 'Bearer '.$accessToken,
        ],
    ]);

<a name="token-scopes"></a>
## Token Scope


<a name="defining-scopes"></a>
### 定義 Scope

Scope 可以讓你的 API 客戶端在請求授權存取帳號時請求一組特定的權限。例如，如果你正在建構一個電子商務的應用程式，並不是所有 API 使用者都需要下訂單的功能。反而，你可以授權讓使用者只能存取出貨狀態。換句話說，Scope 可以讓應用程式的使用者限制第三方應用程式所執行的操作。

你可以在 `AuthServiceProvider` 的 `boot` 方法中使用 `Passport::tokensCan` 方法來定義你的 API Scope。`tokensCan` 方法接受一組 Scope 名稱陣列和 Scope 說明。Scope 說明可以是任何你希望的內容，並會在同意授權畫面上顯示給使用者：

    use Laravel\Passport\Passport;

    Passport::tokensCan([
        'place-orders' => 'Place orders',
        'check-status' => 'Check order status',
    ]);

<a name="assigning-scopes-to-tokens"></a>
### 將 Scope 分配到 Token

#### 在請求授權碼的時候

在使用授權碼授權請求一組 Access Token 的時候，使用者應該把他們想要的範圍作為 `scope` 查詢字串參數。`scope` 參數應該會是由空格來間隔的方式列出範圍：

    Route::get('/redirect', function () {
        $query = http_build_query([
            'client_id' => 'client-id',
            'redirect_uri' => 'http://example.com/callback',
            'response_type' => 'code',
            'scope' => 'place-orders check-status',
        ]);

        return redirect('http://your-app.com/oauth/authorize?'.$query);
    });

#### 在發放個人的 Access Token 的時候

如果你使用 `User` 模型的 `createToken` 方法來發放個人的 Access Token，你可以將想要的 Scope 陣列作為第二個參數傳入該方法：

    $token = $user->createToken('My Token', ['place-orders'])->accessToken;

<a name="checking-scopes"></a>
### 檢查 Scope

Passport 包含了兩個中介層，可被用於驗證傳入的請求是否授權給定的 Scope Token 來進行認證。請新增下列中介層到 `app/Http/Kernel.php` 檔案的 `$routeMiddleware` 屬性中來開始檢查：

    'scopes' => \Laravel\Passport\Http\Middleware\CheckScopes::class,
    'scope' => \Laravel\Passport\Http\Middleware\CheckForAnyScope::class,

#### 檢查所有範圍

`scopes` 中介層可以被分配到路由，並用來驗證傳入的請求的 Access Token 是否具有*所有*列出的 Scope：

    Route::get('/orders', function () {
        // Access Token 具有「檢查狀態」和「下訂單」的範圍...
    })->middleware('scopes:check-status,place-orders');

#### 檢查任何範圍

`scope` 中介層可以被分配到路由，並用來驗證傳入的請求的 Access Token 是否*至少有一個*列出的 Scope：

    Route::get('/orders', function () {
        // Access Token 具有「檢查狀態」或「下訂單」的範圍...
    })->middleware('scope:check-status,place-orders');

#### 檢查在 Token 實例上的範圍

一旦 Access Token 授權請求進入你的應用程式，你仍然可以使用經過驗證的 `User` 實例上的 `tokenCan` 方法來檢查 token 是否具有給定的範圍：

    use Illuminate\Http\Request;

    Route::get('/orders', function (Request $request) {
        if ($request->user()->tokenCan('place-orders')) {
            //
        }
    });

<a name="consuming-your-api-with-javascript"></a>
## 搭配 JavaScript 使用你的 API

當在建構一個 API 的時候，從自己的 JavaScript 應用程式中使用自己的 API 是非常助於開發的。這種 API 開發方法可以讓你的應用程式與世界共用同一個 API。你的網頁和行動版應用程式、第三方應用程式和任何在各種套件管理器上發佈的 SDK 都可以使用同一個 API。

通常來說，如果你想要從 JavaScript 應用程式中使用自己的 API，你會需要手動發送一組 Access Token 到應用程式，並將它與每個請求傳入應用程式。然而，Passport 包含一個中介層，能夠為你處理這件事。你唯一需要做的是新增 `CreateFreshApiToken` 到你的 `web` 中介層群組：

    'web' => [
        // 其他中介層...
        \Laravel\Passport\Http\Middleware\CreateFreshApiToken::class,
    ],

This Passport middleware will attach a `laravel_token` cookie to your outgoing responses. This cookie contains an encrypted JWT that Passport will use to authenticate API requests from your JavaScript application. Now, you may make requests to your application's API without explicitly passing an access token:

    axios.get('/api/user')
        .then(response => {
            console.log(response.data);
        });

當使用這種認證方法時，預設的 Laravel 所架設的 JavaScript 會指示 Axios 總是發出 `X-CSRF-TOKEN` 和 `X-Requested-With` header。然而，你應該確認是否有把 CRSF Token 引入 [HTML Meta Tag](/laravel_tw/docs/5.5/csrf#csrf-x-csrf-token)：

    window.axios.defaults.headers.common = {
        'X-Requested-With': 'XMLHttpRequest',
    };

> {note} 如果你使用不同的 JavaScript 框架，你應該保證每次送出得請求已被設定有發送 `X-CSRF-TOKEN` 和 `X-Requested-With` header。

<a name="events"></a>
## 事件

Passport 會在發放 Access Toekn 和更新 Token 的時候觸發事件。你可以在資料庫中使用這些事件來修改或刪除其他的 Access Token。只要在你應用程式的 `EventServiceProvider` 中為一些事件附加監聽器：

```php
/**
 * 應用程式的事件監聽器映射。
 *
 * @var array
 */
protected $listen = [
    'Laravel\Passport\Events\AccessTokenCreated' => [
        'App\Listeners\RevokeOldTokens',
    ],

    'Laravel\Passport\Events\RefreshTokenCreated' => [
        'App\Listeners\PruneOldTokens',
    ],
];
```

<a name="testing"></a>
## 測試

Passport 的 `actingAs` 方法可被用於指定當前已認證使用者與它的範圍。`actingAs` 方法的第一個參數是使用者實例，第二個參數是一組會被授權給使用者 Token 的範圍陣列：

    public function testServerCreation()
    {
        Passport::actingAs(
            factory(User::class)->create(),
            ['create-servers']
        );

        $response = $this->post('/api/create-server');

        $response->assertStatus(200);
    }
