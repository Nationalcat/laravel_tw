---
layout: post
title: requests
tag: 5.5
---
# HTTP 請求

- [取得請求](#accessing-the-request)
    - [請求路徑與方法](#request-path-and-method)
    - [PSR-7 請求](#psr7-requests)
- [輸入的修飾與標準化](#input-trimming-and-normalization)
- [取得輸入](#retrieving-input)
    - [舊輸入](#old-input)
    - [Cookies](#cookies)
- [檔案](#files)
    - [取得上傳檔案](#retrieving-uploaded-files)
    - [儲存上傳檔案](#storing-uploaded-files)
- [設定可信任的代理](#configuring-trusted-proxies)

<a name="accessing-the-request"></a>
## 取得請求

要透過依賴注入才能獲得目前 HTTP 請求的實例，你應該在你的控制器方法上型別注入 `Illuminate\Http\Request` 類別。傳入的請求會自動由[服務容器](/laravel_tw/docs/5.5/container)注入：

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;

    class UserController extends Controller
    {
        /**
         * 儲存一位新使用者。
         *
         * @param  Request  $request
         * @return Response
         */
        public function store(Request $request)
        {
            $name = $request->input('name');

            //
        }
    }

#### 依賴注入與路由參數

如果你的控制器也預期從路由參數中輸入資料，你應該將路由參數置於其他依賴之後。例如，如果你的路由定義像是：

    Route::put('user/{id}', 'UserController@update');

你仍然可以注入 `Illuminate\Http\Request` 和存取你的路由參數 `id`，只要定義你的控制器方法與如下：

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;

    class UserController extends Controller
    {
        /**
         * 更新特定使用者。
         *
         * @param  Request  $request
         * @param  string  $id
         * @return Response
         */
        public function update(Request $request, $id)
        {
            //
        }
    }

#### 透過路由閉包存取請求

你也可以在路由閉包上型別提示 `Illuminate\Http\Request` 類別。該服務容器會在執行時自動將傳入的請求注入到閉包中：

    use Illuminate\Http\Request;

    Route::get('/', function (Request $request) {
        //
    });

<a name="request-path-and-method"></a>
### 請求路徑與方法

`Illuminate\Http\Request` 實例為檢查應用程式的 HTTP 請求而提供各種方法，並繼承 `Symfony\Component\HttpFoundation\Request` 類別。我們將在下面討論一些重要的方法。

#### 取得請求路徑

`path` 方法會回傳請求的路徑資訊。所以，如果傳來的請求的目的地是 `http://domain.com/foo/bar`，`path` 方法會回傳 `foo/bar`：

    $uri = $request->path();

`is` 方法可以讓你驗證傳入的請求路徑是否與給定的模式相符合。你可以在使用這方法時，使用 `*` 字元作為萬用字元。

    if ($request->is('admin/*')) {
        //
    }

#### 取得請求 URL

要取得傳入請求的完整的 URL，你可以使用 `url` 或 `fullUrl` 方法。`url` 方法會回傳不含查詢字串的 URL，而 `fullUrl` 方法則會包含查詢字串：

    // Without Query String...
    $url = $request->url();

    // With Query String...
    $url = $request->fullUrl();

#### 取得請求方法

`method` 方法會回傳當次請求的 HTTP 動詞。你可以使用 `isMethod` 方法來驗證該 HTTP 動詞是否與給定字串符合：

    $method = $request->method();

    if ($request->isMethod('post')) {
        //
    }

<a name="psr7-requests"></a>
### PSR-7 請求

[PSR-7 標準](http://www.php-fig.org/psr/psr-7/)規範了包含請求與回應的 HTTP 訊息介面。如果你想要得到一個 PSR-7 請求實例，而不是 Laravel 請求，你首先會需要安裝一些函式庫。Laravel 使用 *Symfony HTTP 訊息橋接*元件來將原生 Laravel 請求和回應轉換成 PSR-7 相容的實作：

    composer require symfony/psr-http-message-bridge
    composer require zendframework/zend-diactoros

一旦你安裝好這些函式庫，你可以透過型別提示請求介面在你的路由閉包或是控制器方法上取得 PSR-7 請求：

    use Psr\Http\Message\ServerRequestInterface;

    Route::get('/', function (ServerRequestInterface $request) {
        //
    });

> {tip} 如果你從路由或控制器中回傳一個 PSR-7 回應實例，它會自動的被轉換回 Laravel 回應實例，並顯示於框架中。

<a name="input-trimming-and-normalization"></a>
## 輸入的修飾與標準化

預設的 Laravel 在應用程式的一堆全域中介層中引入了 `TrimStrings` 和 `ConvertEmptyStringsToNull` 中介層。這些中介層被列在 `App\Http\Kernel` 類別中。這些中介層會自動的修飾所有在請求上傳入的字串，並將所有空字串轉換成 `null`。這可以讓你不用擔心你的路由和控制器需要轉換標準化的問題。

如果你想要停用這個行為，你可以從 `App\Http\Kernel` 類別的 `$middleware` 屬性中移除這兩個中介層。

<a name="retrieving-input"></a>
## 取得輸入

#### 取得所有輸入資料

你也可以使用 `all`方法來取得所有輸入資料，並作為一組`陣列`：

    $input = $request->all();

#### 取得一個輸入值

使用一些方法，你可以從 `Illuminate\Http\Request` 實例中存取所有使用者輸入，而不用擔心該請求是使用了哪一個動詞。不論是哪個 HTTP 動詞，`input` 方法可以被用在取得使用者的輸入：

    $name = $request->input('name');

你可以傳入預設值作為 `input` 方法的第二個參數。這個值會因為請求上沒有該輸入值，而回傳預設值：

    $name = $request->input('name', 'Sally');

當處理包含陣列輸入的表單時，請使用「點」符號來存取陣列：

    $name = $request->input('products.0.name');

    $names = $request->input('products.*.name');

#### 從查詢字串中取得輸入

使用 `input` 方法從整個請求資料（包含查詢字串）中取得值，`query` 方法會只從查詢字串中取得值：

    $name = $request->query('name');

如果請求查詢的字串不存在，這個方法會回傳第二個參數所設定的預設值：

    $name = $request->query('name', 'Helen');

你可以不使用任何參數來呼叫 `query` 方法，以便將所有查詢字串值作為一組關聯陣列：

    $query = $request->query();

#### 透過動態屬性來取得輸入

你也可以在 `Illuminate\Http\Request` 實例上使用動態屬性來存取使用者輸入。例如，如果一個應用程式表單有一個 `name` 輸入欄位，你可以像這樣存取該欄位的值：

    $name = $request->name;

當你在使用動態屬性時，Laravel 將首先尋找請求資料中的參數值。如果不存在的話，Laravel 將會搜尋在路由參數中的輸入段落。

#### 取得 JSON 輸入值

當你在發送 JSON 請求到你應用程式時，只要請求的 `Content-Type` header 正確的設為 `application/json`，你可以透過 `input` 方法存取 JSON 資料。你甚至可以使用「點」語法來深入 JSON 陣列：

    $name = $request->input('user.name');

#### 取得一部分的輸入資料

如果你需要取得輸入資料的一部份，你可以使用 `only` 和 `except` 方法。這兩種方法都接受一組`陣列`或參數動態清單：

    $input = $request->only(['username', 'password']);

    $input = $request->only('username', 'password');

    $input = $request->except(['credit_card']);

    $input = $request->except('credit_card');

> {tip} `only` 方法會回傳所有請求的鍵與值。然而，它不會回傳在請求上不存在的鍵與值。

#### 確認輸入的值是否存在

如果想確認該值是否存在於請求上，你可以使用 `has` 方法。如果該值確定存在於請求上，則 `has` 方法會回傳 `true`：

    if ($request->has('name')) {
        //
    }

當給定一個陣列時，`has` 方法會確認所有指定的值是否存在：

    if ($request->has(['name', 'email'])) {
        //
    }

如果你想要確認一個值是否存在於請求上，並且還不是空值，你可以使用 `filled` 方法：

    if ($request->filled('name')) {
        //
    }

<a name="old-input"></a>
### 舊輸入

Laravel 可以讓你在下一個請求期間保留輸入。這個功能對於驗證失敗後重填表單欄位是相當好用的。然而如果你使用 Laravel 內建的[驗證功能](/laravel_tw/docs/5.5/validation)，那你可能就不太需要手動使用這些功能，因為 Laravel 內建的驗證機制已經自動幫你處理好了。

#### 將輸入資料快閃至 Session

`Illuminate\Http\Request` 類別上的 `flash` 方法會將當前輸入快閃到 [Session](/laravel_tw/docs/5.5/session)，這樣就可以讓使用者在下一個請求期間可用這些資料：

    $request->flash();

你也可以使用 `flashOnly` 和 `flashExcept` 方法來快閃一部分的請求資料到 Session。這些方法有助於將密碼或其他敏感資訊保留在 Session 之外：

    $request->flashOnly(['username', 'email']);

    $request->flashExcept('password');

#### 快閃輸入後重導

由於你會經常想要快閃輸入的資料到 Session，並重導到前一個頁面，你可以使用 `withInput` 方法來輕易的鏈結輸入並快閃到要重導的位置：

    return redirect('form')->withInput();

    return redirect('form')->withInput(
        $request->except('password')
    );

#### 取得舊輸入資料

要從前一個請求中取得快閃的輸入資料，請在 `Request` 實例上使用 `old` 方法。`old` 方法會從 [Session](/laravel_tw/docs/5.5/session) 中將取出之前的閃存資料：

    $username = $request->old('username');

Laravel 也提供一個全域的 `old` 輔助函示。如果你在 [Blade 模板](/laravel_tw/docs/5.5/blade)中顯示就資料，使用 `old` 輔助函式會比較方便。如果舊輸入資料不存在給定的內容，就會回傳 `null`：

    <input type="text" name="username" value="{% raw %} {{ old('username') }} {% endraw %}">

<a name="cookies"></a>
### Cookies

#### 從請求中取得 Cookies

所有由 Laravel 框架所建立的 Cookie 都會被加密與使用認證碼簽署，這表示如果使用使用者竄改這些 Cookie，那麼這些 Cookie 就會無效。要從請求中取得 Cookie 值，請在 `Illuminate\Http\Request` 實例上使用 `cookie` 方法：

    $value = $request->cookie('name');

或者，你也可以使用 `Cookie` Facade 來存取 cookie 值：

    $value = Cookie::get('name');

#### 將 Cookies 附加到回應

你可以使用 `cookie` 方法將 Cookie 附加到要傳出的 `Illuminate\Http\Response` 實例。你應該將名稱、值和 Cookie 的有效使用時間傳入這個方法：

    return response('Hello World')->cookie(
        'name', 'value', $minutes
    );

`cookie` 方法也接受一些不太常使用的參數。一般而言，這些參數與 PHP 的 [setcookie](https://secure.php.net/manual/en/function.setcookie.php) 方法的參數有相同意義與目的：

    return response('Hello World')->cookie(
        'name', 'value', $minutes, $path, $domain, $secure, $httpOnly
    );

或者你能從應用程式中使用 `Cookie` facade 來「隊列」Cookie 來附加到回應中。`queue` 方法接受一個 `Cookie` 實例或建立 `Cookie` 實例所需的參數。這些 Cookie 會在送到瀏覽器之前被附加到回應上：

    Cookie::queue(Cookie::make('name', 'value', $minutes));

    Cookie::queue('name', 'value', $minutes);

#### 產生 Cookie 實例

如果你想要產生一個 `Symfony\Component\HttpFoundation\Cookie` 實例，可以晚點在給予回應實例，你可以使用全域的 `cookie` 輔助函式。除非將該 Cookie 附加到回應，不然這個 Cookie 不會被送回客戶端：

    $cookie = cookie('name', 'value', $minutes);

    return response('Hello World')->cookie($cookie);

<a name="files"></a>
## 檔案

<a name="retrieving-uploaded-files"></a>
### 取得上傳檔案

你可以從 `Illuminate\Http\Request` 實例中使用 `file` 方法或使用動態屬性來取得上傳的檔案。`file` 方法回傳一個 `Illuminate\Http\UploadedFile` 類別的實例，它繼承了 PHP `SplFileInfo` 類別，並提供各種與檔案交換資料的各種方法：

    $file = $request->file('photo');

    $file = $request->photo;

如果你想知道檔案是否存在，你可以在請求上使用 `hasFile` 方法來確認：

    if ($request->hasFile('photo')) {
        //
    }

#### 驗證成功的上傳

除了檢查檔案是否存在，你可以透過 `isValid` 方法來驗證上傳的檔案是否有效：

    if ($request->file('photo')->isValid()) {
        //
    }

#### 檔案路徑與副檔名

`UploadedFile` 類別也包含了取得檔案完整路徑和它的副檔名的方法。`extension` 將會根據內容嘗試猜測檔案的副檔名。這個副檔名可能會與客戶端提供的檔名不同：

    $path = $request->photo->path();

    $extension = $request->photo->extension();

#### 其他檔案方法

`UploadedFile` 實例上還有其他各種方法。請查閱[該類別的 API 文件](http://api.symfony.com/3.0/Symfony/Component/HttpFoundation/File/UploadedFile.html) 來獲得更多該方法的資訊。

<a name="storing-uploaded-files"></a>
### 儲存上傳檔案

要儲存上傳的檔案，你通常會使用[檔案系統](/laravel_tw/docs/5.5/filesystem)的其中一個設定。`UploadedFile` 類別有 `store` 方法，它會將上傳的檔案一到你的其中一個硬碟，這可能是本機上的檔案系統，或甚至是 Amazon S3 這種雲端儲存空間。

`store` 方法接受檔案相對於檔案系統設定的根目錄的儲存路徑。這個路徑不包含檔案名稱，因為會自動產生一個唯一 ID 作為檔案名稱。

`store` 方法也接受一個可選的第二個參數，用於指定儲存檔案的硬碟名稱。該方法會回傳檔案檔案對於硬碟根目錄的相對路徑：

    $path = $request->photo->store('images');

    $path = $request->photo->store('images', 's3');

如果你不想自動產生檔案名稱，你可以使用 `storeAs` 方法，該方法接受路徑、檔案名稱和硬碟名稱作為它的參數：

    $path = $request->photo->storeAs('images', 'filename.jpg');

    $path = $request->photo->storeAs('images', 'filename.jpg', 's3');

<a name="configuring-trusted-proxies"></a>
## 設定可信任的代理

當你執行的應用程式背後的負載平衡器的 TLS 和 SSL 憑證過後期，你可能注意到你的應用程式有時不會產生 HTTPS 連結。通常是因為應用程式從 Port 80 上的負載平衡器轉發流量時，並不知道要產生安全的連結。

要解決這個問題，你可以使用在已包含在 Laravel 應用程式中 `App\Http\Middleware\TrustProxies` 中介層，它可以讓你快速自訂可信任的負載平衡器或代理。你信任的代理應該作為陣列被列在這個中介層上的 `$proxies` 方法。除了設定信任的代理，你可以設定代理來發送包含原初請求資訊的標頭：

    <?php

    namespace App\Http\Middleware;

    use Illuminate\Http\Request;
    use Fideloper\Proxy\TrustProxies as Middleware;

    class TrustProxies extends Middleware
    {
        /**
         * 應用程式可信任的代理。
         *
         * @var array
         */
        protected $proxies = [
            '192.168.1.1',
            '192.168.1.2',
        ];

        /**
         * 目前代理 header 的映射。
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

#### 信任所有代理

如果你使用 Amazon AWS 或其他「雲端」負載平衡器的供應商，你可能會不知道平衡器的實際 IP 位址。在這種情況下，你可以使用 `**` 來信任所有代理：

    /**
     * 應用程式可信任的代理。
     *
     * @var array
     */
    protected $proxies = '**';
