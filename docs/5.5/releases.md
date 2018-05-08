---
layout: post
title: releases
tag: 5.5
---
# 發行說明

- [版本規劃](#versioning-scheme)
- [提供支援的原則](#support-policy)
- [Laravel 5.5](#laravel-5.5)

<a name="versioning-scheme"></a>
## 版本規劃

Laravel 的版本規劃維持以下慣例：`主流版本.主要版本.次要版本`。會在每六個月（二月和八月）發佈新的主要框架版本，而次要版本通常可能會每週發布一次。次要版本的更新內容**不會**破壞主要版本的結構。

當你從應用程式或套件中引用 Laravel 框架或它的元件，你應該維持版本的控制，像是 `5.5.*`，由於 Laravel 的主要版本會有重大改變，我們盡力為你能夠在一天或更短的時間內升級新的主要版本號。

主流版本的轉換通常會相隔好幾年，這也表示框架的結構與慣例已經準備重新開始。現階段還尚未規劃開發新的主流版本。

#### 為什麼 Laravel 不使用語意化版控？

一方面，Laravel 的所有可選元件（Cashier、Dusk、Valet、Socialite，等等）**都有**使用語意化版控。然而，Laravel 框架並沒有這麼做。這是因為語意化版控式是為了確認新舊版本的程式碼的相容性之還原方式。就算使用了語意化版控，你仍必須安裝升級套件，並執行自動化測試來確認是否有與**實際**程式碼不相容的地方。

反正 Laravel 就是使用了這個版控規劃，這個版控規劃可以更直觀地表示版本的實際範圍。此外，由於次要版本的更新內容**不會**破壞主要版本的結構，只要你的版本維持在`主流版本.主要版本.*`的慣例，你就不會接收到破壞主體的更新。

<a name="support-policy"></a>
## 提供支援的原則

對於 LTS 版本，像是 Laravel 5.5，會提供兩年的臭蟲修復與五年的安全性修復。這些版本會提供長期的支援與維護。至於一般的版本，只會提供六個月的臭蟲修復和一年的安全性修復。

<a name="laravel-5.5"></a>
## Laravel 5.5 (LTS)

Laravel 5.5 持續對 Laravel 5.4 進行改良：透過新增自動檢查套件、API 資源與轉換、自動註冊終端指令、隊列任務鏈結、隊列任務的速率限制、根據時間來規範重試任務、渲染 mailable、渲染與報告例外、更一致的例外處理、改良資料庫測試、更簡單的自訂驗證規則、React 前端預設、`Route::view` 和 `Route::redirect` 方法、Memcached 和 Redis 快取驅動的「鎖定」、被動通知、在 Dusk 的 headless Chrome 支援、更便捷的 Blade 模板、改良可信任的代理的支援，以及更多。

此外，Laravel 5.5 剛好也發佈了 [Laravel Horizon](https://horizon.laravel.com)，這是一個用於 Redis 為基礎的 Laravel 隊列的全新華麗隊列儀表板和設定系統。

> {tip} 這個文件總結了框架最有感的改良內容。然而，更徹底的更新日誌都可以在 [GitHub](https://github.com/laravel/framework/blob/5.5/CHANGELOG-5.5.md) 上找到。

### Laravel Horizon

Horizon 為你的 Laravel Redis 隊列提供了一個華麗的儀表板以及程式碼驅動的設定。Horizon 可以讓你輕易的監控隊列系統的關鍵指標，像是任務的流量，執行時間和失敗任務。

你的所有執行設定會被儲存在單一且簡單的設定檔中，可以讓你的設定共用在整個團隊的版本控制系統中。

想知道更多 Horizon 的資訊，請查閱[完整的 Horizon 文件](/laravel_tw/docs/5.5/horizon)

### 套件的發掘

> {video} Laracasts 上有提供這個功能的免費[教學影片的網站](https://laracasts.com/series/whats-new-in-laravel-5-5/episodes/5)。

在之前的 Laravel 版本中，安裝套件通常會要求幾個額外的步驟，像是新增服務提供者到 `app` 設定檔，並註冊任何相關的 Facade。然而，從 Laravel 5.5 開始，Laravel 可以自動為你檢測並註冊服務提供者和 Facades。

例如，你能透過安裝 `barryvdh/laravel-debugbar` 套件到 Laravel 應用程式中來體驗這個功能，一旦透過 Composer 安裝套件，該除錯工具會被直接用於應用程式，且不需額外作設定：

    composer require barryvdh/laravel-debugbar

套件開發者只需要新增他們的服務提供者和 Facades 到套件的 `composer.json` 檔案：

    "extra": {
        "laravel": {
            "providers": [
                "Laravel\\Tinker\\TinkerServiceProvider"
            ]
        }
    },

關於升級套件來使用服務提供者和 Facade 發掘的更多相關資訊，請查閱[套件開發](/laravel_tw/docs/5.5/packages)的完整文件。

### API 資源

在建構一個 API 時，你可能需要一個位於 Eloquent 模型和實際回傳應用程式使用者的 JSON 回應之間的轉換層。Laravel 的 Resource 類別可以讓你更直觀且容易的將模型和模型集合轉換成 JSON。

Resource 類別代表著需要被轉換成 JSON 結構的一個模型。例如，這是一個簡單的 `User` Resource 類別：

    <?php

    namespace App\Http\Resources;

    use Illuminate\Http\Resources\Json\Resource;

    class UserResource extends Resource
    {
        /**
         * 將資源轉換成陣列。
         *
         * @param  \Illuminate\Http\Request
         * @return array
         */
        public function toArray($request)
        {
            return [
                'id' => $this->id,
                'name' => $this->name,
                'email' => $this->email,
                'created_at' => $this->created_at,
                'updated_at' => $this->updated_at,
            ];
        }
    }

當然，這只是一個 API 資源的最基本範例。Laravel 也提供各種方法來協助你建構 Resource 和 Resource 的集合。想知道更多的資訊，請查閱[API 資源](/laravel_tw/docs/5.5/eloquent-resources)的完整文件。

### 自動註冊終端指令

> {video} Laracasts 上有提供這個功能的免費[教學影片的網站](https://laracasts.com/series/whats-new-in-laravel-5-5/episodes/12)。

建立新終端指令時，你不再需要在終端 Kernel 的 `$commadns` 屬性中手動列出它們了。反而是從 Kernel 中的 `commands` 方法呼叫新的 `load` 方法，這會去掃描給定目錄中的任何終端指令並自動將它們註冊：

    /**
     * 註冊應用程式的指令。
     *
     * @return void
     */
    protected function commands()
    {
        $this->load(__DIR__.'/Commands');

        // ...
    }

### 新的前端預設框架選項

> {video} Laracasts 上有提供這個功能的免費[教學影片的網站](https://laracasts.com/series/whats-new-in-laravel-5-5/episodes/4)。

雖然基本的 Vue 框架仍然在 Laravel 5.5，但有幾個新的前端預設框架選項可使用。剛建立新的 Laravel 應用程式中，你能使用 `preset` 指令將 Vue 框架換成 React 框架：

    php artisan preset react

或者，你能移除該 JavaScript 和 CSS 框架，並完整使用 `none` 框架預設。該框架預設會提供簡單的純 Sass 和一些簡單的 JavaScript：

    php artisan preset none

> {note} 這些只適用於新安裝的 Laravel，不應該使用在已開發許久的應用程式。

### 隊列任務鏈結

任務鏈結可以讓你指定一個會被依序執行的隊列任務清單。如果有個任務在流程中發生錯誤，則剩下的任務將會停止執行。要執行隊列任務鏈結，你可以在任何指派的任務上使用 `withChain` 方法：

    ProvisionServer::withChain([
        new InstallNginx,
        new InstallPhp
    ])->dispatch();

### 隊列任務速率限制

如果你的應用程式正與 Redis 交換資料，你現在可以透過時間或併發來限制隊列任務。這個功能有助於你的隊列任務正與被限制速率的 API 交換資料時後。例如，你可以限制給定任務類型在六十秒只能執行十次：

    Redis::throttle('key')->allow(10)->every(60)->then(function () {
        // 任務邏輯...
    }, function () {
        // 無法獲得鎖定...

        return $this->release(10);
    });

> {tip} 在以上範例中，`key` 可以是任何字串，專門用來識別你想要限速的任務類型。例如，你可能希望根據任務的類別名稱和 Eloquent 模型的 ID 來建構這個鍵。

或者，你可以指定能同時處理給定任務的最大 Worker 數量。這有助於在隊列任務一次只能修改一個任務資源的時候。例如，我們可以限制給定類型的任務只能一次被一個 worker 處理：

    Redis::funnel('key')->limit(1)->then(function () {
        // 任務邏輯...
    }, function () {
        // 無法獲得鎖定...

        return $this->release(10);
    });

### 根據時間來規範重試任務

如何定義任務超過嘗試次數的替代方法，你現在可以定義任務超時的時間。這可以讓你在給定時間範圍內嘗試執行任務的次數。要定義該任務超時的時間，請新增 `retryUntil` 方法到你的任務類別中：

    /**
     * 確認任務超時的時間。
     *
     * @return \DateTime
     */
    public function retryUntil()
    {
        return now()->addSeconds(5);
    }

> {tip} 你也可以在隊列事件監聽器中定義一個 `retryUntil` 方法。

### 驗證規則物件

> {video} Laracasts 上有提供這個功能的免費[教學影片的網站](https://laracasts.com/series/whats-new-in-laravel-5-5/episodes/7)。

驗證規則物件提供一個既新又嚴謹的方式來新增自訂驗證規則到你的應用程式。在 Laravel 的前一個版本中，`Validator::extend` 方法透過使用閉包來自訂驗證規則。然而，這會使程式碼變的複雜。在 Laravel 5.5 中，新的 Artisan 的 `make:rule` 指令會在 `app\Rules` 目錄中產生一個驗證規則：

    php artisan make:rule ValidName

規則物件只有兩種方法：`passes` 和 `message`。`passes` 方法接受屬性值與名稱，並會根據屬性值的有效性來回傳 `true` 或 `false`。`message` 方法會回傳應該被用在驗證錯誤時所回傳的驗證錯誤訊息：

    <?php

    namespace App\Rules;

    use Illuminate\Contracts\Validation\Rule;

    class ValidName implements Rule
    {
        /**
         * 確認是否有通過驗證規則。
         *
         * @param  string  $attribute
         * @param  mixed  $value
         * @return bool
         */
        public function passes($attribute, $value)
        {
            return strlen($value) === 6;
        }

        /**
         * 取得驗證錯誤訊息。
         *
         * @return string
         */
        public function message()
        {
            return 'The name must be six characters long.';
        }
    }

一旦規則被定義，你就可以傳入一個規則物件實例與其他驗證規則來使用：

    use App\Rules\ValidName;

    $request->validate([
        'name' => ['required', new ValidName],
    ]);

### 可信任的代理整合

當你執行的應用程式背後的負載平衡器的 TLS 和 SSL 憑證過後期，你可能注意到你的應用程式有時不會產生 HTTPS 連結。通常是因為應用程式從 Port 80 上的負載平衡器轉發流量時，並不知道要產生安全的連結。

要解決這件事，許多 Laravel 使用者會去安裝 Chris Fidao 的 [Trusted Proxies](https://github.com/fideloper/TrustedProxy)套件。 由於這個套件很常被使用，所以這個套件現在會內建到預設的 Laravel 5.5 中。

在預設的 Laravel 5.5 應用程式中，已包含一個新的 `App\Http\Middleware\TrustProxies`中介層。這個中介層可以讓你快速自訂應用程式應該信任的代理：

    <?php

    namespace App\Http\Middleware;

    use Illuminate\Http\Request;
    use Fideloper\Proxy\TrustProxies as Middleware;

    class TrustProxies extends Middleware
    {
        /**
         * 這個應用程式可信任的代理。
         *
         * @var array
         */
        protected $proxies;

        /**
         * 當前代理標頭映射。
         *
         * @var array
         */
        protected $headers = [
            Request::HEADER_FORWARDED => 'FORWARDED',
            Request::HEADER_X_FORWARDED_FOR => 'X_FORWARDED_FOR',
            Request::HEADER_X_FORWARDED_HOST => 'X_FORWARDED_HOST',
            Request::HEADER_X_FORWARDED_PORT => 'X_FORWARDED_PORT',
            Request::HEADER_X_FORWARDED_PROTO => 'X_FORWARDED_PROTO',
        ];
    }

### 被動通知

有時你可能需要發送一個通知給尚未在應用程式中儲存的「使用者」。使用新的 `Notification::route` 方法，你可以指定在發送通知之前，指定臨時的通知路由資訊：

    Notification::route('mail', 'taylor@laravel.com')
                ->route('nexmo', '5555555555')
                ->send(new InvoicePaid($invoice));

### 渲染 Mailable

> {video} Laracasts 上有提供這個功能的免費[教學影片的網站](https://laracasts.com/series/whats-new-in-laravel-5-5/episodes/6)。

Mailable 現在能直接從路由中回傳，可以讓你在瀏覽器中快速預覽 Mailable 的設計：

    Route::get('/mailable', function () {
        $invoice = App\Invoice::find(1);

        return new App\Mail\InvoicePaid($invoice);
    });

### 渲染與回報例外

> {video} Laracasts 上有提供這個功能的免費[教學影片的網站](https://laracasts.com/series/whats-new-in-laravel-5-5/episodes/18)。

在上個 Laravel 版本中，對於渲染給定例外的自訂回應，你可以不由得的在例外處理中使用「型別檢查」。例如，你可能在例外處理的 `render` 方法中撰寫像這樣的程式碼：

    /**
     * 將例外渲染到 HTTP 回應。
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \Exception  $exception
     * @return \Illuminate\Http\Response
     */
    public function render($request, Exception $exception)
    {
        if ($exception instanceof SpecialException) {
            return response(...);
        }

        return parent::render($request, $exception);
    }

在 Laravel 5.5 中，你現在可以在例外上直接定義一個 `render` 方法。這可以讓你在例外上直接放置自訂回應的渲染邏輯，這有助於避免堆積條件表達式在例外處理中。如果你也想要自訂例外的回報邏輯，你可以在類別上定義 `report` 方法：

    <?php

    namespace App\Exceptions;

    use Exception;

    class SpecialException extends Exception
    {
        /**
         * 回報例外。
         *
         * @return void
         */
        public function report()
        {
            //
        }

        /**
         * 回報例外。
         *
         * @param  \Illuminate\Http\Request
         * @return void
         */
        public function render($request)
        {
            return response(...);
        }
    }

### 請求驗證

> {video} Laracasts 上有提供這個功能的免費[教學影片的網站](https://laracasts.com/series/whats-new-in-laravel-5-5/episodes/2)。

`Illuminate\Http\Request` 物件現在提供了一個 `validate` 方法，可以讓你更快地從路由閉包或控制器中驗證傳進來的請求：

    use Illuminate\Http\Request;

    Route::get('/comment', function (Request $request) {
        $request->validate([
            'title' => 'required|string',
            'body' => 'required|string',
        ]);

        // ...
    });

### 更一致的例外處理

現在整個框架的驗證例外處理是可被集中規範的。在以前，框架中有幾多個地方需要透過自訂來更改 JSON 驗證錯誤回應的預設格式。另外，Laravel 5.5 中的 JSON 驗證回應的預設格式目前遵循以下規範：

    {
        "message": "The given data was invalid.",
        "errors": {
            "field-1": [
                "Error 1",
                "Error 2"
            ],
            "field-2": [
                "Error 1",
                "Error 2"
            ],
        }
    }

所有 JSON 驗證錯誤格式都能透過定義 `App\Exceptions\Handler` 類別的一個方法來控制。例如，以下自訂內容會使用 Laravel 5.4 規範來格式化 JSON 驗證回應。

    use Illuminate\Validation\ValidationException;

    /**
     * 將驗證例外轉換成 JSON 回應。
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \Illuminate\Validation\ValidationException  $exception
     * @return \Illuminate\Http\JsonResponse
     */
    protected function invalidJson($request, ValidationException $exception)
    {
        return response()->json($exception->errors(), $exception->status);
    }

### 快取鎖定

Redis 和 Memcached 快取驅動現在支援獲得與釋放原子「鎖」。這提供一個簡單獲得屬性所的方法，而不用擔心其他條件。例如，在處理任務之前，你可能希望獲得一個鎖定，來確保沒有線程在執行相同的任務：

    if (Cache::lock('lock-name', 60)->get()) {
        // 將鎖定 60 秒，再繼續處理...

        Cache::lock('lock-name')->release();
    } else {
        // 無法鎖定...
    }

或者，你可以將閉包傳入 `get` 方法。只有在鎖定的情況下才會執行閉包，執行閉包後就會自動釋放鎖定：

    Cache::lock('lock-name', 60)->get(function () {
        // 將鎖定 60 秒...
    });

此外，你可以「阻礙」鎖定，直到可以使用為止：

    if (Cache::lock('lock-name', 60)->block(10)) {
        // 最多等待十秒鐘，直到可以使用鎖定為止...
    }

### Blade 模板改良

> {video} Laracasts 上有提供這個功能的免費[教學影片的網站](https://laracasts.com/series/whats-new-in-laravel-5-5/episodes/10)。

在定義簡單的自定義條件語句時，對自定的指令進行撰寫有時比所需的更為複雜。因此，Blade 現在提供了一個 `Blade::if` 方法，可以讓你使用閉包來快速定義自定條件指令。例如，讓我們定義一個自定條件來檢查當前的應用程式的環境。我們可以在我們的 `AppServiceProvider` 的 `boot` 方法中做到這一點：

    use Illuminate\Support\Facades\Blade;

    /**
     * 處理服務註冊後啟動。
     *
     * @return void
     */
    public function boot()
    {
        Blade::if('env', function ($environment) {
            return app()->environment($environment);
        });
    }

一旦定義了自訂的條件。我們就能輕易的在自己的模板上使用它：

    @env('local')
        // 應用程式在本機的環境...
    @else
        // 應用程式不在本機的環境...
    @endenv

除了可以輕易地自訂 Blade 條件指令外，還新增了更方便的指令來快速檢查當前使用者的的驗證狀態：

    @auth
        // 該使用者已認證過...
    @endauth

    @guest
        // 該使用者未認證過...
    @endguest

### 新的路由方法

> {video} Laracasts 上有提供這個功能的免費[教學影片的網站](https://laracasts.com/series/whats-new-in-laravel-5-5/episodes/16)。

如果你正在定義一個用來重導到另一個 URI 的路由，你現在可以使用 `Route::redirect` 方法。這個方法提供了一個便捷的方式，所以你不必為了執行簡單的重導而定義完整的路由或控制器：

    Route::redirect('/here', '/there', 301);

如果你的路由只需要回傳一個視圖，你現在可以使用 `Route::view` 方法。就像是使用 `redirect` 方法一樣，這個方法提供了一個簡易的使用方式，所以你不必定義一個完整的路由或控制器。`view` 方法接受一個 URI 作為第一個參數，一個視圖名稱作為其第二個參數。另外，你可以提供一個資料陣列作為可選的第三個參數傳入視圖：

    Route::view('/welcome', 'welcome');

    Route::view('/welcome', 'welcome', ['name' => 'Taylor']);

### "Sticky" 資料庫連線

#### `sticky` 選項

在設定資料庫連線的讀寫分離時，可以使用新的 `sticky` 設定選項：

    'mysql' => [
        'read' => [
            'host' => '192.168.1.1',
        ],
        'write' => [
            'host' => '196.168.1.2'
        ],
        'sticky'    => true,
        'driver'    => 'mysql',
        'database'  => 'database',
        'username'  => 'root',
        'password'  => '',
        'charset' => 'utf8mb4',
        'collation' => 'utf8mb4_unicode_ci',
        'prefix'    => '',
    ],

`sticky` 選項是一個*可選的*值，可被用於在當前請求週期內立刻讀取已寫入資料庫的記錄。如果在當前請求週期中啟用了 `sticky` 選項並對資料庫執行了「寫入」的操作，則任何進一步的「讀取」操作都會使用「寫入」的連接。這可以確保在請求週期內寫入的任何資料可以在同一請求期間即時從資料庫讀取。由你來決定這是否是你的應用程序所需的行為。
