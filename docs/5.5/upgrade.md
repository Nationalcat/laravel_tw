---
layout: post
title: upgrade
tag: 5.5
---
# 升級指南

- [從 5.4 升級到 5.5.0](#upgrade-5.5.0)

<a name="upgrade-5.5.0"></a>
## 從 5.4 升級到 5.5.0

#### 預估升級時間：1 小時

> {note} 我們試圖記錄每個重大的改變。由於有些重大的改變在框架最隱密的地方，但事實上只有一小部分可能會影響到你的應用程式。

### PHP

Laravel 5.5 要求 PHP 7.0.0 或更高的版本。

### 升級依賴項目

在 `composer.json` 檔案升級你的 `laravel/framework` 依賴項目到 `5.5.*`。此外，你應該升級 `phpunit/phpunit` 依賴項目到 `~6.0`。接著，新增 `filp/whoops` 套件且版本為 `~2.0`，並放置於 `composer.json` 中的 `require-dev` 選項。最後，在 `composer.json`  檔案中的 `scripts` 選項，新增 `package:discover` 指令到 `post-autoload-dump` 事件中:

    "scripts": {
        ...
        "post-autoload-dump": [
            "Illuminate\\Foundation\\ComposerScripts::postAutoloadDump",
            "@php artisan package:discover"
        ],
    }

當然，別忘了檢查應用程式使用的第三方套件，並檢查是否有無支援 Laravel 5.5。

#### Laravel Installer

> {tip} 如果你平常都是透過 `laravel new` 指令來安裝 Laravel，你應該使用 `composer global update` 指令來更新 Laravel 全域安裝版本。

#### Laravel Dusk

Laravel Dusk `2.0.0` 已經發布啦！並提供 Laravel 5.5 和 headless Chrome 測試的相容性。

#### Pusher

Pusher 事件廣播驅動的 Pusher SDK 現在要求 `~3.0` 版本。

#### Swift Mailer

Laravel 5.5 的 Swift Mailer 現在要求 `~6.0` 版本。

### Artisan

#### 自動載入指令

Artisan 能在 Laravel 5.5 中自動的發現指令，這樣你就不需要手動將它們註冊到核心中。若要使用這個新功能，你應該底下的程式碼新增到 `App\Console\Kernel` 類別的 `commands`：

    $this->load(__DIR__.'/Commands');

#### `fire` 方法

在 Artisan 指令中上的任何 `fire` 方法，現在被重新命名為 `handle`。

#### `optimize` 指令

最近 PHP 的 op-code 快取有進行了改善，之後就不需要使用 Artisan 的 optimize 指令。你應該從部署的腳本上移除任何對這個指令的任何引用，因為它會在 Laravel 之後的版本被移除。

### 授權

> {note} 當你從 Laravel 5.4 升級到 5.5 時，所有的 `remember_me` cookie 會因此而失效並且將使用者登出。

#### `authorizeResource` 控制器方法

將駝峰式命名的模型傳入 `authorizeResource` 方法時，會為了與資源控制器的行為匹配，把路由改成使用蛇狀命名的方式。

#### `before` Policy 方法

如果類別不包含與該方法名稱匹配的檢查名稱，就不會呼叫 Policy 類別的 `before` 方法。

### 快取

#### 資料庫驅動

如果你使用資料庫快取驅動，你應該在部署升級 Laravel 5.5 的第一時間使用 `php artisan cache:clear` 指令。

### Eloquent

#### `belongsToMany` 方法

如果你在 Eloquent 模型上複寫 `belongsToMany` 方法，你應該更新方法的簽署來對應新增的參數：

    /**
     * 定義一個多對多關聯。
     *
     * @param  string  $related
     * @param  string  $table
     * @param  string  $foreignPivotKey
     * @param  string  $relatedPivotKey
     * @param  string  $parentKey
     * @param  string  $relatedKey
     * @param  string  $relation
     * @return \Illuminate\Database\Eloquent\Relations\BelongsToMany
     */
    public function belongsToMany($related, $table = null, $foreignPivotKey = null,
                                  $relatedPivotKey = null, $parentKey = null,
                                  $relatedKey = null, $relation = null)
    {
        //
    }

#### BelongsToMany 的 `getQualifiedRelatedKeyName` 方法

`getQualifiedRelatedKeyName` 方法被重新命名為 `getQualifiedRelatedPivotKeyName`。

#### BelongsToMany 的 `getQualifiedForeignKeyName` 方法

`getQualifiedForeignKeyName` 方法被重新命名為 `getQualifiedForeignPivotKeyName`。

#### 模型的 `is` 方法

如果你覆寫了 Eloquent 模型的 `is` 方法，你應該從該方法中移除`模型`的型別注入。這會讓 `is` 方法可以接受 `null` 參數：

    /**
     * 確認兩個模型是否有相同的 ID 並屬於同一張資料表。
     *
     * @param  \Illuminate\Database\Eloquent\Model|null  $model
     * @return bool
     */
    public function is($model)
    {
        //
    }

#### 模型的 `$events` 屬性

在你的模型上的 `$events` 屬性應該被重新命名為 `$dispatchesEvents`。會做這項的變更，是因為大量的使用者會需要定義一個 `events` 關聯，導致新舊屬性名稱發生衝突。

#### 中介層的 `$parent` 屬性

The protected `$parent` property on the在 `Illuminate\Database\Eloquent\Relations\Pivot` 類別上的 $parent 屬性被重新命名為 `$pivotParent`。

#### 關聯的 `create` 方法

`BelongsToMany`、`HasOneOrMany` 和 `MorphOneOrMany` 類別的 `create` 方法以修改成提供 `$attributes` 參數一個空陣列。如果你正要覆寫這些方法，你應該更新你的簽署來與新的定義匹配：

    public function create(array $attributes = [])
    {
        //
    }

#### 軟刪除模型

當你在刪除一個「被軟刪除」的模型時，在模型上的 `exists` 屬性會維持在 `true`。

#### `withCount` 欄位格式化

在使用別名時，`withCount` 方法將不在自動附加 `_count` 到產生的欄位名稱上。例如，在 Laravel 5.4 中，以下查詢會導致 `bar_count` 欄位新增到查詢中：

    $users = User::withCount('foo as bar')->get();

然而在 Laravel 5.5，預設不在使用別名。如果你想要附加 `_count` 到結果的欄位，你必須在定義別名時指定後綴給它：

    $users = User::withCount('foo as bar_count')->get();

#### 模型方法與屬性名稱

為了預防使用陣列存取時，存取到模型的私有屬性。現在模型方法將不能再使用屬性相同的名稱。這麼做會導致在透過陣列存取（`$user['name']`）或 `data_get` 輔助函式的時候拋出例外。

### 例外格式

在 Laravel 5.5 包括驗證的例外的所有例外，都會經由例外處理器而轉換成 HTTP 回應。此外，JSON 驗證錯誤的預設格式也被更動。新的格式符合以下的慣例：

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

然而，如果你想要維持 Laravel 5.4 JSON 錯誤格式，你可以新增以下方法到 `App\Exceptions\Handler` 類別：

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

#### JSON 認證嘗試

這個變更還會影響通過JSON進行的驗證嘗試的驗證錯誤格式。在 Laravel 5.5 的 JSON 驗證失敗會依照上述新的格式慣例回傳錯誤訊息。

#### 表單請求的注意事項

如果你正在自訂一個表單請求的回應格式，則現在應該覆寫該表單請求的 `failedValidation` 方法，並拋出一個包含你的自訂回應的 `HttpResponseException` 實例：

    use Illuminate\Http\Exceptions\HttpResponseException;

    /**
     * 處理失敗的驗證嘗試。
     *
     * @param  \Illuminate\Contracts\Validation\Validator  $validator
     * @return void
     *
     * @throws \Illuminate\Validation\ValidationException
     */
    protected function failedValidation(Validator $validator)
    {
        throw new HttpResponseException(response()->json(..., 422));
    }

### 檔案系統

#### `files` 方法

`Illuminate\Filesystem\Filesystem` 類別的 `files` 方法已經改變了它的簽署，並新增 `$hidden` 參數，現在會回傳一個 `SplFileInfo` 物件陣列，類似於 `allFiles` 方法。在之前，`files` 方法只會回傳一個字串路徑名稱的陣列。新的簽署如下：

    public function files($directory, $hidden = false)

### 信箱

#### 未使用的參數

從 `Illuminate\Contracts\Mail\MailQueue` 的 Contract 的 `queue` 和 `later` 方法中移除未使用的 `$data` 和 `$callback` 參數：

    /**
     * 隊列一個正要發送新電子郵件的訊息。
     *
     * @param  string|array|MailableContract  $view
     * @param  string  $queue
     * @return mixed
     */
    public function queue($view, $queue = null);

    /**
     * 在幾秒後隊列一個正要發送新電子郵件的訊息。
     *
     * @param  \DateTimeInterface|\DateInterval|int  $delay
     * @param  string|array|MailableContract  $view
     * @param  string  $queue
     * @return mixed
     */
    public function later($delay, $view, $queue = null);

### 隊列

#### `dispatch` 輔助函式

如果你想指派一個可以立即執行的任務，並從 `handle` 方法中回傳一個值，你應該使用 `dispatch_now` 或 `Bus::dispatchNow` 方法來指派任務：

    use Illuminate\Support\Facades\Bus;

    $value = dispatch_now(new Job);

    $value = Bus::dispatchNow(new Job);

### 請求

#### `all` 方法

如果你正要覆寫 `Illuminate\Http\Request` 類別的 `all` 方法，你應該更新方法的簽署來對應新的 `$keys` 參數：

    /**
     * 取得請求的所有輸入與檔案。
     *
     * @param  array|mixed  $keys
     * @return array
     */
    public function all($keys = null)
    {
        //
    }

#### `has` 方法

就算輸入的值是空字串或 `null`，`$request->has` 方法現在會回傳 `true`。現在已新增新的 `$request->filled` 方法，並提供之前的 `has` 方法的用法。

#### `intersect` 方法

`intersect` 已被移除了。你可以在呼叫 `$request->only` 時使用 `array_filter` 方法來重置這個行為：

    return array_filter($request->only('foo'));

#### `only` 方法

`only` 方法目前只會回傳實際存在於請求負載中的屬性。如果你想保留 `only` 方法的舊用法，則可以使用 `all` 方法。

    return $request->all('foo');

#### `request()` 輔助函式

`request`輔助函式將不再取得嵌入的鍵。如果你有需要，可以使用請求的 `input` 方法來實現這個行為：

    return request()->input('filters.date');

### 測試

#### 認證的斷言

為了與框架的其他部分維持更好的一致性，有些認證斷言已被重新命名：

<div class="content-list" markdown="1">
- `seeIsAuthenticated` 被重新命名為 `assertAuthenticated`.
- `dontSeeIsAuthenticated` 被重新命名為 `assertGuest`.
- `seeIsAuthenticatedAs` 被重新命名為 `assertAuthenticatedAs`.
- `seeCredentials` 被重新命名為 `assertCredentials`.
- `dontSeeCredentials` 被重新命名為 `assertInvalidCredentials`.
</div>

#### 假信箱

如果你正在使用 `Mail` Fake 來確認在請求期間信件是否被**隊列**，你現在應該使用 `Mail::assertQueued `而不是 `Mail::assertSent`。這可以讓你明確地斷言信件隊列正在等待背景發送，而不是在請求階段本身就發送。

#### Tinker

Laravel Tinker 現在支援在引用應用程式類別時省略命名空間。這個功能需要一個優化的 Composer 類別映射，所以你應該在 `composer.json` 檔案的 `config` 部分新增 `optimize-autoloader` 指令：

    "config": {
        ...
        "optimize-autoloader": true
    }

### 語系

#### `LoaderInterface`

`Illuminate\Translation\LoaderInterface` 介面現在被移到 `Illuminate\Contracts\Translation\Loader`.

### 驗證

#### 驗證方法

現在所有驗證器的驗證方法都是 `public`，而不是 `protected`。

### 視圖

#### 動態的「With」變數名稱

當可以讓動態的 `__call` 方法與視圖共用變數時，這些變數會自動使用「駝峰式命名」。例如，給定以下內容：

    return view('pool')->withMaximumVotes(100);

可以在模板中存取 `maximumVotes` 變數，就像是：

    {% raw %} {{ $maximumVotes }} {% endraw %}

#### `@php` Blade 指令

`@php` blade 指令不再接受行內標籤。反而是使用完整形式的指令：

    @php
        $teamMember = true;
    @endphp

### 其他

我們也鼓勵你查看 `laravel/laravel` [GitHub 儲存庫](https://github.com/laravel/laravel)中的任何異動。儘管許多更改並不是必要的，但你可能希望保持這些文件與你的應用程序同步。其中一些更改將在本升級指南中介紹，但其他更改（例如更改設定檔案或註釋）將不會被介紹。你可以使用 [GitHub 比較工具](https://github.com/laravel/laravel/compare/5.4...5.5)來輕易的檢查更動的內容，並選擇哪些更新對你比較重要。
