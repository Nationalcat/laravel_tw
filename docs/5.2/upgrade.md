---
layout: post
title: upgrade
tag: 5.2
---
# 升級導引

- [從 5.1 升級到 5.2.0](#upgrade-5.2.0)
- [升級到 5.1.11](#upgrade-5.1.11)
- [升級到 5.1.0](#upgrade-5.1.0)
- [升級到 5.0.16](#upgrade-5.0.16)
- [從 4.2 升級到 5.0](#upgrade-5.0)
- [從 4.1 升級到 4.2](#upgrade-4.2)
- [從 4.1.x 以前版本升級到 4.1.29](#upgrade-4.1.29)
- [從 4.1.25 以前版本升級到 4.1.26](#upgrade-4.1.26)
- [從 4.0 升級到 4.1](#upgrade-4.1)

<a name="upgrade-5.2.0"></a>
## 從 5.1 升級到 5.2.0

#### 估計升級需時：少於 1 小時

> **注意：** 我們嘗試在每次框架進行重大改變時，提供非常全面的更新清單。然而，許多改變或許不適用於你的應用程式。

### 更新依賴套件

如果你有安裝 Laravel 5.2 beta 釋出版本，在你的 `composer.json` 檔案中加入 `"minimum-stability": "beta"`。

更新你的 `composer.json` 檔案到 `laravel/framework 5.2.*`。

在 `composer.json` 檔案的 `require-dev` 部分新增 `symfony/dom-crawler ~3.0` 和 `symfony/css-selector ~3.0`。

### 認證

#### 設定檔案

你應該根據 [https://github.com/laravel/laravel/blob/develop/config/auth.php](https://github.com/laravel/laravel/blob/master/config/auth.php) 來更新你的 `config/auth.php` 設定檔案：

一旦你更新檔案與副本，基於你的舊設定檔來設定所需要的認證檔案選項。如果你使用的是基於 Laravel 5.1 典型的 Eloquent 認證服務，大部分設定值應該保持不變。

在 Laravel 5.2 到視圖的預設路徑已經改變了，特別注意到新的 `auth.php` 設定檔案 `passwords.users.email` 設定檔案選項和驗證視圖路徑對應於你的實際視圖路徑。如果新的設定檔預設值沒有對應到你現有的視圖，請更新設定檔案選項。

#### Contracts

如果你要實作 `Illuminate\Contracts\Auth\Authenticatable` contract 而**不是**使用 `Authenticatable` trait，你應該新增一個新的 `getAuthIdentifierName` 方法到你的 contract 實作。通常來說，這個方法會回傳「Primary Key」的欄位名稱的認證實體。例如：`id`。

除非你手動的去實作這個介面，不然通常不會影響你的應用程式。

#### 客製化驅動程式

如果你使用 `Auth::extend` 方法來定義一個取得使用者的客製化方法，你現在應該使用 `Auth::provider` 來定義你的客製化使用者提供者。一旦你定義了客製化的提供者，你可以設定新的 `auth.php` 設定檔內的 `providers` 陣列設定。

更多關於客製化認證提供者，參考[完整的認證文件](/laravel_tw/docs/5.2/authentication)。

### 授權

`Illuminate\Auth\Access\UnauthorizedException` 被重新命名成 `Illuminate\Auth\Access\AuthorizationException`。如果你沒有手動捕捉例外的話，通常不會影響到你的應用程式。

### 集合

#### Eloquent 基礎集合

Eloquent 集合實例現在回傳一個基礎的集合（`Illuminate\Support\Collection`），有以下幾種方法：`pluck`、`keys`、`zip`、`collapse`、`flatten`、`flip`。

#### Key 保留

在集合中，現在 `slice`、`chunk` 和 `reverse` 方法會保留 key。如果你不想要這些方法保留 key，在你的 `Collection` 實例使用 `values` 方法

### Composer 類別

`Illuminate\Foundation\Support\Composer` 類別現在已經被移到 `Illuminate\Support\Composer`。如果你沒有手動使用這些類別，通常不會影響到你的應用程式。

### 命令和處理程序

#### 自行處理命令

在你的工作和命令不再需要實作 `SelfHandling` contract 。預設情況下所有的工作都會自行處理，所以你可以從你的類別中移除這個介面。

#### 分離命令和處理程序

Laravel 5.2 命令匯流現在只支援本身的處理命令，不再支援分離的命令和處理程序。

如果想要繼續使用分離命令和處理程序，你可以安裝 Laravel Collective 的套件，它提供向下兼容的支援：[https://github.com/LaravelCollective/bus](https://github.com/laravelcollective/bus)。

### 設定檔

#### 環境變數

新增一個 `env` 設定選項到你的 `app.php` 設定檔，如下：

    'env' => env('APP_ENV', 'production'),

#### 快取和環境

如果你在部署時使用 `config:cache` 命令，你**必須**確保你是從你的設定檔中呼叫 `env` 函式，而不是從應用程式其他地方。

如果你要從應用程式中呼叫 `env`，強烈建議你新增適合的設定到你的設定檔案，呼叫本機端的 `env` 來替代，轉換你的 `env` 到 `config`。

### CSRF 驗證

當你執行單元測試時，不會自動執行 CSRF 驗證。通常這不會影響到你的程式。

### Elixir

`elixir` PHP 函式現在回傳一個完整的 URL 而不是一個相對的 URL。除非你手動轉換這些 URLs 成完整 URLs，不然通常不會影響你的應用程式。

### Eloquent

#### 日期轉換

任何已經加入到你的 `$casts` 屬性作為 `date` 或 `datetime`，當模型或是模型的集合呼叫 `toArray` 會將它轉換成一個字串。這可以使你 `$dates` 陣列內指定的日期轉換一致。

#### 全域範圍

全域範圍實作是為了讓改寫可以更簡單的使用。你的全域範圍不需要 `remove` 方法，因此它可能從任何你所寫的全域範圍中被移除。

如果在 Elquent 查詢構造器呼叫 `getQuery` 來存取底層的查詢構造器實例，現在應該稱為 `toBase`。

如果你因為任何原因直接呼叫 `remove` 方法，你應該更改 `$eloquentBuilder->withoutGlobalScope($scope)` 的呼叫方式。

任何呼叫 `$model->removeGlobalScopes($builder)` 可以改成簡單的 `$builder->withoutGlobalScopes()`。

### 事件

#### 核心事件物件

Laravel 一些核心事件的觸發現在是使用事件物件而不是字串事件命名和動態參數。以下是基於舊事件名稱的新物件列表：

舊  | 新
------------- | -------------
`auth.attempting`  |  `Illuminate\Auth\Events\Attempting`
`auth.login`  |  `Illuminate\Auth\Events\Login`
`auth.logout`  |  `Illuminate\Auth\Events\Logout`
`cache.missed`  |  `Illuminate\Cache\Events\CacheMissed`
`cache.hit`  |  `Illuminate\Cache\Events\CacheHit`
`cache.write`  |  `Illuminate\Cache\Events\KeyWritten`
`cache.delete`  |  `Illuminate\Cache\Events\KeyForgotten`
`connection.{name}.beginTransaction`  |  `Illuminate\Database\Events\TransactionBeginning`
`connection.{name}.committed`  |  `Illuminate\Database\Events\TransactionCommitted`
`connection.{name}.rollingBack`  |  `Illuminate\Database\Events\TransactionRolledBack`
`illuminate.query`  |  `Illuminate\Database\Events\QueryExecuted`
`illuminate.queue.after`  |  `Illuminate\Queue\Events\JobProcessed`
`illuminate.queue.failed`  |  `Illuminate\Queue\Events\JobFailed`
`illuminate.queue.stopping`  |  `Illuminate\Queue\Events\WorkerStopping`
`mailer.sending`  |  `Illuminate\Mail\Events\MessageSending`
`router.matched`  |  `Illuminate\Routing\Events\RouteMatched`

在 Laravel 5.1 每個事件物件包含**完全**相同參數被傳送到事件處理程序。例如，假設你在 5.1.* 使用 `DB::listen` 你可以在 5.2.* 中更新你的程式碼像是：

    DB::listen(function ($event) {
        dump($event->sql);
        dump($event->bindings);
    });

你可以查看每個新的事件物件類別它們的公用屬性。

### 例外處理

你的 `App\Exceptions\Handler` 類別 `$dontReport` 屬性應該被更新，至少包含以下的例外類型：

    use Illuminate\Auth\Access\AuthorizationException;
    use Illuminate\Database\Eloquent\ModelNotFoundException;
    use Symfony\Component\HttpKernel\Exception\HttpException;
    use Illuminate\Foundation\Validation\ValidationException;

    /**
     * 不需要報告的例外類型的清單。
     *
     * @var array
     */
    protected $dontReport = [
        AuthorizationException::class,
        HttpException::class,
        ModelNotFoundException::class,
        ValidationException::class,
    ];

### 顯式模型綁定

Laravel 5.2 包含「顯式模型綁定」，一個方便基於你的 URI 標示自動地注入模型實例到你的路由和控制器的新功能。然而，這個改變了路由和控制器的類型提示模型實例的行為。

如果你要在路由或控制器類型提示一個模型實例，並期望一個**空的**模型實例可以被注入，你應該移除類型提示並直接建立一個空的模型實例到你的路由或控制器；否則，Laravel 會基於路由的 URI 標示，嘗試從資料庫取得存在的模型實例。

### IronMQ

IronMQ 隊列驅動已經被移出獨立成一個套件了，核心框架已經不附帶。

[http://github.com/LaravelCollective/iron-queue](http://github.com/laravelcollective/iron-queue)

### 任務和隊列

預設上，`php artisan make:job` 指令現在可以建立一個定義「隊列任務」工作類別。如果你想要建立一個「同步」的任務，當你使用命令時使用 `--sync` 選項。

### 郵件

`pretend` 郵件設定選項已經被移除了。相反的，使用 `log` 郵件驅動可以執行像是 `pretend` 的函式和記錄更多關於郵件訊息的資訊。

### 分頁

為了符合由框架產生的其他 URLs，分頁 URLs 後面不再包含一個斜線，這不太會影響到你的應用程式。

### 服務提供者

`Illuminate\Foundation\Providers\ArtisanServiceProvider` 可能從你的 `app.php` 設定檔案的服務提供者列表被移除了。

`Illuminate\Routing\ControllerServiceProvider` 可能從你的 `app.php` 設定檔案的服務提供者列表被移除了。

### Sessions

#### 資料庫 Session 驅動

一個新的 `database` session 驅動已經為框架撰寫包括像是使用者 ID、IP、地址、以及使用者代理的詳細資訊。如果你想要繼續使用舊的驅動，你可以在你的 `session.php` 設定檔案內指定 `legacy-database` 驅動。

如果你想要使用新的驅動，你應該新增 `user_id（可為空的整數）`、`ip_address（可為空的字串）` 以及 `user_agent（文字）` 欄位到你的 session 資料庫的資料表。

### Stringy

「Stringy」libray 已經不包含在框架了。如果你想要在你的應用程式使用，你可以透過 Composer 來手動安裝。

### 驗證

`ValidatesRequests` trait 現在會拋出一個 `Illuminate\Foundation\Validation\ValidationException` 實例，而不是拋出一個 `Illuminate\Http\Exception\HttpResponseException` 實例。

### 棄用功能

以下的功能在 Laravel 5.2 中已經被棄用，在 2016 年 6 月的 5.3 版本中會被移除：

- `Illuminate\Contracts\Bus\SelfHandling` contract 從任務中移除。
- 在集合的 `lists` 方法，查詢構造器和 Eloquent 查詢構造器物件會被重新命名成 `pluck`。方法的署名還是不變。
- 使用 `Route::controller` 的顯式控制器路由已經被棄用。請在你的路由檔案中使用顯式路由註冊。這有可能會被提取成一個套件。
- 從 Laravel 5.1 中 `database` session 驅動已經被重新命名成 `legacy-database` ，它將會被移除。 在前面有更多的「資料庫 session 驅動」參考說明資訊。
- `Str::randomBytes` 函式已經被棄用，而使用原生的 PHP `random_bytes` 函式做取代。
- `Str::equals` 函式已經被棄用，而使用原生的 PHP `hash_equals` 函式做取代。
- `Illuminate\View\Expression` 已經被棄用，而使用 `Illuminate\Support\HtmlString` 取代。

<a name="upgrade-5.1.11"></a>
## 升級到 5.1.11

Laravel 5.1.11 包含了對於[授權](/laravel_tw/docs/5.2/authorization)及[原則](/laravel_tw/docs/5.2/authorization#policies)的支援。要將這些功能新增到你現有的 Laravel 5.1 應用程式是相當容易的。

> **注意：**這些升級是**可選的**，忽略它們並不會影響你的應用程式。

#### 建立 Policies 目錄

首先，在你的應用程式建立一個空的 `app/Policies` 目錄。

#### 建立並註冊 AuthServiceProvider 與 Gate Facade

在你的 `app/Providers` 目錄建立一個 `AuthServiceProvider`。你可以複製[從 GitHub](https://raw.githubusercontent.com/laravel/laravel/master/app/Providers/AuthServiceProvider.php) 提供的預設內容。切記，如果你的應用程式使用自定的命名空間，請修改提供者的命名空間。在建立提供者之後，請務必在你的 `app.php` 設定檔的 `providers` 陣列註冊它。

同樣的，你必須在你的 `app.php` 設定檔的 `aliases` 陣列註冊 `Gate` facade：

    'Gate' => Illuminate\Support\Facades\Gate::class,

#### 更新使用者模型

再來，在你的 `App\User` 模型使用 `Illuminate\Foundation\Auth\Access\Authorizable` trait 及 `Illuminate\Contracts\Auth\Access\Authorizable` contract：

    <?php

    namespace App;

    use Illuminate\Auth\Authenticatable;
    use Illuminate\Database\Eloquent\Model;
    use Illuminate\Auth\Passwords\CanResetPassword;
    use Illuminate\Foundation\Auth\Access\Authorizable;
    use Illuminate\Contracts\Auth\Authenticatable as AuthenticatableContract;
    use Illuminate\Contracts\Auth\Access\Authorizable as AuthorizableContract;
    use Illuminate\Contracts\Auth\CanResetPassword as CanResetPasswordContract;

    class User extends Model implements AuthenticatableContract,
                                        AuthorizableContract,
                                        CanResetPasswordContract
    {
        use Authenticatable, Authorizable, CanResetPassword;
    }

#### 更新基礎控制器

接著，更新你基礎的 `App\Http\Controllers\Controller` 控制器，讓它使用 `Illuminate\Foundation\Auth\Access\AuthorizesRequests` trait：

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Foundation\Bus\DispatchesJobs;
    use Illuminate\Routing\Controller as BaseController;
    use Illuminate\Foundation\Validation\ValidatesRequests;
    use Illuminate\Foundation\Auth\Access\AuthorizesRequests;

    abstract class Controller extends BaseController
    {
        use AuthorizesRequests, DispatchesJobs, ValidatesRequests;
    }

<a name="upgrade-5.1.0"></a>
## 升級到 5.1.0

#### 估計升級需時：少於 1 小時

### 更新 `bootstrap/autoload.php`

將 `bootstrap/autoload.php` 裡的 `$compiledPath` 變數按照以下方式更新：

    $compiledPath = __DIR__.'/cache/compiled.php';

### 建立 `bootstrap/cache` 目錄

在你的 `bootstrap` 目錄裡，建立一個 `cache` 目錄 (`bootstrap/cache`)。放置一個有以下內容的 `.gitignore` 檔案在這個目錄：

    *
    !.gitignore

這個目錄必須為可寫入的，框架會暫時存放如 `compiled.php`、`routes.php`、`config.php` 和 `services.json` 的最佳化檔案在此目錄。

### 增加 `BroadcastServiceProvider` 提供者

在你的 `config/app.php` 設定檔裡面，增加 `Illuminate\Broadcasting\BroadcastServiceProvider` 到 `providers` 陣列裡。

### 認證

如果你有使用內建的 `AuthController`，他使用了 `AuthenticatesAndRegistersUsers` trait，你會需要對新使用者如何建立跟驗證做一些修改。

首先，你不再需要傳遞 `Guard` 和 `Registrar` 實例到基底建構子。你可以從控制器的建構子完全移除這些依賴。

第二，已經不再需要 Laravel 5.0 中使用的 `App\Services\Registrar` 類別。你可以簡單的從這個類別直接複製你的 `validator` 和 `create` 方法貼上至你的 `AuthController`。這些方法不需要做其他修改。然而，你必須確定有在你的 `AuthController` 頂端引入 `Validator` facade 跟你的 `User` 模型。

#### 密碼控制器

內建的 `PasswordController` 的建構子不再需要任何依賴。你可以把 5.0 下需要的依賴都移除。

### 驗證

如果你在基底控制器類別上覆寫了 `formatValidationErrors` 方法，你現在應該把型別提示改成 `Illuminate\Contracts\Validation\Validator` contract 來取代具體的 `Illuminate\Validation\Validator` 實例。

同樣地，如果你在基底表單請求類別上覆寫了 `formatErrors` 方法，你現在應該把型別提示改成 `Illuminate\Contracts\Validation\Validator` contract 來取代具體的 `Illuminate\Validation\Validator` 實例。

### Eloquent

#### `create` 方法

Eloquent 的 `create` 方法現在可以不帶任何參數呼叫。如果你有在自己的模型覆寫了 `create` 方法，請把 `$attributes` 參數的預設值設定成空陣列：

    public static function create(array $attributes = [])
    {
        // 你的客製化實作
    }

#### `find` 方法

如果你有在自己的模型中覆寫了 `find` 方法並在你的方法中呼叫了 `parent::find()`，你現在應該把它改成呼叫 Eloquent 查詢產生器上的 `find` 方法：

    public static function find($id, $columns = ['*'])
    {
        $model = static::query()->find($id, $columns);

        // ...

        return $model;
    }

#### `lists` 方法

`lists` 方法現在回傳一個 `Collection` 實例而不是 Eloquent 查詢用的一般陣列。如果你想要把 `Collection` 轉換成一般陣列，請使用 `all` 方法：

    User::lists('id')->all();

必須小心查詢產生器的 `lists` 方法仍然是回傳一個陣列。

#### 日期格式

以前，Eloquent 日期欄位的儲存格式可以藉由覆寫模型上的 `getDateFormat` 方法來修改。這仍然是可行的。然而，方便起見你可以直接在模型上指定 `$dateFormat` 屬性來取代覆寫方法。

當序列化模型成 `array` 或 JSON 時，也會採用該日期格式。當從 Laravel 5.0 遷移到 5.1 時，這可能會改變你的 JSON 序列化的日期欄位格式。要針對序列化模型設定特定的日期格式，你可以在你的模型上覆寫 `serializeDate(DateTime $date)` 方法。這個方法讓你可以在不改變日期欄位儲存格式的情況下，精細的控制 Eloquent 序列化格式。

### 集合類別

#### `sort` 方法

`sort` 方法現在回傳全新的集合實例，而不是修改原有的集合：

    $collection = $collection->sort($callback);

#### `sortBy` 方法

`sortBy` 方法現在回傳一個全新的集合實例而不會去改動到既有的集合：

    $collection = $collection->sortBy('name');

#### `groupBy` 方法

`groupBy` 方法現在會回傳 `Collection` 實例給在父 `Collection` 中的每一個元素。如果你想要把所有元素轉換回一般陣列，你可以透過 `map` 處理它們：

    $collection->groupBy('type')->map(function($item)
    {
        return $item->all();
    });

#### `lists` 方法

`lists` 方法現在回傳一個 `Collection` 實例而不是一個一般陣列。如果你想要把 `Collection` 轉換成一般陣列，請使用 `all` 方法：

    $collection->lists('id')->all();

### 命令和處理程序

`app/Commands` 目錄已經被改名成 `app/Jobs`。然而，你不需要移動你所有的命令到新的位置，並且你可以繼續用 `make:command` 和 `handler:command` Artisan 命令來生成你的類別。

同樣地，`app/Handlers` 目錄已經被改名成 `app/Listeners` 並且現在只包含事件監聽者。然而，你不需要移動或重新命名你既有的命令和事件處理程序，而且你可以繼續使用 `handler:event` 命令來生成事件處理程序。

藉由提供對 Laravel 5.0 目錄結構的向下相容，你可以先升級你的應用程式到 Laravel 5.1，然後在你或你的團隊方便的時候慢慢地升級你的事件跟命令到它們的新位置。

### Blade

`createMatcher`、`createOpenMatcher` 和 `createPlainMatcher` 方法已經從 Blade 編譯器移除。在 Laravel 5.1，請使用新的 `directive` 方法來建立客製化的 Blade 標籤。請查看[擴展 blade](/laravel_tw/docs/5.2/blade#extending-blade) 文件來了解更多資訊。

### 測試

增加 protected `$baseUrl` 屬性到 `tests/TestCase.php` 檔案中：

    protected $baseUrl = 'http://localhost';

### 語系檔

第三方套件發佈語系檔的預設目錄已經改變。必須從 `resources/lang/packages/{locale}/{namespace}` 移動所有的第三方套件語系檔到 `resources/lang/vendor/{namespace}/{locale}` 目錄。例如，`Acme/Anvil` 套件命名空間為 `acme/anvil::foo` 的英文語系檔應該從 `resources/lang/packages/en/acme/anvil/foo.php` 移動到 `resources/lang/vendor/acme/anvil/en/foo.php`。

### Amazon 網路服務 SDK

如果你有使用 AWS SQS 隊列驅動或 AWS SES 電子郵件驅動，你應該升級你安裝的 AWS PHP SDK 到 3.0 版本。

如果你有使用 Amazon S3 檔案系統驅動，你將需要藉由 Composer 更新對應的檔案系統套件：

- Amazon S3: `league/flysystem-aws-s3-v3 ~1.0`

### 棄用的功能

以下的 Laravel 功能已經被棄用並將會在 2015 十二月釋出的 Laravel 5.2 中完全地移除：

<div class="content-list" markdown="1">
- 路由篩選器已經被棄用而偏好使用[中介層](/laravel_tw/docs/5.2/middleware)。
- `Illuminate\Contracts\Routing\Middleware` contract 已經被棄用。你的中介層上不需要任何 contract。此外，`TerminableMiddleware` contract 也已經被棄用。不要實作介面，簡單地定義一個 `terminate` 方法在你的中介層上就好。
- `Illuminate\Contracts\Queue\ShouldBeQueued` contract 已經被棄用而用 `Illuminate\Contracts\Queue\ShouldQueue` 取代。
- Iron.io 的「推送隊列」已經被棄用而用一般的 Iron.io 隊列和[隊列監聽者](/laravel_tw/docs/5.2/queues#running-the-queue-listener)取代。
- `Illuminate\Foundation\Bus\DispatchesCommands` trait 已經被棄用並改名成 `Illuminate\Foundation\Bus\DispatchesJobs`。
- `Illuminate\Container\BindingResolutionException` 被移到 `Illuminate\Contracts\Container\BindingResolutionException`。
- 服務容器的 `bindShared` 方法已經被棄用而用 `singleton` 方法取代。
- Eloquent 和查詢產生器的 `pluck` 方法已經被棄用並改名成 `value`。
- 集合的 `fetch` 方法已經被棄用而用 `pluck` 方法取代。
- `array_fetch` 輔助函式已經被棄用而用 `array_pluck` 方法取代。
</div>

<a name="upgrade-5.0.16"></a>
## 升級到 5.0.16

在你的 `bootstrap/autoload.php` 檔案中，更新 `$compiledPath` 變數成：

    $compiledPath = __DIR__.'/../vendor/compiled.php';

<a name="upgrade-5.0"></a>
## 從 4.2 升級到 5.0

### 全新安裝，然後遷移

推薦的升級方式是建立一個全新的 Laravel `5.0` 專案，然後複製你 `4.2` 網站特定的應用程式檔案到此新的應用程式。這將包含控制器、路由、Eloquent 模型、Artisan 指令、資源檔，和其他專屬於你的應用程式的程式碼。

開始前，在你的本地環境中[安裝一個新的 Laravel 5 應用程式](/docs/5.0/installation)到一個全新的目錄中。不要安裝超過 5.0 的任何版本，因為我們需要先完成遷移至 5.0 的步驟。我們將會在後面詳細探討各部分的遷移過程。

### Composer 相依與套件

別忘了複製任何額外的 Composer 相依套件到你的 5.0 應用程式內。這包含第三方程式碼，例如 SDKs。

部分 Laravel 專用套件也許不相容於剛釋出的 Laravel 5。請向套件的維護者確認該套件支援 Laravel 5 的版本。一旦你加入應用程式需要的任何額外 Composer 相依套件後，請執行 `composer update`。

### 命名空間

預設情況下，Laravel 4 應用程式 沒有在應用程式的程式碼中使用命名空間。所以，舉例來說，所有的 Eloquent 模型和控制器都簡單地存在於「全域」的命名空間中。為了更快速的遷移，Laravel 5 也允許您可以將這些類別一樣保留在全域的命名空間

### 設定

#### 遷移環境變數

複製新的 `.env.example` 檔案到 `.env`，在 `5.0` 這相當於原本的 `.env.php`。設定所有該有的設定值，像是 `APP_ENV` 和 `APP_KEY` (你的加密金鑰)、資料庫憑證、快取驅動與 session 驅動。

此外，將你原本的 `.env.php` 檔案中自訂的設定值都複製並搬到 `.env` (本機環境的實際設定值) 和 `.env.example` (給其他團隊成員的範本教學)。

更多關於環境設定的資訊，請查看[完整文件](/laravel_tw/docs/5.2/installation#environment-configuration)。

> **注意：**在部署你的 Laravel 5 應用程式之前，你需要在正式主機上放置 `.env` 檔案並設定適當的值。

#### 設定檔

Laravel 5.0 不再使用 `app/config/{environmentName}/` 目錄結構來提供對應該環境的設定檔。取而代之的是，將環境對應的各種設定值移到 `.env`，並接著藉由 `env('key', 'default value')` 來取用在設定檔的值。你可以在 `config/database.php` 設定檔看到相關範例。

將設定檔放在 `config/` 目錄下，來代表所有環境共用的設定檔，或是在使用 `env()` 來取得對應該環境的設定值。

請記住，如果你增加了其他的鍵值到 `.env` 檔案中，同時也要加範例值到 `.env.example` 檔。這將可以幫助其他團隊成員建立他們自己的 `.env` 檔。

### 路由

複製貼上你原本的 `routes.php` 檔案到 `app/Http/routes.php`。

### 控制器

下一步，將你所有的控制器移到 `app/Http/Controllers` 目錄下。因為在本指南中我們不打算遷移到完整的命名空間，請將 `app/Http/Controllers` 目錄增加到 `composer.json` 的 `classmap` 屬性中。接下來，你可以從 `app/Http/Controllers/Controller.php` 基底抽象類別中移除命名空間。請確認你遷移過來的控制器都繼承這個基底類別。

在你的 `app/Providers/RouteServiceProvider.php` 檔案，設定 `namespace` 屬性為 `null`。

### 路由篩選器

將篩選邏輯綁定從 `app/filters.php` 複製到 `app/Providers/RouteServiceProvider.php` 的 `boot()` 方法。並在 `app/Providers/RouteServiceProvider.php` 增加 `use Illuminate\Support\Facades\Route;` 來繼續使用 `Route` Facade。

您不需要移動任何 Laravel 4.0 的預設過濾器，像是 `auth` 和 `csrf`。他們已經內建其中，只是換作以中介層形式出現。那些在路由或控制器內有使用到舊有預設過濾器  (例如，`['before' => 'auth']`) 請修改使用新的中介層 (例如，`['middleware' => 'auth']`)。

篩選器在 Laravel 5 中沒有被移除。你仍然可以綁定並藉由 `before` 和 `after`使用你自己客製化的篩選器。

### 全域 CSRF

預設情況下，所有路由都會啟用[CSRF 保護](/laravel_tw/docs/5.2/routing#csrf-protection)。如果你想要關閉它們或是只想在特定路由手動開啟，請從 `App\Http\Kernel` 的 `middleware` 陣列移除這行：

    'App\Http\Middleware\VerifyCsrfToken',

如果你想要在其他地方使用它，增加此行到 `$routeMiddleware`：

    'csrf' => 'App\Http\Middleware\VerifyCsrfToken',

現在你可以於路由內使用 `['middleware' => 'csrf']` 增加中介層到各別路由/控制器。要了解更多關於中介層的資訊，請查看[完整文件](/laravel_tw/docs/5.2/middleware)。

### Eloquent 模型

你可以自由地建立一個新的 `app/Models` 目錄來放置你的 Eloquent 模型。同樣地，必須把這個目錄加到 `composer.json` 檔案的 `classmap` 屬性。

更新任何現在有使用 `SoftDeletingTrait` 的模型改用 `Illuminate\Database\Eloquent\SoftDeletes`。

#### Eloquent 快取

Eloquent 不再提供 `remember` 方法來快取查詢結果。現在你有責任手動地使用 `Cache::remember` 方法快取查詢結果。要了解更多關於快取的資訊，請查看[完整文件](/laravel_tw/docs/5.2/cache)。

### 使用者認證模型

要使用 Laravel 5 的認證系統，請遵循以下指引來升級您的 `User` 模型：

**從你的 `use` 區塊刪除以下內容：**

```php
use Illuminate\Auth\UserInterface;
use Illuminate\Auth\Reminders\RemindableInterface;
```

**增加以下內容到你的 `use` 區塊：**

```php
use Illuminate\Auth\Authenticatable;
use Illuminate\Auth\Passwords\CanResetPassword;
use Illuminate\Contracts\Auth\Authenticatable as AuthenticatableContract;
use Illuminate\Contracts\Auth\CanResetPassword as CanResetPasswordContract;
```

**移除 UserInterface 和 RemindableInterface 介面。**

**標記類別實作以下介面：**

```php
implements AuthenticatableContract, CanResetPasswordContract
```

**在類別宣告引入以下 traits：**

```php
use Authenticatable, CanResetPassword;
```

**如果你有使用它們，請把 `Illuminate\Auth\Reminders\RemindableTrait` 和 `Illuminate\Auth\UserTrait` 從你的 use 區塊和類別宣告中移除。**

### Cashier 的使用者需要做的修改

[Laravel Cashier](/laravel_tw/docs/5.2/billing) 使用的 trait 和介面名稱已作修改。trait 請改用 `Laravel\Cashier\Billable` 取代 `BillableTrait`。介面請實作 `Laravel\Cashier\Contracts\Billable` 取代 `Laravel\Cashier\BillableInterface`。不需要修改其他任何方法。

### Artisan 命令

將你所有的命令類別從原本的 `app/commands` 目錄移到新的 `app/Console/Commands` 目錄。接下來，把 `app/Console/Commands` 目錄增加到 `composer.json` 檔案的 `classmap` 屬性中。

然後，把你的 Artisan 命令清單從 `start/artisan.php` 複製到 `app/Console/Kernel.php` 檔案裡的 `command` 陣列中。

### 資料庫遷移和資料填充

因為你的資料庫裡應該已經有 users 表了，請刪除 Laravel 5.0 內建的兩個遷移檔。

將你所有的遷移檔從原本的 `app/database/migrations` 目錄移到新的 `database/migrations`。你所有的資料填充檔也應該從 `app/database/seeds` 移到 `database/seeds`。

### 全域 IoC 綁定

如果你在 `start/global.php` 有任何的[服務容器](/laravel_tw/docs/5.2/container)綁定，請將它們全部移至 `app/Providers/AppServiceProvider.php` 檔案的 `register` 方法。你可能需要引入 `App` facade。

你也可以選擇將這些綁定，依照類別拆分到各別的服務提供者中。

### 視圖

將你的視圖從 `app/views` 移到新的 `resources/views` 目錄。

### 語系檔

將你的語系檔從 `app/lang` 移到新的 `resources/lang` 目錄。

### 公開目錄

從你的 `4.2` 應用程式的 `public` 目錄內複製公開的 assets 到新應用程式的 `public` 目錄內。請確定有保留 `5.0` 版本的 `index.php`。

### 測試

將你的測試從 `app/tests` 移到新的 `tests` 目錄。

### 雜項檔案

複製專案內任何其他的檔案，例如： `.scrutinizer.yml`, `bower.json` 以及其他類似的工具設定檔。

你可以將 Sass、Less 或 CoffeeScript 移動到任何你想放置的地方。`resources/assets` 目錄是一個不錯的預設位置。

### 表單和 HTML 輔助函式

如果你使用表單或 HTML 輔助函數，你將會看到 `class 'Form' not found` 或 `class 'Html' not found` 的錯誤。表單和 HTML 輔助函式已經在 Laravel 5.0 中被棄用。然而，有些社群導向的替代品，例如：[Laravel Collective](http://laravelcollective.com/laravel_tw/docs/5.2/html) 維護的這些。

舉例來說，你可以把 `"laravelcollective/html": "~5.0"` 增加到你的 `composer.json` 的 `require` 區塊。

你也需要表單和 HTML 的 facades 以及服務提供者。編輯 `config/app.php` 並增加這行到 'providers' 陣列內：

    'Collective\Html\HtmlServiceProvider',

接著，增加這幾行到 'aliases' 陣列內：

    'Form' => 'Collective\Html\FormFacade',
    'Html' => 'Collective\Html\HtmlFacade',

### 快取管理員

如果你的程式碼有注入 `Illuminate\Cache\CacheManager` 來取得非 Facade 版本的 Laravel 快取，請改成注入 `Illuminate\Contracts\Cache\Repository` 取代。

### 分頁

置換所有的 `$paginator->links()` 為 `$paginator->render()`。

分別地置換所有的 `$paginator->getFrom()` 和 `$paginator->getTo()` 為 `$paginator->firstItem()` 和 `$paginator->lastItem()`。

從 `$paginator->getPerPage()`、`$paginator->getCurrentPage()`、`$paginator->getLastPage()` 和 `$paginator->getTotal()` 移除「get」前綴 (例如：`$paginator->perPage()`)。

### Beanstalk 隊列

Laravel 5.0 現在需要 `"pda/pheanstalk": "~3.0"` 取代原本的 `"pda/pheanstalk": "~2.1"`。

### Remote

Remote 元件已被棄用。

### Workbench

Workbench 元件已被棄用。

<a name="upgrade-4.2"></a>
## 從 4.1 升級到 4.2

### PHP 5.4+

Laravel 4.2 需要 PHP 5.4.0 或更高的版本。

### 預設加密

在你的 `app/config/app.php` 設定檔中增加一個新的 `cipher` 選項。這個選項的值應該為 `MCRYPT_RIJNDAEL_256`。

    'cipher' => MCRYPT_RIJNDAEL_256

這個設定可以被用來調整 Laravel 加密工具使用的預設加密方法。

> **注意：**在 Laravel 4.2，預設的加密方法為 `MCRYPT_RIJNDAEL_128` (AES)，它認為是最安全的加密方法。必須將加密方法改回 `MCRYPT_RIJNDAEL_256` 來解密在 Laravel 4.1 以前版本下加密的 cookies 和值。

### 現在使用 Traits 在可以軟刪除的模型上

如果你有使用可以軟刪除的模型，`softDeletes` 屬性已經被移除。現在你必須使用 `SoftDeletingTrait` 如下：

    use Illuminate\Database\Eloquent\SoftDeletingTrait;

    class User extends Eloquent
    {
        use SoftDeletingTrait;
    }

你也必須手動地增加 `deleted_at` 欄位到你的 `dates` 屬性中：

    class User extends Eloquent
    {
        use SoftDeletingTrait;

        protected $dates = ['deleted_at'];
    }

而所有軟刪除的 API 使用方式維持相同。

> **注意：**`SoftDeletingTrait` 無法在基底模型下被使用。他必須用在一個實際的模型類別。

### 視圖 / 分頁的環境名稱變更

如果你直接參照 `Illuminate\View\Environment` 或 `Illuminate\Pagination\Environment` 類別， 請更新你的程式碼將其改為參照 `Illuminate\View\Factory` 和 `Illuminate\Pagination\Factory`。改名後的這兩個類別更可以代表他們的功能。

### 額外的參數 On Pagination Presenter

如果你擴展了 `Illuminate\Pagination\Presenter` 類別，抽象方法 `getPageLinkWrapper` 的參數列變成要加上 `rel` 參數：

    abstract public function getPageLinkWrapper($url, $page, $rel = null);

### Iron.Io 隊列加密

如果你有使用 Iron.io 隊列驅動，你需要增加一個新的 `encrypt` 選項到你的 queue 設定檔中：

    'encrypt' => true

<a name="upgrade-4.1.29"></a>
## 從 4.1.x 以前版本升級到 4.1.29

Laravel 4.1.29 對於所有的資料庫驅動加強了 column quoting 的部分。當你的模型中**沒有**使用 `fillable` 屬性時，他保護你的應用程式不會受到 mass assignment 漏洞影響。如果你在模型中使用 `fillable` 屬性來防範 mass assignment，你的應用程式將不會有漏洞。然而，如果你使用 `guarded` 並傳遞使用者控制的陣列到「更新」或「儲存」類型的函式中，那你應該立即升級到 `4.1.29` 以避免你的應用程式遭受 mass assignment 的風險。

要升級到 Laravel 4.1.29，只要 `composer update` 即可。在這個發行版本中沒有重大的更新。

<a name="upgrade-4.1.26"></a>
## 從 4.1.25 以前版本升級到 4.1.26

Laravel 4.1.26 採用了針對「記得我」cookies 的安全性更新。在此更新之前，如果記得我 cookies 被惡意使用者劫持，該 cookie 將還可以生存很長一段時間，即使真實的用戶重設密碼或者登出等等後還是一樣。

此更動需要在你的 `users` (或同等的) 資料表中增加一個額外的 `remember_token` 欄位。在更新之後，當用戶每次登入你的應用程式將會被給予一個全新的 token。在使用者登出應用程式後，這個 token 也會被更新。這個更新的影響為：如果一個「記得我」的 cookie 被劫持，只要使用者登出應用程式該 cookie 將會失效。

### 升級路徑

首先，增加一個新的 `remember_token` 欄位到你的 `users` 資料表中，它可為空值且為 VARCHAR(100)、TEXT 或同等的類型。

然後，如果你是使用 Eloquent 認證驅動，請更新你的 `User` 類別的以下三個方法：

    public function getRememberToken()
    {
        return $this->remember_token;
    }

    public function setRememberToken($value)
    {
        $this->remember_token = $value;
    }

    public function getRememberTokenName()
    {
        return 'remember_token';
    }

> **注意：**所有現存的「記得我」sessions 在此更新後將會失效，所以應用程式的所有使用者將會被迫重新認證。

### 套件維護者

`Illuminate\Auth\UserProviderInterface` 介面加了兩個新的方法。簡單的實作可以在預設驅動中找到：

    public function retrieveByToken($identifier, $token);

    public function updateRememberToken(UserInterface $user, $token);

`Illuminate\Auth\UserInterface` 也增加了三個新方法並被描述在「升級路徑」部分。

<a name="upgrade-4.1"></a>
## 從 4.0 升級到 4.1

### 升級你的 Composer 相依

要升級你的應用程式至 Laravel 4.1，必須將你的 `composer.json` 裡的 `laravel/framework` 版本更改至 `4.1.*`。

### 置換檔案

置換你的 `public/index.php` 為[這個從 repository 複製的全新檔案](https://github.com/laravel/laravel/blob/v4.1.0/public/index.php)。

置換你的 `artisan` 為[這個從 repository 複製的全新檔案](https://github.com/laravel/laravel/blob/v4.1.0/artisan)。

### 新增設定檔案及選項

更新你在 `app/config/app.php` 設定檔裡的 `aliases` 和 `providers` 陣列。而這些陣列更新後的值可以在[這個檔案](https://github.com/laravel/laravel/blob/v4.1.0/app/config/app.php)中找到。請確定有把你自己和套件的服務提供者與別名加回陣列。

[從 repository](https://github.com/laravel/laravel/blob/v4.1.0/app/config/remote.php) 增加新的 `app/config/remote.php` 檔案。

在你的 `app/config/session.php` 檔案裡，增加新的 `expire_on_close` 設定選項。預設值應該為 `false`。

在你的 `app/config/queue.php` 檔案裡，增加新的 `failed` 設定區塊。以下為這區塊的預設值：

    'failed' => [
        'database' => 'mysql', 'table' => 'failed_jobs',
    ],

**(非必要)**  在你的 `app/config/view.php` 檔案裡，將 `pagination` 設定選項更新為 `pagination::slider-3`。

### 控制器的變更

如果 `app/controllers/BaseController.php` 有個 `use` 語句在最上面，將 `use Illuminate\Routing\Controllers\Controller;` 改為 `use Illuminate\Routing\Controller;`。

### 密碼提醒的變更

密碼提醒功能已經為了更大的彈性而大幅翻修。你可以執行 `php artisan auth:reminders-controller` Artisan 指令來檢查新的存根控制器。你也可以瀏覽[更新後的文件](/docs/security#password-reminders-and-reset)並相應地更新你的應用程式。

更新你的 `app/lang/en/reminders.php` 語系檔案來對應[這個新版檔案](https://github.com/laravel/laravel/blob/v4.1.0/app/lang/en/reminders.php)。

### 環境偵測的變更

為了安全因素，不再使用網域來偵測應用程式的環境。因為這些值很容易被偽造欺騙，繼而讓攻擊者透過請求來變更環境。你必須改為使用機器的主機名稱 (在 Mac、Linux 和 Windows 下執行 `hostname` 指令的值) 來偵測環境。

### 更簡單的日誌檔案

Laravel 目前只會產生單一的日誌檔案：`app/storage/logs/laravel.log`。然而，你還是可以在 `app/start/global.php` 檔案作設定更改它的行為。

### 移除重導向結尾的斜線

在你的 `bootstrap/start.php` 檔案中，移除對 `$app->redirectIfTrailingSlash()` 的呼叫。這個方法已經不再需要，因為這個功能現在已交由框架內的 `.htaccess` 檔案來處理。

然後，用[新的檔案](https://github.com/laravel/laravel/blob/v4.1.0/public/.htaccess)置換掉你的 Apache `.htaccess`，來處理結尾的斜線。

### 取得目前路由

目前路由現在透過 `Route::current()` 取得，而不是 `Route::getCurrentRoute()`。

### Composer 更新

一旦你完成以上的更新，你可以執行 `composer update` 功能來更新應用程式的核心檔案！如果發生 class load 錯誤，試著執行 `update` 指令並加上 `--no-scripts` 選項，樣這樣：`composer update --no-scripts`。

### 萬用字元事件監聽者

萬用字元事件監聽者不再增加事件為參數到你的處理函數。如果你需要尋找你觸發的事件你應該用 `Event::firing()`。
