---
layout: post
title: facades
tag: 5.5
---
# Facades

- [介紹](#introduction)
- [何時該使用 Facade](#when-to-use-facades)
    - [Facades Vs. 依賴注入](#facades-vs-dependency-injection)
    - [Facades Vs. 輔助函式](#facades-vs-helper-functions)
- [Facade 運作原理](#how-facades-work)
- [即時 Facades](#real-time-facades)
- [Facade 類別總清單](#facade-class-reference)

<a name="introduction"></a>
## 介紹

Facades 在應用程式的[服務容器](/laravel_tw/docs/5.5/container)中為類別提供了一個「靜態」介面。Laravel 內建了許多 facade 來存取 Laravel 幾乎全部的功能。Laravel facade 作為服務容器裡的底層類別的「靜態代理」，提供了簡潔明瞭的語法，同時維持比傳統的靜態方法更高的可測試性和彈性。

Laravel 所有的 Facade 都被定義在 `Illuminate\Support\Facades` 命名空間。所以我們能夠輕易的存取 Facade，就像是：

    use Illuminate\Support\Facades\Cache;

    Route::get('/cache', function () {
        return Cache::get('key');
    });

所有的 Laravel 文件中的許多範例都會使用 Facade 來示範框架的各種功能。

<a name="when-to-use-facades"></a>
## 何時該使用 Facade

Facades 有許多優點。它們提供了一個簡潔而有力的語法來讓你使用 Laravel 的功能，而不用再去記那些需要手動注入或冗長的類別名稱。此外，也因為 PHP 動態方法的特別用法，使得它們更容易測試。

然而， 使用 Facade 時最好要小心幾個地方。使用 Facade 最主要注意的地方是類別的範圍性蔓延。由於 Facade 容易被使用，且還不要求注入，因此會在你不斷地將 Facade 塞入類別的同時還讓你的類別變的更肥。使用依賴注入的話，會因為建構子增長而有視覺性的反饋來提醒你應該適量注入。所以，在使用 Facade 時，要格外注意類別的大小與耦合程度。

> {tip} 在建構與 Laravel 交換資料的第三方套件時。最好是注入 [Laravel contracts](/laravel_tw/docs/5.5/contracts)，而不是注入 facades。因為套件是屬於 Laravel 外部構造，所以你無法存取 Laravel 的 Facade 測試輔助函式。

<a name="facades-vs-dependency-injection"></a>
### Facades Vs. 依賴注入

依賴注入的主要優點是能夠切換注入類別的實作。這在測試的時候很好用，因為你可以注入一個 mock 或 stub，並斷言 stub 上呼叫的各種方法。

通常來說，真正的靜態類別方法不可能被 mock 或 stub。然而，因為 Facade 使用了動態方法來代替從服務容器中解析的物件方法呼叫，所以我們實際上能像測試注入類別實例一樣測試 Facade。例如，給予下面的路由：

    use Illuminate\Support\Facades\Cache;

    Route::get('/cache', function () {
        return Cache::get('key');
    });

我們能撰寫下面的測試來驗證 `Cache::get` 方法是否能被我們預期的參數所呼叫：

    use Illuminate\Support\Facades\Cache;

    /**
     * 基本功能測試範例。
     *
     * @return void
     */
    public function testBasicExample()
    {
        Cache::shouldReceive('get')
             ->with('key')
             ->andReturn('value');

        $this->visit('/cache')
             ->see('value');
    }

<a name="facades-vs-helper-functions"></a>
### Facades Vs. 輔助函式

除了 Facade，Laravel 還包含了各種可以用來執行常見任務的「輔助」函式，像是產生視圖、觸發事件、調度任務或發送 HTTP 回應。許多輔助函式都有對應的 Facade。例如，這個 Facade 呼叫與輔助函式呼叫的結果會是相等的：

    return View::make('profile');

    return view('profile');

Facade 與輔助函式之間絕對沒有實際上的區別。在使用輔助函式時，你仍可以使用對應的 Facade 來測試它們。例如，給予以下路由：

    Route::get('/cache', function () {
        return cache('key');
    });

事實上，`cache` 輔助函式會去呼叫 `Cache` Facade 底層類別的 `get` 方法。即便我們使用了輔助函式，我們也可以寫下面的測試來驗證這個方法是否使用我們預期的參數所呼叫的：

    use Illuminate\Support\Facades\Cache;

    /**
     * 基本功能測試範例。
     *
     * @return void
     */
    public function testBasicExample()
    {
        Cache::shouldReceive('get')
             ->with('key')
             ->andReturn('value');

        $this->visit('/cache')
             ->see('value');
    }

<a name="how-facades-work"></a>
## Facade 運作原理

在 Laravel 應用程式中，Facade 是一個為容器中的物件提供存取的類別。因為 `Facade` 類別使得機器可以執行。Laravel 的 Facade 以及你建立的任何自訂 Facade 都要去繼承底層的 `Illuminate\Support\Facades\Facade` 類別。

`Facade` 基本類別是使用了 `__callStatic()` 魔術方法來從你的 Facade 延遲呼叫來解析在容器中的物件。在底下範例中，會去呼叫 Laravel 的快取系統。先是預覽這些程式碼，可能會一口咬定的認為靜態方法 `get` 是被 `Cache` 類別所呼叫的：

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use Illuminate\Support\Facades\Cache;

    class UserController extends Controller
    {
        /**
         * 為特定使用者顯示個資。
         *
         * @param  int  $id
         * @return Response
         */
        public function showProfile($id)
        {
            $user = Cache::get('user:'.$id);

            return view('profile', ['user' => $user]);
        }
    }

請注意，在檔案的的上方，我們會先「導入」`Cache` facade。這個 facade 做為存取底層實作  `Illuminate\Contracts\Cache\Factory` 介面的代理。我們使用 facade 的任何呼叫將會傳送給 Laravel 快取服務的底層實例。

如果我們查看 `Illuminate\Support\Facades\Cache` 類別，你會發現沒有靜態方法 `get`：

    class Cache extends Facade
    {
        /**
         * 取得元件的註冊名稱。
         *
         * @return string
         */
        protected static function getFacadeAccessor() { return 'cache'; }
    }

反而是 `Cache` Facade 繼承了基底 `Facade` 類別，並定義了 `getFacadeAccessor()` 方法。請記住，這個方法的任務是回傳服務容器綁定的名稱。當使用者在 `Cache` Facade 上參用任何的靜態方法，Laravel 會從[服務容器](/laravel_tw/docs/5.5/container)解析被綁定的 `cache` 以及針對物件執行請求的方法（在這個範例中是 `get`）。

<a name="real-time-facades"></a>
## 即時 Facades

使用即時 Facades 可以將應用程式的任何類別是為 Facade。為了說明如何使用這個，讓我們來審視一個替代方案。例如，讓我們假設 `Podcast` 模型有一個 `publish` 方法。然而，為了發佈播客，我們需要會注入 `Publisher` 實例：

    <?php

    namespace App;

    use App\Contracts\Publisher;
    use Illuminate\Database\Eloquent\Model;

    class Podcast extends Model
    {
        /**
         * 發佈播客。
         *
         * @param  Publisher  $publisher
         * @return void
         */
        public function publish(Publisher $publisher)
        {
            $this->update(['publishing' => now()]);

            $publisher->publish($this);
        }
    }

將 Publisher 實作注入到該方法，可以讓我們對該方法做獨立測試，因為我們這樣就可以模擬注入 publisher。然而，每當我們呼叫 `publish` 方法時，都會需要我們傳入一個 `Publisher` 實例。為了產生一個即時的 Facade，只需要用將 `Facade` 前綴導入到類別的命名空間即可：

    <?php

    namespace App;

    use Facades\App\Contracts\Publisher;
    use Illuminate\Database\Eloquent\Model;

    class Podcast extends Model
    {
        /**
         * 發佈播客。
         *
         * @return void
         */
        public function publish()
        {
            $this->update(['publishing' => now()]);

            Publisher::publish($this);
        }
    }

使用即時 Facade 時，Publisher 實作會藉由服務容器使用「Facade」前綴後面顯示的介面或類別名稱來解析而得。在測試時，我們能夠使用 Laravel 的內建 Facade 測試輔助函式來模擬這個方法呼叫：

    <?php

    namespace Tests\Feature;

    use App\Podcast;
    use Tests\TestCase;
    use Facades\App\Contracts\Publisher;
    use Illuminate\Foundation\Testing\RefreshDatabase;

    class PodcastTest extends TestCase
    {
        use RefreshDatabase;

        /**
         * 一個測試範例。
         *
         * @return void
         */
        public function test_podcast_can_be_published()
        {
            $podcast = factory(Podcast::class)->create();

            Publisher::shouldReceive('publish')->once()->with($podcast);

            $podcast->publish();
        }
    }

<a name="facade-class-reference"></a>
## Facade 類別總清單

在下方你可以找到每個 facade 及其底層的類別。這個工具對於透過給定 facade 的來源快速尋找 API 文件相當有用。可應用的[服務容器綁定](/laravel_tw/docs/5.5/container)關鍵字也包含在裡面。

Facade  |  類別  |  服務容器綁定
------------- | ------------- | -------------
App  |  [Illuminate\Foundation\Application](https://laravel.com/api/{% raw %} {{version}} {% endraw %}/Illuminate/Foundation/Application.html)  |  `app`
Artisan  |  [Illuminate\Contracts\Console\Kernel](https://laravel.com/api/{% raw %} {{version}} {% endraw %}/Illuminate/Contracts/Console/Kernel.html)  |  `artisan`
Auth  |  [Illuminate\Auth\AuthManager](https://laravel.com/api/{% raw %} {{version}} {% endraw %}/Illuminate/Auth/AuthManager.html)  |  `auth`
Auth (Instance)  |  [Illuminate\Contracts\Auth\Guard](https://laravel.com/api/{% raw %} {{version}} {% endraw %}/Illuminate/Contracts/Auth/Guard.html)  |  `auth.driver`
Blade  |  [Illuminate\View\Compilers\BladeCompiler](https://laravel.com/api/{% raw %} {{version}} {% endraw %}/Illuminate/View/Compilers/BladeCompiler.html)  |  `blade.compiler`
Broadcast  |  [Illuminate\Contracts\Broadcasting\Factory](https://laravel.com/api/{% raw %} {{version}} {% endraw %}/Illuminate/Contracts/Broadcasting/Factory.html)  |  &nbsp;
Broadcast (Instance)  |  [Illuminate\Contracts\Broadcasting\Broadcaster](https://laravel.com/api/{% raw %} {{version}} {% endraw %}/Illuminate/Contracts/Broadcasting/Broadcaster.html)  |  &nbsp;
Bus  |  [Illuminate\Contracts\Bus\Dispatcher](https://laravel.com/api/{% raw %} {{version}} {% endraw %}/Illuminate/Contracts/Bus/Dispatcher.html)  |  &nbsp;
Cache  |  [Illuminate\Cache\CacheManager](https://laravel.com/api/{% raw %} {{version}} {% endraw %}/Illuminate/Cache/CacheManager.html)  |  `cache`
Cache (Instance)  |  [Illuminate\Cache\Repository](https://laravel.com/api/{% raw %} {{version}} {% endraw %}/Illuminate/Cache/Repository.html)  |  `cache.store`
Config  |  [Illuminate\Config\Repository](https://laravel.com/api/{% raw %} {{version}} {% endraw %}/Illuminate/Config/Repository.html)  |  `config`
Cookie  |  [Illuminate\Cookie\CookieJar](https://laravel.com/api/{% raw %} {{version}} {% endraw %}/Illuminate/Cookie/CookieJar.html)  |  `cookie`
Crypt  |  [Illuminate\Encryption\Encrypter](https://laravel.com/api/{% raw %} {{version}} {% endraw %}/Illuminate/Encryption/Encrypter.html)  |  `encrypter`
DB  |  [Illuminate\Database\DatabaseManager](https://laravel.com/api/{% raw %} {{version}} {% endraw %}/Illuminate/Database/DatabaseManager.html)  |  `db`
DB (Instance)  |  [Illuminate\Database\Connection](https://laravel.com/api/{% raw %} {{version}} {% endraw %}/Illuminate/Database/Connection.html)  |  `db.connection`
Event  |  [Illuminate\Events\Dispatcher](https://laravel.com/api/{% raw %} {{version}} {% endraw %}/Illuminate/Events/Dispatcher.html)  |  `events`
File  |  [Illuminate\Filesystem\Filesystem](https://laravel.com/api/{% raw %} {{version}} {% endraw %}/Illuminate/Filesystem/Filesystem.html)  |  `files`
Gate  |  [Illuminate\Contracts\Auth\Access\Gate](https://laravel.com/api/{% raw %} {{version}} {% endraw %}/Illuminate/Contracts/Auth/Access/Gate.html)  |  &nbsp;
Hash  |  [Illuminate\Contracts\Hashing\Hasher](https://laravel.com/api/{% raw %} {{version}} {% endraw %}/Illuminate/Contracts/Hashing/Hasher.html)  |  `hash`
Lang  |  [Illuminate\Translation\Translator](https://laravel.com/api/{% raw %} {{version}} {% endraw %}/Illuminate/Translation/Translator.html)  |  `translator`
Log  |  [Illuminate\Log\Writer](https://laravel.com/api/{% raw %} {{version}} {% endraw %}/Illuminate/Log/Writer.html)  |  `log`
Mail  |  [Illuminate\Mail\Mailer](https://laravel.com/api/{% raw %} {{version}} {% endraw %}/Illuminate/Mail/Mailer.html)  |  `mailer`
Notification  |  [Illuminate\Notifications\ChannelManager](https://laravel.com/api/{% raw %} {{version}} {% endraw %}/Illuminate/Notifications/ChannelManager.html)  |  &nbsp;
Password  |  [Illuminate\Auth\Passwords\PasswordBrokerManager](https://laravel.com/api/{% raw %} {{version}} {% endraw %}/Illuminate/Auth/Passwords/PasswordBrokerManager.html)  |  `auth.password`
Password (Instance)  |  [Illuminate\Auth\Passwords\PasswordBroker](https://laravel.com/api/{% raw %} {{version}} {% endraw %}/Illuminate/Auth/Passwords/PasswordBroker.html)  |  `auth.password.broker`
Queue  |  [Illuminate\Queue\QueueManager](https://laravel.com/api/{% raw %} {{version}} {% endraw %}/Illuminate/Queue/QueueManager.html)  |  `queue`
Queue (Instance)  |  [Illuminate\Contracts\Queue\Queue](https://laravel.com/api/{% raw %} {{version}} {% endraw %}/Illuminate/Contracts/Queue/Queue.html)  |  `queue.connection`
Queue (Base Class)  |  [Illuminate\Queue\Queue](https://laravel.com/api/{% raw %} {{version}} {% endraw %}/Illuminate/Queue/Queue.html)  |  &nbsp;
Redirect  |  [Illuminate\Routing\Redirector](https://laravel.com/api/{% raw %} {{version}} {% endraw %}/Illuminate/Routing/Redirector.html)  |  `redirect`
Redis  |  [Illuminate\Redis\RedisManager](https://laravel.com/api/{% raw %} {{version}} {% endraw %}/Illuminate/Redis/RedisManager.html)  |  `redis`
Redis (Instance)  |  [Illuminate\Redis\Connections\Connection](https://laravel.com/api/{% raw %} {{version}} {% endraw %}/Illuminate/Redis/Connections/Connection.html)  |  `redis.connection`
Request  |  [Illuminate\Http\Request](https://laravel.com/api/{% raw %} {{version}} {% endraw %}/Illuminate/Http/Request.html)  |  `request`
Response  |  [Illuminate\Contracts\Routing\ResponseFactory](https://laravel.com/api/{% raw %} {{version}} {% endraw %}/Illuminate/Contracts/Routing/ResponseFactory.html)  |  &nbsp;
Response (Instance)  |  [Illuminate\Http\Response](https://laravel.com/api/{% raw %} {{version}} {% endraw %}/Illuminate/Http/Response.html)  |  &nbsp;
Route  |  [Illuminate\Routing\Router](https://laravel.com/api/{% raw %} {{version}} {% endraw %}/Illuminate/Routing/Router.html)  |  `router`
Schema  |  [Illuminate\Database\Schema\Builder](https://laravel.com/api/{% raw %} {{version}} {% endraw %}/Illuminate/Database/Schema/Builder.html)  |  &nbsp;
Session  |  [Illuminate\Session\SessionManager](https://laravel.com/api/{% raw %} {{version}} {% endraw %}/Illuminate/Session/SessionManager.html)  |  `session`
Session (Instance)  |  [Illuminate\Session\Store](https://laravel.com/api/{% raw %} {{version}} {% endraw %}/Illuminate/Session/Store.html)  |  `session.store`
Storage  |  [Illuminate\Filesystem\FilesystemManager](https://laravel.com/api/{% raw %} {{version}} {% endraw %}/Illuminate/Filesystem/FilesystemManager.html)  |  `filesystem`
Storage (Instance)  |  [Illuminate\Contracts\Filesystem\Filesystem](https://laravel.com/api/{% raw %} {{version}} {% endraw %}/Illuminate/Contracts/Filesystem/Filesystem.html)  |  `filesystem.disk`
URL  |  [Illuminate\Routing\UrlGenerator](https://laravel.com/api/{% raw %} {{version}} {% endraw %}/Illuminate/Routing/UrlGenerator.html)  |  `url`
Validator  |  [Illuminate\Validation\Factory](https://laravel.com/api/{% raw %} {{version}} {% endraw %}/Illuminate/Validation/Factory.html)  |  `validator`
Validator (Instance)  |  [Illuminate\Validation\Validator](https://laravel.com/api/{% raw %} {{version}} {% endraw %}/Illuminate/Validation/Validator.html)  |  &nbsp;
View  |  [Illuminate\View\Factory](https://laravel.com/api/{% raw %} {{version}} {% endraw %}/Illuminate/View/Factory.html)  |  `view`
View (Instance)  |  [Illuminate\View\View](https://laravel.com/api/{% raw %} {{version}} {% endraw %}/Illuminate/View/View.html)  |  &nbsp;
