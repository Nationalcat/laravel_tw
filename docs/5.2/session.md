---
layout: post
title: session
---
# Session

- [簡介](#introduction)
- [基本用法](#basic-usage)
    - [快閃資料](#flash-data)
- [加入自定 Session 驅動](#adding-custom-session-drivers)

<a name="introduction"></a>
## 簡介

由於 HTTP 應用程式是無狀態的，session 是一種在請求間儲存用戶資訊的方試。Laravel 提供了多種後端 session，可以透過乾淨、一致的 API 進行操作。預設支援常見的 [Memcached](http://memcached.org)、[Redis](http://redis.io) 和資料庫等後端驅動。

### 設定

Session 的設定檔放在 `config/session.php`。記得看一下此設定檔中可用的選項設定及註解。Laravel 預設使用 `file` 的 session 驅動，它在大多的應用中可以良好運作。在上線的應用程式中，你可以考慮使用更快的 `memcached ` 或 `redis` 等驅動。

Session `driver` 定義了資料會存放的位置。Laravel 預設有很多不錯且的驅動：

<div class="content-list" markdown="1">
- `file` - sessions 會被儲存在 `storage/framework/sessions` 中。
- `cookie` - sessions 會被儲存安全的儲存在加密後的 cookies 中。
- `database` - sessions 會被儲存在應用程式使用的資料庫中。
- `memcached` / `redis` - sessions 會被儲存在這些快速、基於快取的儲存系統中。
- `array` - 將 sessions 儲存在簡單的 PHP 陣列中，並只存活在當次請求。
</div>

> **注意：**陣列驅動通常使用在[測試](/laravel_tw/docs/5.2/testing)上，用來防止 session 資料持續存在。

### 驅動介紹

#### 資料庫

使用 `database` session 驅動前，必需先設置包含 session 項目的資料表。下面是建立資料表的 `Schema` 範例語法：

    Schema::create('sessions', function ($table) {
        $table->string('id')->unique();
        $table->integer('user_id')->nullable();
        $table->string('ip_address', 45)->nullable();
        $table->text('user_agent')->nullable();
        $table->text('payload');
        $table->integer('last_activity');
    });

你也可使用 `session:table` Artisan 指令產生遷移檔！

    php artisan session:table

    composer dump-autoload

    php artisan migrate

#### Redis

在 Laravel 使用 Redis Session 之前，你需要先透過 Composer 安裝 `predis/predis`(~1.0) 套件。

### 其它 Session 注意事項

Laravel 框架在內部使用到 `flash` 當作 session 鍵，在增加項目到 session 時應避免使用此名稱。

如果你的 session 資料需要加密，將設定檔中的 `encrypt` 選項設為 `true`。

<a name="basic-usage"></a>
## 基本用法

#### 存取 Session

首先來取得 session。可以在控制器裡的方法，經由型別提示的 HTTP 請求取得 session 實例。請記得，控制器方法的依賴是透過 Laravel 的[服務容器](/laravel_tw/docs/5.2/container)注入：

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;

    class UserController extends Controller
    {
        /**
         * 顯示給定使用者的個人檔案。
         *
         * @param  Request  $request
         * @param  int  $id
         * @return Response
         */
        public function showProfile(Request $request, $id)
        {
            $value = $request->session()->get('key');

            //
        }
    }

當你要從 session 取值，你可以在 `get` 方法中的第二個參數內設定預設值。如果指定的鍵值不存在時，將會回傳設定的預設值。如果傳入一個 `閉包` 作為 `get` 方法的預設值，該 `必包` 將被執行並回傳結果：

    $value = $request->session()->get('key', 'default');

    $value = $request->session()->get('key', function() {
        return 'default';
    });

如果你想從 session 取得所有資料，你可以使用 `all` 方法：

    $data = $request->session()->all();

你也可使用全域的 `session` PHP 函式取得 session 中儲存的資料：

    Route::get('home', function () {
        // 取得 session 中的一筆資料...
        $value = session('key');

        // 寫入一筆資料至 session 中...
        session(['key' => 'value']);
    });

#### 判斷項目在 Session 中是否存在

`has` 方法被用於檢查項目是否存在於 session 內。如果存在將會回傳 `true`：

    if ($request->session()->has('users')) {
        //
    }

#### 儲存資料到 Session 中

取得 session 實例後，就可以使用很多方法和裡面的資料互動。例如，`put` 方法會將一個新的資料加入 session。

    $request->session()->put('key', 'value');

#### 儲存資料進 Session 陣列值中

`push` 方法可以將一個新的值加到一個 session 陣列。例如，假設 `user.teams` 這個鍵是包含團隊名稱的陣列，你可以將一個新的值加入此陣列中：

    $request->session()->push('user.teams', 'developers');

#### 從 Session 取回資料，並且刪除資料

`pull` 方法將把資料從 session 內取出，並且刪除它：

    $value = $request->session()->pull('key', 'default');

#### 從 Session 中移除資料

`forget` 方法可以從 session 內刪除一筆資料。如果你想刪除 session 內所有的資料，可以使用 `flush` 方法：

    $request->session()->forget('key');

    $request->session()->flush();

#### 重新產生 Session ID

如果你想重新產生 session ID，你可以使用 `regenerate` 方法：

    $request->session()->regenerate();

<a name="flash-data"></a>
### 快閃資料

有時候你想存入資料，並只有在下一次的請求內有效。你可以使用 `flash` 方法。使用這個方法儲存的資料，只能在下個 HTTP 請求時使用，然後就會被刪除。快閃資料對於短期的狀態訊息很有用：

    $request->session()->flash('status', 'Task was successful!');

如果需要保留閃存資料給更多的請求，可以使用 `reflash` 方法，這將會再一次保留所有的快閃資料給下一次請求。如果只想保留特定的快閃資料，可以使用 `keep` 方法：

    $request->session()->reflash();

    $request->session()->keep(['username', 'email']);

<a name="adding-custom-session-drivers"></a>
## 加入自定 Session 驅動

要加入額外驅動至 Laravel 的後端 session 中，你可以使用 `Session` [facade](/laravel_tw/docs/5.2/session) 的 `extend` 方法。你可以在[服務提供者](/laravel_tw/docs/5.2/providers) 的 `boot` 方法內呼叫 `extend` 方法：

    <?php

    namespace App\Providers;

    use Session;
    use App\Extensions\MongoSessionStore;
    use Illuminate\Support\ServiceProvider;

    class SessionServiceProvider extends ServiceProvider
    {
        /**
         * 提供註冊後執行的服務。
         *
         * @return void
         */
        public function boot()
        {
            Session::extend('mongo', function($app) {
                // Return implementation of SessionHandlerInterface...
                return new MongoSessionStore;
            });
        }

        /**
         * 在容器中註冊綁定。
         *
         * @return void
         */
        public function register()
        {
            //
        }
    }

請注意，你的自定義 session 驅動必需實作 `SessionHandlerInterface`。這個介面包含了一些需要實作的方法。一個基本的 MongoDB 實作看起來像：

    <?php

    namespace App\Extensions;

    class MongoHandler implements SessionHandlerInterface
    {
        public function open($savePath, $sessionName) {}
        public function close() {}
        public function read($sessionId) {}
        public function write($sessionId, $data) {}
        public function destroy($sessionId) {}
        public function gc($lifetime) {}
    }

由於這些方法就像快取的 `StoreInterface` 一樣並不是那麼容易理解，讓我們快速的了解每個方法的作用：

<div class="content-list" markdown="1">
- `open` 方法通常用在基於檔案的 session 儲存系統中。因為 Larvel 內建就有 `file` 的驅動，你幾乎不用在方法內寫任何東西。你可以讓這個方法保持空的。只是基於不良的介面設計（我們將在之後討論），PHP 要求必實作此方法。
- `close` 方法跟 `open` 方法很像，通常可以忽略，對大多數的驅動而言都不需要。
- `read` 方法必須根據給予的 `$sessionId` 回傳關聯的 session 資料的字串版本。在驅動中取得或儲存資料都不需要做任何的編碼跟序列化的動作，因為 Laravel 會幫你完成序列化。
- `write` 方法要能將與 `$sessionId` 關聯的 `$data` 字串存入永久儲存系統，如 MongoDB、Dynamo 等等。
- `destroy` 方法要能從永久儲存系統刪除與 `$sessionId` 關聯的資料。
- `gc` 方法要能刪除給予 `$lifetime` 之前的所有資料，`$lifetime` 是一個 UNIX 的時間戳記。在 Memcached 和 Redis 這類會自動過期的系統中，可以讓此方法保持空的。
</div>

在 session 驅動被註冊後，你可以在 `config/session.php` 的設定檔內使用 `mongo` 驅動。
