---
layout: post
title: responses
tag: 5.5
---
# HTTP 回應

- [建立回應](#creating-responses)
    - [附加 Header 到回應](#attaching-headers-to-responses)
    - [附加 Cookie 到回應](#attaching-cookies-to-responses)
    - [Cookie 與加密](#cookies-and-encryption)
- [重導](#redirects)
    - [重導到已命名的路由](#redirecting-named-routes)
    - [重導到控制器行為](#redirecting-controller-actions)
    - [重導並加上快閃 Session 資料](#redirecting-with-flashed-session-data)
- [其他回應類型](#other-response-types)
    - [視圖回應](#view-responses)
    - [JSON 回應](#json-responses)
    - [檔案下載](#file-downloads)
    - [檔案回應](#file-responses)
- [回應巨集](#response-macros)

<a name="creating-responses"></a>
## 建立回應

#### 字串與陣列

所有路由與控制器應該回傳一個回應到使用者的瀏覽器。Laravel 提供幾個不同的方式來回傳回應。最基本的回應是只從路由和控制器中回傳一組字串。Laravel 會自動將字串轉換成完整的 HTTP 回應：

    Route::get('/', function () {
        return 'Hello World';
    });

除了從路由和控制器中回傳字串，你也可以回傳陣列。Laravel 會自動將陣列轉換成 JSON 回應：

    Route::get('/', function () {
        return [1, 2, 3];
    });

> {tip} 你知道你也能從路由和控制器中回傳 [Eloquent 集合](/laravel_tw/docs/5.5/eloquent-collections)嗎？它們會自動被轉換成 JSON。快來試試看！

#### 回應物件

通常你不會只從路由行為中回傳簡單的字串或陣列，反而是回傳完整的 `Illuminate\Http\Response` 實例或[視圖](/laravel_tw/docs/5.5/views)。

回傳完整的 `Response` 實例可以讓你自訂 HTTP 狀態碼和 header。`Response` 實例繼承自 `Symfony\Component\HttpFoundation\Response` 類別，這會附帶各種方法來提供建構 HTTP 回應時使用：

    Route::get('home', function () {
        return response('Hello World', 200)
                      ->header('Content-Type', 'text/plain');
    });

<a name="attaching-headers-to-responses"></a>
#### 回應附加標頭

請記得，大多數的方法是可以被鏈結的，這可以讓你優雅的建構回應實例。例如，在發送回應給使用者前，你可以使用 `header` 方法來新增一系列的 header 到回應中：

    return response($content)
                ->header('Content-Type', $type)
                ->header('X-Header-One', 'Header Value')
                ->header('X-Header-Two', 'Header Value');

或是你使用 `withHeaders` 方法來指定一組 header 陣列，並新增到回應中：

    return response($content)
                ->withHeaders([
                    'Content-Type' => $type,
                    'X-Header-One' => 'Header Value',
                    'X-Header-Two' => 'Header Value',
                ]);

<a name="attaching-cookies-to-responses"></a>
#### 回應附加 Cookie

回應實例上的 `cookie` 方法可以讓你輕易的附加 Cookie 到回應上。例如，你可以使用 `cookie` 方法來產生一組 Cookie，並優雅地將它附加到回應實例上，就像是：

    return response($content)
                    ->header('Content-Type', $type)
                    ->cookie('name', 'value', $minutes);

`cookie` 方法也接受一些不常使用的參數。通常這些參數會和 PHP 原生的 [setcookie](https://secure.php.net/manual/en/function.setcookie.php) 方法的參數具有相同的意義和目的：

    ->cookie($name, $value, $minutes, $path, $domain, $secure, $httpOnly)

或者，你能從應用程式中使用 `Cookie` Facade 來「隊列」與附加一組 Cookies 到即將輸出的回應上。`queue` 方法接受一個 `Cookie` 實例或建立 `Cookie` 實例所需的參數。在發送回應到瀏覽器之前，這些 cookie 會被附加到即將輸出的回應上：

    Cookie::queue(Cookie::make('name', 'value', $minutes));

    Cookie::queue('name', 'value', $minutes);

<a name="cookies-and-encryption"></a>
#### Cookie 與加密

預設所有 Laravel 產生的 Cookie 會被加密與簽署，使它無法在客戶端被串改或讀取。如果你想要停用應用程式所產生部分的 Cookie 加密，你可以使用 `App\Http\Middleware\EncryptCookies` 中介層的 `$except` 屬性，它位在 `app/Http/Middleware` 目錄中：

    /**
     * 被列出名稱的 Cookie，都不會被加密。
     *
     * @var array
     */
    protected $except = [
        'cookie_name',
    ];

<a name="redirects"></a>
## 重導

重導回應是 `Illuminate\Http\RedirectResponse` 類別的實例，並包含需要被重導到使用者另一個 URL 所需要的正確 header。這有幾種方式來產生 `RedirectResponse` 實例。最簡單的方法是使用全域的 `redirect` 輔助函式：

    Route::get('dashboard', function () {
        return redirect('home/dashboard');
    });

有時你可能希望將使用者重導到他們先前使用的頁面，像是在送出的表單是無效的時候。你可以使用全域的 `back` 輔助函式。由於這個功能會用到 [Session](/laravel_tw/docs/5.5/session)，請確認使用 `back` 函式的路由有使用 `web` 中介層群組或套用到所有 Session 中介層：

    Route::post('user/profile', function () {
        // 驗證請求...

        return back()->withInput();
    });

<a name="redirecting-named-routes"></a>
### 重導到已命名的路由

當你不使用參數的呼叫 `redirect` 輔助函式，就會回傳 `Illuminate\Routing\Redirector` 實例，可以讓你呼叫任何在 `Redirector` 實例上的方法。例如，要產生一個 `RedirectResponse` 到已命名的路由，你可以使用 `route` 方法：

    return redirect()->route('login');

如果你的路由帶有其他參數，你可以將它們作為第二個參數傳入 `route` 方法：

    // 以下路由會對應到這組 URI： profile/{id}

    return redirect()->route('profile', ['id' => 1]);

#### 透過 Eloquent 模型填入參數

如果你正要重導到需要從 Eloquent 模型填入「ID」參數的路由，你可以只傳入模型本身。該 ID 會被自動取出：

    // 以下路由會對應到這組 URI： profile/{id}

    return redirect()->route('profile', [$user]);

如果你想要自定義位在路由參數中的值，你應該在 Eloquent 模型上覆寫 `getRouteKey` 方法：

    /**
     * 取得模型的路由鍵的值。
     *
     * @return mixed
     */
    public function getRouteKey()
    {
        return $this->slug;
    }

<a name="redirecting-controller-actions"></a>
### 重導到控制行為

你也可以產生重導到[控制器行為](/laravel_tw/docs/5.5/controllers)。可以將控制器和行為名稱傳入 `action` 方法來做到。請記得，你不需要指定完整的命名空間到控制器，因為 Laravel 的 `RouteServiceProvider` 已預設為基本控制器的命名空間：

    return redirect()->action('HomeController@index');

如果你的控制器路由需要參數，你可以將它們作為第二個參數來傳入 `action` 方法：

    return redirect()->action(
        'UserController@profile', ['id' => 1]
    );

<a name="redirecting-with-flashed-session-data"></a>
### 重導並加上快閃 Session 資料

通常重導到新的 URL 並同時加上[快閃資料到 Session](/laravel_tw/docs/5.5/session#flash-data) 幾乎是在同一時間完成的。通常是用在成功執行一個行為後，將成功的訊息快閃到 Session。為了方便，你可以建立一個 `RedirectResponse` 實例，並使用一個優雅的方法鏈結將資料快閃到 Session：

    Route::post('user/profile', function () {
        // 更新使用者的個人資料...

        return redirect('dashboard')->with('status', 'Profile updated!');
    });

使用者被重導後，你可以從 [Session](/laravel_tw/docs/5.5/session) 中顯示被快閃的訊息。例如，使用 [Blade 語法](/laravel_tw/docs/5.5/blade)：

    @if (session('status'))
        <div class="alert alert-success">
            {% raw %} {{ session('status') }} {% endraw %}
        </div>
    @endif

<a name="other-response-types"></a>
## 其他回應類型

`response` 輔助函式可以被用於產生其他回應實例的類型。當 `response` 輔助函式不帶參數的方式被呼叫時，會回傳 `Illuminate\Contracts\Routing\ResponseFactory` [Contract](/laravel_tw/docs/5.5/contracts) 實作。這個 Contract 提供幾個好用的方法來產生回應。

<a name="view-responses"></a>
### 視圖回應

如果你需要掌控回應的狀態與標頭，同時也需要回傳一個[視圖](/laravel_tw/docs/5.5/views)作為回應的內容，你應該使用 `view` 方法：

    return response()
                ->view('hello', $data, 200)
                ->header('Content-Type', $type);

當然，如果你不需要傳入一個自訂的 HTTP 狀態碼或自訂的 header，就直接使用 `view` 輔助函式。

<a name="json-responses"></a>
### JSON 回應

`json` 方法會自動設定 `Content-Type` header 到 `application/json`，以及使用 PHP 的 `json_encode` 函式將陣列轉換成 JSON：

    return response()->json([
        'name' => 'Abigail',
        'state' => 'CA'
    ]);

如果你想要建立一個 JSONP 回應，你可以使用 `json` 方法並加上 `withCallback` 方法：

    return response()
                ->json(['name' => 'Abigail', 'state' => 'CA'])
                ->withCallback($request->input('callback'));

<a name="file-downloads"></a>
### 檔案下載

`download` 方法可被用於產生一個會強制使用者的瀏覽器去下載給定路徑的檔案的回應。`download` 方法接受一個檔案名稱作為該方法的第二個參數，這會確認下載該檔案的使用者所看到的檔案名稱。最後，你可以將一組 HTTP header 的陣列作為第三個參數傳入到該方法：

    return response()->download($pathToFile);

    return response()->download($pathToFile, $name, $headers);

    return response()->download($pathToFile)->deleteFileAfterSend(true);

> {note} 管理檔案下載的套件 Symfony HttpFoundation，要求被下載的檔案名稱必須為 ASCII。

<a name="file-responses"></a>
### 檔案回應

`file` 方法可直接被用於顯示一個檔案在使用者的瀏覽器，像是一個圖像或 PDF，而不是去下載。這個方法接受檔案的路徑作為它的第一個參數，並將一組標頭的陣列作為它的第二個參數：

    return response()->file($pathToFile);

    return response()->file($pathToFile, $headers);

<a name="response-macros"></a>
## 回應巨集

如果你想要定義一個自訂一個可以在各種路由和控制器上重複使用的回應，你可以在 `Response` facade 上使用 `macro` 方法。例如，從一個[服務提供者的](/laravel_tw/docs/5.5/providers) `boot` 方法開始示範：

    <?php

    namespace App\Providers;

    use Illuminate\Support\ServiceProvider;
    use Illuminate\Support\Facades\Response;

    class ResponseMacroServiceProvider extends ServiceProvider
    {
        /**
         * 註冊該路由的回應巨集。
         *
         * @return void
         */
        public function boot()
        {
            Response::macro('caps', function ($value) {
                return Response::make(strtoupper($value));
            });
        }
    }

`macro` 函式接受一個名稱作為它的第一個參數，並將閉包作為它的第二個參數。從一個 `ResponseFactory` 實作或 `response` 輔助函式中呼叫巨集名稱時，該巨集的閉包會被執行：

    return response()->caps('foo');
