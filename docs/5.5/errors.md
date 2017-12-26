# 錯誤與日誌

- [介紹](#introduction)
- [設定檔](#configuration)
    - [錯誤細節](#error-detail)
    - [日誌儲存](#log-storage)
    - [日誌的嚴重性級別](#log-severity-levels)
    - [自訂 Monolog 設定](#custom-monolog-configuration)
- [異常處理](#the-exception-handler)
    - [Report 方法](#report-method)
    - [Render 方法](#render-method)
    - [可報告與可見的例外](#renderable-exceptions)
- [HTTP 例外](#http-exceptions)
    - [自訂 HTTP 錯誤頁面](#custom-http-error-pages)
- [日誌](#logging)

<a name="introduction"></a>
## 介紹

當你開始一個新的 Laravel 專案時，都已經為你設定好錯誤與異常處理。`App\Exceptions\Handler` 類別是記錄應用程式所觸發的所有異常並將其呈現給使用者的地方。在這個文件中，我們將會深入這個類別。

關於日誌，Laravel 使用 [Monolog](https://github.com/Seldaek/monolog) 函式庫，它提供了支援各種強大日誌的處理程序。Laravel 為你設定了其中幾個處理程序，可以讓你在單一或多個日誌檔案，或將錯誤訊息寫入系統日誌之間做選擇。

<a name="configuration"></a>
## 設定

<a name="error-detail"></a>
### 錯誤細節

`config/app.php` 設定檔中的 `debug` 選項會確認實際有多少錯誤資訊顯示給使用者。預設這個選項會遵守存放在 `.env` 檔案的 `APP_DEBUG` 環境變數的值。

在本機開發的時候，你應該將 `APP_DEBUG` 環境變數設定為 `true`。在你的上線環境中，這個值應該永遠為 `false`。如果在正式上線主機上設定為 `true`，你可能會將設定檔的敏感資訊暴露給應用程式末端的使用者。

<a name="log-storage"></a>
### 日誌儲存

Laravel 提供立即可用的日誌模式，支援寫入日誌資訊到 `single` 檔案、`daily` 檔案、`syslog` 和 `errorlog`。要設定 Laravel 使用的儲存系統，你應該在 `config/app.php` 設定檔修改 `log` 選項。例如，如果你希望使用 `daily` 記錄檔而不是單個檔案，你應該將 `app` 設定檔的 `log` 值設定為 `daily`：

    'log' => 'daily'

#### 最大天數日誌檔案

使用 `daily` 日誌模式，Laravel 預設只會保留五天的日誌檔案。如果你想要調整保留檔案的天數，你可以新增 `log_max_files` 設定值到 `app` 設定檔：

    'log_max_files' => 30

<a name="log-severity-levels"></a>
### 日誌的嚴重性級別

使用 Monolog 時，日誌訊息可以有不同級別的嚴重性。預設的 Laravel 會儲存所有寫入的日誌級別。然而，在你正式環境中，你可能希望通過增加 `log_level` 選項到你的 `app.php` 設定檔來將設定應該記錄的最低嚴重性。

這個選項一旦被設定，Laravel 會記錄所有級別大於或等於指定的嚴重性。例如，預設的 `error` `日誌級別` 會記錄 **錯誤**、**嚴重**、**警告** 和 **緊急** 訊息：

    'log_level' => env('APP_LOG_LEVEL', 'error'),

> {tip} Monolog 可以識別以下嚴重程度，從最低到最嚴重： `debug`, `info`, `notice`, `warning`, `error`, `critical`, `alert`, `emergency`.

<a name="custom-monolog-configuration"></a>
### 自訂 Monolog 設定

如果你想要完全掌控 Monolog 為應用程式的設定，可以使用 `configureMonologUsing` 方法。在檔案回傳 `$app` 變數之前，你應該在你的 `bootstrap/app.php` 檔案終回呼這個方法：

    $app->configureMonologUsing(function ($monolog) {
        $monolog->pushHandler(...);
    });

    return $app;

#### 自訂頻道名稱

預設的 Monolog 實例化名稱會與當前環境匹配，像是 `production` 或 `local`。要變更這個值，請新增 `log_channel` 選項到 `app.php` 設定檔中：

    'log_channel' => env('APP_LOG_CHANNEL', 'my-app-name'),

<a name="the-exception-handler"></a>
## 異常處理

<a name="report-method"></a>
### Report 方法

`App\Exceptions\Handler` 類別會幫你處理所有例外。這個類別會有兩個方法：`report` 和 `render`。我們將詳細的討論這些方法。`report` 方法用於記錄例外或發送它們到外部服務，像是 [Bugsnag](https://bugsnag.com) 或 [Sentry](https://github.com/getsentry/sentry-laravel)。預設的 `report` 方法只是將例外傳給記錄例外的基礎類別。不過，你可以自由的記錄例外。

例如，如果你需要以不同的方式回報不同類型的例外，你可以使用 `instanceof` 類型運算子：

    /**
     * 回報或記錄一個例外。
     *
     * 這是向 Sentry，Bugsnag 等發送例外的好地方。
     *
     * @param  \Exception  $exception
     * @return void
     */
    public function report(Exception $exception)
    {
        if ($exception instanceof CustomException) {
            //
        }

        return parent::report($exception);
    }

#### `report` 輔助函式

有時你一方面要回報例外，另一方面要處理當前的請求。`report` 輔助函式可以讓你使用異常處理器的 `report` 方法快速的回報例外，而不會顯示錯誤頁面：

    public function isValid($value)
    {
        try {
            // Validate the value...
        } catch (Exception $e) {
            report($e);

            return false;
        }
    }

#### 依類型忽略例外

異常處理器的 `$dontReport` 屬性會包含一組例外類型的陣列不會被記錄。例如，404 錯誤導致的例外以及其它類型的錯誤將不會寫入你的日誌中。可以根據實際的需要來新增其他例外類型到這組陣列：

    /**
     * 不該被回報的例外類型清單。
     *
     * @var array
     */
    protected $dontReport = [
        \Illuminate\Auth\AuthenticationException::class,
        \Illuminate\Auth\Access\AuthorizationException::class,
        \Symfony\Component\HttpKernel\Exception\HttpException::class,
        \Illuminate\Database\Eloquent\ModelNotFoundException::class,
        \Illuminate\Validation\ValidationException::class,
    ];

<a name="render-method"></a>
### Render 方法

`render` 方法負責將給定的例外轉換成應該回傳給瀏覽器的 HTTP 回應。預設的例外會被傳入產生回應的基礎類別。然而，你可以自由的檢查例外類型或回傳你自訂的回應：

    /**
     * 渲染例外到 HTTP 回應。
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \Exception  $exception
     * @return \Illuminate\Http\Response
     */
    public function render($request, Exception $exception)
    {
        if ($exception instanceof CustomException) {
            return response()->view('errors.custom', [], 500);
        }

        return parent::render($request, $exception);
    }

<a name="renderable-exceptions"></a>
### 可報告與可見的例外

想要取代異常處理器用來檢查類型例外的 `report` 和 `render` 方法，你可以直接在你自訂的例外中定義 `report` 和 `render` 方法。當這些方法存在時，它們會被框架自動呼叫：

    <?php

    namespace App\Exceptions;

    use Exception;

    class RenderException extends Exception
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
         * 渲染例外到 HTTP 回應。
         *
         * @param  \Illuminate\Http\Request
         * @return \Illuminate\Http\Response
         */
        public function render($request)
        {
            return response(...);
        }
    }

<a name="http-exceptions"></a>
## HTTP 例外

有些例外會描述 HTTP 錯誤代碼來自伺服器。例如，這可能是「找不到頁面」的 404 錯誤代碼，「未授權的錯誤」的 401 錯誤代碼或者是開發者產生的 500 錯誤代碼。為了在你應用程式的任何地方產生這樣的回應，你可以使用 `abort` 輔助函式：

    abort(404);

`abort` 輔助函式會馬上發出異常處理器即將渲染的例外。或者，你可以提供回應的文字內容：

    abort(403, 'Unauthorized action.');

<a name="custom-http-error-pages"></a>
### 自訂 HTTP 錯誤頁面

Laravel 很容易為各種的 HTTP 狀態碼設計所要顯示的自訂錯誤頁面。例如，如果你希望自訂 HTTP 404 錯誤代碼頁面，而去建立 `resources/views/errors/404.blade.php`。這個檔案會為應用程式產生所有的 404 錯誤代碼而服務。在這個目錄中的視圖應該命名與 HTTP 狀態碼一致。透過 `abort` 函式發出的 `HttpException` 實例會作為 `$exception` 變數傳入視圖：

    <h2>{% raw %} {{ $exception->getMessage() }} {% endraw %}</h2>

<a name="logging"></a>
## 日誌

Laravel 在強大的 [Monolog](https://github.com/seldaek/monolog) 函式庫上提供了一個簡單的抽象層。預設的 Laravel 已設定好在 `storage/logs` 目錄中建立日誌檔案。你可以使用 `Log` [facade](/laravel_tw/docs/5.5/facades) 撰寫資訊到日誌中：

    <?php

    namespace App\Http\Controllers;

    use App\User;
    use Illuminate\Support\Facades\Log;
    use App\Http\Controllers\Controller;

    class UserController extends Controller
    {
        /**
         * 顯示給定使用者的個人資料。
         *
         * @param  int  $id
         * @return Response
         */
        public function showProfile($id)
        {
            Log::info('Showing user profile for user: '.$id);

            return view('user.profile', ['user' => User::findOrFail($id)]);
        }
    }

日誌提供了符合 [RFC 5424](https://tools.ietf.org/html/rfc5424) 定義的八種日誌級別：**emergency**、**alert**、**critical**、**error**、**warning**、**notice**、**info** 和 **debug**。

    Log::emergency($message);
    Log::alert($message);
    Log::critical($message);
    Log::error($message);
    Log::warning($message);
    Log::notice($message);
    Log::info($message);
    Log::debug($message);

#### Context 資訊

Context 資料的陣列也可以被傳入 `Log` 方法。這個 Context 資料會被格式化並顯示日誌訊息：

    Log::info('User failed to login.', ['id' => $user->id]);

#### 存取底層的 Monolog 實例

Monolog 有多種額外的處理器可被用於記錄上。如果有需要，你可以存取被 Laravel 使用的底層 Monolog 實例：

    $monolog = Log::getMonolog();
