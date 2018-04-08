---
layout: post
title: releases
tag: 5.2
---
# 發行說明

- [支援政策](#support-policy)
- [Laravel 5.2](#laravel-5.2)
- [Laravel 5.1.11](#laravel-5.1.11)
- [Laravel 5.1.4](#laravel-5.1.4)
- [Laravel 5.1](#laravel-5.1)
- [Laravel 5.0](#laravel-5.0)
- [Laravel 4.2](#laravel-4.2)
- [Laravel 4.1](#laravel-4.1)

<a name="support-policy"></a>
## 支援政策

對於像是 Laravel 5.1 的 LTS 版本，會提供兩年的臭蟲修復及三年的安全性修復。這些版本為支援及維護提供了長時間的窗口。

對於一般的版本，會提供六個月的臭蟲修復及一年的安全性修復。

<a name="laravel-5.2"></a>
## Laravel 5.2

Laravel 5.2 繼續了 Laravel 5.1 的改進，透過增加多重認證驅動的支援、隱式模型綁定、簡化的 Eloquent 全域範圍、可選用的認證鷹架、中介層群組、頻率限制中介層、陣列驗證的改進等等。

### 認證驅動與「多種認證」

在之前版本的 Laravel 中，只有基於 session 的預設認證驅動可立即使用，且你無法在單一應用程式中擁有一個以上的可認證模型實例。

不過，在 Laravel 5.2，你可以定義額外的認證驅動，同時也可以定義多個可驗證的模型或使用者資料表，並分別控制他們的認證過程。例如，如果你的應用程式有一張「管理員」的使用者資料表及一張「學生」的使用者資料表，你現在可以使用 `Auth` 方法來針對上述的資料表各別進行驗證。

### 認證鷹架

Laravel 已經讓後端處理認證變得相當容易；不過，Laravel 5.2 提供了一個方便且非常快速的方式來為你的前端建構驗證的視圖。只要輕鬆的在你的終端機執行 `make:auth`：

    php artisan make:auth

這個指令會產生簡單並相容 Bootstrap 的視圖，包含使用者登入、註冊、及重置密碼。此指令會用對應的路由更新你的路由檔案。

> **注意：**這個功能應該只用於新的應用程式，而不能用於升級中的應用程式。

### 隱式模型綁定

隱式模型綁定讓直接注入對應模型至你的路由及控制器變得相當容易。例如，假設你定義了一個如下的路由：

    use App\User;

    Route::get('/user/{user}', function (User $user) {
        return $user;
    });

在 Laravel 5.1 時，你通常需要使用 `Route::model` 方法來告知 Laravel 要注入與你定義的路由中 `{user}` 參數相符的 `App\User` 實例。但是，在 Laravel 5.2 中，框架會基於 URI 的片段**自動**注入這個模型，讓你快速能夠存取到你所需的模型實例。

當路由參數片段（`{user}`）符合路由閉包或控制器方法所對應變數名稱（`$user`），且該變數使用了 Eloquent 模型類別作為型別提示時，Laravel 將會自動的注入模型。

### 中介層群組

路由群組讓你將多個路由中介層組織到單一、方便的鍵下面，讓你一次將多個中介層指派給路由。例如，當在同一個應用程式中建構 web UI 及 API 時相當有用。你可以將 session 及 CSRF 路由組合成一個 `web` 群組，而 `api` 群組則可能包含頻率限制。

事實上，預設的 Laravel 5.2 應用程式結構正是採用此種方式。例如，在預設的 `App\Http\Kernel.php` 檔案中你會看到如下的內容：

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
        ],
    ];

接著，`web` 群組可以被指派給路由，像這樣：

    Route::group(['middleware' => ['web']], function () {
        //
    });

### 頻率限制

現在框架中包含了新的頻率限制中介層，讓你簡單的限制給定的 IP 位置，對某個路由在一分鐘內最多的請求次數。例如，限制一個路由針對單一 IP 位置每分鐘只能有 60 次的請求，你可以像這樣：

    Route::get('/api/users', ['middleware' => 'throttle:60,1', function () {
        //
    }]);

### 陣列驗證

在 Laravel 5.2 中驗證表單輸入欄位的陣列變得更簡單。例如，要驗證給定陣列輸入欄位的每個 e-mail 是唯一的，你可以像這樣：

    $validator = Validator::make($request->all(), [
        'person.*.email' => 'email|unique:users'
    ]);

同樣的，你可以在語言檔中指定驗證訊息時使用 `*` 符號，讓基於陣列欄位使用單一驗證訊息成為輕而易舉的事：

    'custom' => [
        'person.*.email' => [
            'unique' => 'Each person must have a unique e-mail address',
        ]
    ],

### Eloquent 全域範圍改進

在前次版本的 Laravel，實作全域 Eloquent 範圍是複雜且容易出錯的；不過，在 Laravel 5.2，全域查詢範圍只需要實作一個簡單的方法：`apply`。

更多關於撰寫全域範圍的資訊，請查閱完整的 [Eloquent 文件](/laravel_tw/docs/5.2/eloquent#global-scopes)。

<a name="laravel-5.1.11"></a>
## Laravel 5.1.11

Laravel 5.1.11 推出了內建的[授權](/laravel_tw/docs/5.2/authorization)支援！透過回呼或原則類別方便的組織你應用程式的授權邏輯，使用簡單明瞭的方法對行為進行授權。

更多的資訊請參考[授權的文件](/laravel_tw/docs/5.2/authorization)。

<a name="laravel-5.1.4"></a>
## Laravel 5.1.4

Laravel 5.1.4 為框架推出了簡單的登入限制。查閱[認證的文件](/laravel_tw/docs/5.2/authentication#authentication-throttling)以取得更多資訊。

<a name="laravel-5.1"></a>
## Laravel 5.1

Laravel 5.1 繼續以 Laravel 5.0 改進而成，透過採用 PSR-2 及新增事件廣播、中介層參數、Artisan 的改進及其他等等。

### PHP 5.5.9+

由於 PHP 5.4 將在九月「結束壽命」，不再接收來自 PHP 開發團隊的安全性更新，Laravel 需要 PHP 5.5.9 或更高的版本。PHP 5.5.9 可以相容最新版本的 PHP 函式庫，像是 Guzzle 及 AWS SDK。

### LTS

Laravel 5.1 是 Laravel 獲得**長期支援**的第一個版本。Laravel 5.1 會獲得兩年的臭蟲修復及三年的安全性修復。此支援窗口是 Laravel 有史以來最大的提供，為較大型的企業客戶及消費者提供了穩定性及安心。
### PSR-2

[PSR-2 程式碼風格指南](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-2-coding-style-guide.md)已經被 Laravel 框架採用為預設的風格指南。此外，所以的產生器都已進行更新，以產生 PSR-2 相容的語法。

### 文件

Laravel 文件的每一頁已被精心審查，並得到顯著的改善。所有的程式碼範例也進行了審查，並擴增以提供更多的關聯性與情境。

### 事件廣播

在許多現代的 web 應用程式，web sockets 都用在實現即時，即時更新使用者介面。當在伺服器上更新一些資料，websocket 連線通常傳送一個訊息透過客戶端處理。

為了協助你建立這些類型的應用程式，Laravel 讓你可以簡單的經由 websocket 連線來「廣播」你的事件。廣播你的 Laravel 事件讓你能夠在你的伺服器端程式碼和你的客戶端 JavaScript 框架間分享相同的事件名稱。

欲瞭解更多關於事件廣播，請查閱[事件的文件](/laravel_tw/docs/5.2/events#broadcasting-events)。

### 中介層參數

中介層也可以額外接受自訂參數，例如，如果應用程式要在執行特定操作之前，檢查通過驗證的使用者是否具備該操作的「角色」，可以建立 `RoleMiddleware` 來接收角色名稱作為額外的參數。

    <?php

    namespace App\Http\Middleware;

    use Closure;

    class RoleMiddleware
    {
        /**
         * 執行請求過濾。
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

    Route::put('post/{id}', ['middleware' => 'role:editor', function ($id) {
        //
    }]);

關於中介層的更多訊息，請查閱[中介層的文件](/laravel_tw/docs/5.2/middleware)。

### 測試翻修

Laravel 內建的測試功能已得到顯著的改善。各種新的方法提供了流暢、簡明的介面與應用程式進行互動，並檢查回應。例如，查看下方的測試：

    public function testNewUserRegistration()
    {
        $this->visit('/register')
             ->type('Taylor', 'name')
             ->check('terms')
             ->press('Register')
             ->seePageIs('/dashboard');
    }

關於測試的更多訊息，請查閱[測試的文件](/laravel_tw/docs/5.2/testing)。

### 模型工廠

Laravel 現在提供一個簡單的方式建立模擬的 Eloquent 模型，使用[模型工廠](/laravel_tw/docs/5.2/testing#model-factories)。模型工廠讓你簡單的為 Eloquent 模型定義一組「預設」的屬性，並為你的測試或資料填充產生測試模型實例。模型工廠也使用進階的 [Faker](https://github.com/fzaninotto/Faker) PHP 函式庫來產生隨機屬性的資料：

    $factory->define(App\User::class, function ($faker) {
        return [
            'name' => $faker->name,
            'email' => $faker->email,
            'password' => str_random(10),
            'remember_token' => str_random(10),
        ];
    });

更多關於模型工廠的訊息，請查閱[它的文件](/laravel_tw/docs/5.2/testing#model-factories)。

### Artisan 的改進

Artisan 指令現在可以使用簡單的方式定義，相似於路由的「署名」，提供了一個非常簡單的介面來定義指令列的參數及選項。舉個例子，你可以定義一個簡單的指令及它的選項，如下：

    /**
     * 指令列的名字及署名。
     *
     * @var string
     */
    protected $signature = 'email:send {user} {--force}';

更多關於定義 Artisan 指令的訊息，請參考 [Artisan 的文件](/laravel_tw/docs/5.2/artisan)。

### 資料夾結構

為了更好地表明目的，`app/Commands` 目錄已經被更名為 `app/Jobs`。此外，`app/Handler` 目錄已經被合併成一個只包含事件監聽器的 `app/Listeners` 目錄。但是，這不是一個重大的改變，你不必更新成新的資料夾結構也能使用 Laravel 5.1。

### 加密

在 Laravel 之前的版本，加密是透過 `mcrypt` PHP 擴充功能進行處理。不過，從 Laravel 5.1 起，加密透過更積極維護的 `openssl` 擴充功能進行處理。

<a name="laravel-5.0"></a>
## Laravel 5.0

Laravel 5.0 在預設的專案上引進了新的應用程式架構。新的架構提供了更好的基礎在 Laravel 中建立強健的應用程式，以及在應用程式中全面採用新的自動載入標準（PSR-4）。首先，來檢視一些主要更動：

### 新的目錄結構

舊的 `app/models` 目錄已經完全被移除。相對的，你所有的程式碼都放在 `app` 目錄下，以及預設上使用 `App` 命名空間。這個預設的命名空間可以快速的使用新的 `app:name` Artisan 命令來做更改。

控制器、中介層，以及請求（Laravel 5.0 中新型態的類別），現在分門別類的放在 `app/Http` 目錄下，因為他們都與應用程式的 HTTP 傳輸層相關。除了一個路由設定的檔案外，所有的中介層現在都分開成為獨自的類別檔。

新的 `app/Providers` 目錄取代了舊版 Laravel 4.x `app/start` 裡的檔案。這些服務提供者為應用程式提供了各種的引導功能，像是錯誤處理，日誌紀錄，路由載入等等。當然，你可以任意的建立新的服務提供者到應用程式中。

應用程式的語系檔案和視圖都被移到 `resources` 目錄下。

### Contracts

所有 Laravel 主要元件實作所用的介面都放在 `illuminate/contracts` 儲存庫中。這個儲存庫沒有其他的外部相依。這些方便、集成的介面，可以讓你用來讓依賴注入變得低耦合，將可以簡單作為 Laravel Facades 的替代選項。

更多關於 contracts 的資訊，參考[完整文件](/laravel_tw/docs/5.2/contracts)。

### 路由快取

如果你的應用程式全部都是使用控制器路由，你可以使用新的 `route:cache` Artisan 命令來大幅度地加快註冊路由。這對於有 100 個以上路由規則的應用程式很有用，可以**大幅度地**加快應用程式這部分的處理速度。

### 路由中介層

除了像 Laravel 4 風格的路由「過濾器」，Laravel 5 現在有 HTTP 中介層，而原本的認證和 CSRF 「過濾器」已經改寫成中介層。中介層提供了單一、一致的介面取代了各種過濾器，讓你在請求進到應用程式前，可以簡單地檢查甚至拒絕請求。

更多關於中介層的資訊，參考[完整文件](/laravel_tw/docs/5.2/middleware)。

### 控制器方法注入

除了之前有的建構函數注入外，你現在可以在控制器方法使用型別提示進行依賴注入。[服務容器](/laravel_tw/docs/5.2/container)會自動注入依賴，即使路由包含了其他參數也不成問題：

    public function createPost(Request $request, PostRepository $posts)
    {
        //
    }

### 認證基本架構

使用者註冊，認證，以及重設密碼的控制器現在已經預設含括了，包含相對應的視圖，放在 `resources/views/auth`。除此之外，「users」資料表遷移也已經預設存在框架中了。這些簡單的資源，可以讓你快速開發應用程式的點子，而不用陷在撰寫認證模板的泥淖上。認證相關的視圖可以經由 `auth/login` 以及 `auth/register` 路由訪問。`App\Services\Auth\Registrar` 服務會負責處理使用者認證和新增的相關邏輯。

### 事件物件

你現在可以將事件定義成物件，而不是僅使用字串。例如，瞧瞧以下的事件：
    <?php

    class PodcastWasPurchased
    {
        public $podcast;

        public function __construct(Podcast $podcast)
        {
            $this->podcast = $podcast;
        }
    }

這個事件可以像一般使用那樣被派發：

    Event::fire(new PodcastWasPurchased($podcast));

當然，你的事件處理會收到事件的物件而不是資料的列表：

    <?php

    class ReportPodcastPurchase
    {
        public function handle(PodcastWasPurchased $event)
        {
            //
        }
    }

更多關於使用事件的資訊，參考[完整文件](/laravel_tw/docs/5.2/events)。

### 指令及隊列

除了 Laravel 4 形式的隊列任務，Laravel 5 以簡單的命令物件作為隊列任務。這些命令放在 `app/Commands` 目錄下。下面是個範例的命令：

    <?php

    class PurchasePodcast extends Command implements SelfHandling, ShouldBeQueued
    {

        use SerializesModels;

        protected $user, $podcast;

        /**
         * Create a new command instance.
         *
         * @return void
         */
        public function __construct(User $user, Podcast $podcast)
        {
            $this->user = $user;
            $this->podcast = $podcast;
        }

        /**
         * Execute the command.
         *
         * @return void
         */
        public function handle()
        {
            // Handle the logic to purchase the podcast...

            event(new PodcastWasPurchased($this->user, $this->podcast));
        }
    }

Laravel 的基底控制器使用了新的 `DispatchesCommands` trait，讓你可以簡單的派發命令執行。

    $this->dispatch(new PurchasePodcastCommand($user, $podcast));

當然，你也可以將命令視為同步執行（而不會被放到隊列裡）的任務。事實上，使用命令是個好方式，讓你可以封裝應用程式需要執行的複雜任務。更多相關的資訊，參考 [command bus](/laravel_tw/docs/5.2/bus) 文件。

### 資料庫隊列

`database` 隊列驅動現在已經包含在 Laravel 中了，提供了簡單的本地端隊列驅動，讓你除了資料庫相關軟體外不需安裝其他套件。

### Laravel 排程器

過去，開發者可以產生 Cron 設定，用以排程所有他們想要執行的命令列指令。然而，這是件很頭痛的事情，因為你的命令列排程不在版本控制中，而你必須登入到伺服器裏加入 Cron 設定。讓生活變得簡單點。Laravel 命令列排程，讓你可以流暢而且具有表達性的定義在 Laravel 裡面，定義你的命令排程，而且只需要在伺服器裡設定一個 Cron 設定。

它會看起來如下：

    $schedule->command('artisan:command')->dailyAt('15:00');

當然，快參考[完整文件](/laravel_tw/docs/5.2/artisan#scheduling-artisan-commands)學習所有排程相關知識。

### Tinker 與 Psysh

`php artisan tinker` 命令現在使用 Justin Hileman 的 [Psysh](https://github.com/bobthecow/psysh)，一個 PHP 更強大的 REPL。如果你喜歡 Laravel 4 的 Boris，你也會喜歡上 Psysh。更好的是，它可以在 Windows 上運行！要開始使用，只要輸入：

    php artisan tinker

### DotEnv

比起一堆令人困惑的、巢狀的環境設定檔目錄，Laravel 5 現在使用了 Vance Lucas 的 [DotEnv](https://github.com/vlucas/phpdotenv)。這個套件提供了超級簡單的方式管理設定檔，並且讓 Laravel 5 環境偵測變得輕鬆。更多的細節，參考完整的[設定檔文件](/laravel_tw/docs/5.2/configuration#environment-configuration)。

### Laravel Elixir

Jeffrey Way 的 Laravel Elixir 提供了一個流暢、口語化的介面，可以編譯以及合併 assets。如果你曾經因為學習 Grunt 或 Gulp 而被嚇到，不必再害怕了。Elixir 讓使用 Gulp 編譯 Less、Sass 及 CoffeeScript 變得簡單。它甚至可以幫你執行測試！

更多關於 Elixir 的資訊，參考[完整文件](/laravel_tw/docs/5.2/elixir)。

### Laravel Socialite

Laravel Socialite 是個選用的，Laravel 5.0 以上相容的套件，提供了無痛的 OAuth 認證。目前 Socialite 支援 Facebook、Twitter、Google 以及 GitHub。它寫起來可能像這樣：

    public function redirectForAuth()
    {
        return Socialize::with('twitter')->redirect();
    }

    public function getUserFromProvider()
    {
        $user = Socialize::with('twitter')->user();
    }

不用再花上數小時撰寫 OAuth 的認證流程。只要幾分鐘！查看[完整文件](/laravel_tw/docs/5.2/authentication#social-authentication) 裡有所有的細節。

### 檔案系統整合

Laravel 現在包含了強大的[檔案系統](https://github.com/thephpleague/flysystem)（一個檔案系統的抽象函式庫），提供了無痛的整合，把本地端檔案系統、Amazon S3 和 Rackspace 雲端儲存整合在一起，
有統一且優雅的 API！現在要將檔案存到 Amazon S3 相當簡單：

    Storage::put('file.txt', 'contents');

更多關於 Laravel 檔案系統整合，參考[完整文件](/laravel_tw/docs/5.2/filesystem)。

### Form Requests

Laravel 5.0 引進了 **form requests**，是繼承自 `Illuminate\Foundation\Http\FormRequest` 的類別。這些 request 物件可以和控制器方法依賴注入結合，提供一個不需樣板的方法，來驗證使用者輸入。讓我們深入點，看一個 `FormRequest` 的範例：

    <?php

    namespace App\Http\Requests;

    class RegisterRequest extends FormRequest
    {
        public function rules()
        {
            return [
                'email' => 'required|email|unique:users',
                'password' => 'required|confirmed|min:8',
            ];
        }

        public function authorize()
        {
            return true;
        }
    }

定義好類別後，我們可以在控制器動作裡使用型別提示：

    public function register(RegisterRequest $request)
    {
        var_dump($request->input());
    }

當 Laravel 的服務容器辨別出要注入的類別是個 `FormRequest` 實例，該請求將會被**自動驗證**。意味著，當你的控制器動作被呼叫了，你可以安全的假設 HTTP 的請求輸入己經被驗證過，根據你在 form request 類別裡自定的規則。甚至，若這個請求驗證不通過，一個 HTTP 重導（可以自定），會自動發出，錯誤訊息可以被閃存到 session 中或是轉換成 JSON 返回。**表單驗證再簡單不過如此。**更多關於 `FormRequest` 驗證，參考[文件](/laravel_tw/docs/5.2/validation#form-request-validation)。

### 簡易控制器請求驗證

Laravel 5 基底控制器包含一個 `ValidatesRequests` trait。這個 trait 包含了一個簡單的 `validate` 方法可以驗證請求。如果對你的應用程式來說 `FormRequests` 太複雜了，參考這個：

    public function createPost(Request $request)
    {
        $this->validate($request, [
            'title' => 'required|max:255',
            'body' => 'required',
        ]);
    }

如果驗證失敗，會拋出例外以及回傳適當的 HTTP 回應到瀏覽器。驗證錯誤資訊會被閃存到 session！而如果請求是 AJAX 請求，Laravel 會自動回傳 JSON 格式的驗證錯誤資訊。

更多關於這個新方法的資訊，參考[這個文件](/laravel_tw/docs/5.2/validation#controller-validation)。

### 新的產生器

因應新的應用程式預設架構，框架新增了 Artisan generator 命令。使用 `php artisan list` 瞧瞧更多細節。

### 設定檔快取

你現在可以透過 `config:cache` 命令將所有的設定檔緩存在單一檔案中了。

### Symfony VarDumper

出名的 `dd` 輔助函式，其可以在除錯時印出變數資訊，已經升級成使用令人驚豔的 Symfony VarDumper 套件。它提供了顏色標記的輸出，甚至陣列可以自動縮合。在專案中試試下列程式碼：

    dd([1, 2, 3]);

<a name="laravel-4.2"></a>
## Laravel 4.2

此發行版本的完整更動列表可以從一個 4.2 的完整安裝下，執行 `php artisan changes` 命令，或者 [Github 上的更動紀錄](https://github.com/laravel/framework/blob/4.2/src/Illuminate/Foundation/changes.json)。此紀錄僅含括主要的強化更新和此發行的更動部分。

> **附註:** 在 4.2 發佈週期間，許多小的臭蟲修正與功能強化被整併至各個 4.1 的子發行版本中。所以最好確認 Laravel 4.1 版本的更新列表。

### PHP 5.4 需求

Laravel 4.2 需要 PHP 5.4 以上的版本。此 PHP 更新版本讓我們可以使用 PHP 的新功能：traits 來為像是 [Laravel 收銀台](/docs/billing) 來提供更具表達力的介面。PHP 5.4 也比 PHP 5.3 帶來顯著的速度及效能提升。

### Laravel Forge

Larvel Forge，一個網頁應用程式，提供一個簡單的介面去建立管理你雲端上的 PHP 伺服器，像是 Linode、DigitalOcean、Rackspace 和 Amazon EC2。支援自動化 nginx 設定、SSH 金鑰管理、Cron job 自動化、透過 NewRelic & Papertrail 伺服器監控、「推送部署」、Laravel queue worker 設定等等。Forge 提供最簡單且更實惠的方式來部署所有你的 Laravel 應用程式。

預設 Laravel 4.2 的安裝裡，`app/config/database.php` 設定檔預設已為 Forge 設定完成，讓在平台上的全新應用程式更方便部署。

關於 Laravel Forge 的更多資訊可以在[官方 Forge 網站](https://forge.laravel.com)上找到。

### Laravel Homestead

Laravel Homestead 是一個為部署健全的 Laravel 和 PHP 應用程式的官方 Vagrant 環境。絕大多數的封裝包的相依與軟體在發佈前已經部署處理完成，讓封裝包可以極快的被啟用。Homestead 包含 Nginx 1.6、PHP 5.5.12、MySQL、Postres、Redis、Memcached、Beanstalk、Node、Gulp、Grunt 和 Bower。Homestead 包含一個簡單的 `Homestead.yaml` 設定檔，讓你在單一個封裝包中管理多個 Laravel 應用程式。

預設的 Laravel 4.2 安裝中包含的 `app/config/local/database.php` 設定檔使用 Homestead 的資料庫作為預設。讓 Laravel 初始化安裝與設定更為方便。

官方文件已經更新並包含在 [Homestead 文件](/docs/homestead) 中。

### Laravel 收銀台

Laravel 收銀台是一個簡單、具表達性的資源庫，用來管理 Stripe 的訂閱帳務。雖然在安裝中此元件依然是選用，我們依然將收銀台文件包含在主要 Laravel 文件中。此收銀台發布版本帶來了數個錯誤修正、多貨幣支援還有支援最新的 Stripe API。

### Queue Workers 常駐程式

Artisan `queue:work` 命令現在支援 `--daemon` 參數讓 worker 可以以「常駐程式」啟用。代表 worker 可以持續的處理隊列工作不需要重啓框架。這讓一個複雜的應用程式部署過程中，使得 CPU 的使用有顯著的降低。

更多關於 Queue Workers 常駐程式資訊請詳閱 [queue 文件](/docs/queues#daemon-queue-worker)。

### Mail API Drivers

Laravel 4.2 為 `Mail` 函式採用了新的 Mailgun 和 Mandrill API 驅動。對許多應用程式而言，他提供了比 SMTP 更快也更可靠的方法來遞送郵件。新的驅動使用了 Guzzle 4 HTTP 資源庫。

### 軟刪除 Traits

對於軟刪除和全作用域更簡潔的方案，PHP 5.4 的 `traits` 提供了一個更加簡潔的軟刪除架構和全局作用域，這些新架構為框架提供了更有擴展性的功能，並且讓框架更加簡潔。

更多關於軟刪除的文檔請見: [Eloquent documentation](/docs/eloquent#soft-deleting).

### 更為方便的 認證(auth) & Remindable Traits

得益於 PHP 5.4 traits，我們有了一個更簡潔的用戶認證和密碼提醒接口，這也讓 `User` 模型文件更加精簡。

### "簡易分頁"

一個新的 `simplePaginate` 方法已被加入到查詢以及 Eloquent 查詢器中。讓你在分頁視圖中，使用簡單的「上一頁」和「下一頁」連結查詢更為高效。

### 遷移確認

在正式環境中，破壞性的遷移動作將會被再次確認。如果希望取消提示字元確認請使用 `--force` 參數。

<a name="laravel-4.1"></a>
## Laravel 4.1

### 完整更動列表

此發行版本的完整更動列表，可以在版本 4.1 的安裝中命令列執行 `php artisan changes` 取得，或者瀏覽 [Github 更動檔](https://github.com/laravel/framework/blob/4.1/src/Illuminate/Foundation/changes.json) 中了解。其中只記錄了該版本比較主要的強化功能和更動。

### 新的 SSH 元件

一個全新的 `SSH` 元件在此發行版本中登場。此功能讓你可以輕易的 SSH 至遠端伺服器並執行命令。更多資訊，可以參閱 [SSH 元件文件](/docs/ssh)。

新的 `php artisan tail` 指令就是使用這個新的 SSH 元件。更多的資訊，請參閱 `tail` [指令集文件](http://laravel.com/docs/ssh#tailing-remote-logs)。

### Boris In Tinker

如果您的系統支援 [Boris REPL](https://github.com/d11wtq/boris)，`php artisan thinker` 指令將會使用到它。系統中也必須先行安裝好 `readline` 和 `pcntl` 兩個 PHP 套件。如果你沒這些套件，從 4.0 之後將會使用到它。

### Eloquent 強化

Eloquent 新增了新的 `hasManyThrough` 關係鏈。想要了解更多，請參見 [Eloquent 文件](/docs/eloquent#has-many-through)。

一個新的 `whereHas` 方法也同時登場，他將允許[檢索基於關係模型的約束](/docs/eloquent#querying-relations)。

### 資料庫讀寫分離

Query Builder 和 Eloquent 目前透過資料庫層，已經可以自動做到讀寫分離。更多的資訊，請參考 [文件](/docs/database#read-write-connections)。

### 隊列排序

隊列排序已經被支援，只要在 `queue:listen` 命令後將隊列以逗號分隔送出。

### 失敗隊列作業處理

現在隊列將會自動處理失敗的作業，只要在 `queue:listen` 後加上 `--tries` 即可。更多的失敗作業處理可以參見 [隊列文件](/docs/queues#failed-jobs)。

### 緩存標籤

緩存「區塊」已經被「標籤」取代。緩存標籤允許你將多個「標籤」指向同一個緩存物件，而且可以清空所有被指定某個標籤的所有物件。更多使用緩存標籤資訊請見 [緩存文件](/docs/cache#cache-tags)。

### 更具彈性的密碼提醒

密碼提醒引擎已經可以提供更強大的開發彈性，如：認證密碼、顯示狀態訊息等等。使用強化的密碼提醒引擎，更多的資訊 [請參閱文件](/docs/security#password-reminders-and-reset)。

### 強化路由引擎

Laravel 4.1 擁有一個完全重新編寫的路由層。API 一樣不變。然而與 4.0 相比，速度快上 100%。整個引擎大幅的簡化，且對於路由表達式的編譯大大減少對 Symfony Routing 的依賴。

### 強化 Session 引擎

此發行版本中，我們亦發佈了全新的 Session 引擎。如同路由增進的部分，新的 Session 曾更加簡化且更快速。我們不再使用 Symfony 的 Session 處理工具，並且使用更簡單、更容易維護的客製化解法。

### Doctrine DBAL

如果你有在你的遷移中使用到 `renameColumn`，之後你必須在 `composer.json` 裡加 `doctrine/dbal` 進相依套件中。此套件不再預設包含在 Laravel 之中。
