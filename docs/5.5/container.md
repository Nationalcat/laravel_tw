---
layout: post
title: container
---
# 服務容器

- [介紹](#introduction)
- [綁定](#binding)
    - [綁定基礎](#binding-basics)
    - [綁定介面至實作](#binding-interfaces-to-implementations)
    - [情境綁定](#contextual-binding)
    - [標記](#tagging)
- [解析](#resolving)
    - [Make 方法](#the-make-method)
    - [自動注入](#automatic-injection)
- [容器事件](#container-events)
- [PSR-11](#psr-11)

<a name="introduction"></a>
## 介紹

Laravel 服務容器是管理類別依賴與執行依賴注入的強大工具。依賴注入是個花俏的名詞，實際上是指：類別的依賴透過建構子「注入」，或在某些情況下透過「setter」方法注入。

讓我們看個簡單的範例：

    <?php

    namespace App\Http\Controllers;

    use App\User;
    use App\Repositories\UserRepository;
    use App\Http\Controllers\Controller;

    class UserController extends Controller
    {
        /**
         * 使用者 repository 的實作。
         *
         * @var UserRepository
         */
        protected $users;

        /**
         * 建立新控制器實例。
         *
         * @param  UserRepository  $users
         * @return void
         */
        public function __construct(UserRepository $users)
        {
            $this->users = $users;
        }

        /**
         * 顯示個人資料給特定使用者。
         *
         * @param  int  $id
         * @return Response
         */
        public function show($id)
        {
            $user = $this->users->find($id);

            return view('user.profile', ['user' => $user]);
        }
    }

在這個範例中，`UserController` 需要從資料來源中取得使用者。所以我們將**注入**一個服務讓我們能取得使用者。在這個情境下，我們的 `UserRepository` 像是使用 [Eloquent](/laravel_tw/docs/5.5/eloquent) 一樣，從資料庫取得使用者資訊。然而，由於 Repository 是被注入的，我們可以更容易的抽換成其他的實作。我們可以很容易的「mock」，或是建立一個假的 `UserRepository` 實作來測試我們的應用程式。

深入理解 Laravel 的服務容器對於建立一個強大、大型的應用程式以及為 Laravel 核心程式碼本身做出貢獻是必要的。

<a name="binding"></a>
## 綁定

<a name="binding-basics"></a>
### 綁定基礎

幾乎所有的服務容器綁定都會在[服務提供者](/laravel_tw/docs/5.5/providers)中被註冊，所以下方所有的範例將示範在該情境中使用容器。

> {tip} 如果類別沒有依賴任何的介面，那麼就沒有將類別綁定至容器中的必要。並不需要告訴容器如何建構這些物件，因為它會透過 PHP 的 reflection 自動解析出物件。

#### 簡易綁定

在服務提供者中，隨時可以透過 `$this->app` 物件屬性來取得容器。我們可以使用 `bind` 方法註冊一個綁定，並傳遞一組我們希望綁定的類別或介面名稱作為第一個參數，接著第二個參數放入用來回傳類別實例的`閉包`：

    $this->app->bind('HelpSpot\API', function ($app) {
        return new HelpSpot\API($app->make('HttpClient'));
    });

注意，我們將容器本身作為參數，傳入到解析器。然後我們可以使用該容器來解析我們正在建構的物件的子依賴。

#### 綁定單一實例

`singleton` 方法綁定一個類別或介面至容器中，只會被解析一次，且爾後的呼叫都會從容器中回傳相同的實例：

    $this->app->singleton('HelpSpot\API', function ($app) {
        return new HelpSpot\API($app->make('HttpClient'));
    });

#### 綁定實例

你也可以使用 `instance` 方法，綁定一個已經存在的物件實例至容器中。爾後的呼叫都會從容器中回傳給訂的實例：

    $api = new HelpSpot\API(new HttpClient);

    $this->app->instance('HelpSpot\API', $api);

#### 綁定原始值

有時候，你的類別可能會接收一些類別的注入，同時也需要注入原始值，像是整數。這時你可以使用情境綁定輕鬆地注入此類別需要的任何值：

    $this->app->when('App\Http\Controllers\UserController')
              ->needs('$variableName')
              ->give($value);

<a name="binding-interfaces-to-implementations"></a>
### 綁定介面至實作

服務容器有個非常強大的特色，就是將給定的實作綁定至介面。舉例來說，假設我們有個 `EventPusher` 介面以及一個 `RedisEventPusher` 實作。一旦我們撰寫完這個介面的 `RedisEventPusher` 實作，我們可以像是如下將它註冊至服務容器：

    $this->app->bind(
        'App\Contracts\EventPusher',
        'App\Services\RedisEventPusher'
    );

這麼做會告知容器：當一個類別需要一個 `EventPusher` 的實作，這段程式碼告訴容器應該注入 `RedisEventPusher`。現在我們可以在建構子或透過服務容器在任何地方依賴注入 `EventPusher` 介面的類型提示。

    use App\Contracts\EventPusher;

    /**
     * 建立新類別實例。
     *
     * @param  EventPusher  $pusher
     * @return void
     */
    public function __construct(EventPusher $pusher)
    {
        $this->pusher = $pusher;
    }

<a name="contextual-binding"></a>
### 情境綁定

有時候，你可能有兩個類別使用到相同介面，但你希望每個類別能注入不同實作。例如，兩個控制器可能依賴於 `Illuminate\Contracts\Filesystem\Filesystem` [contract](/laravel_tw/docs/5.5/contracts) 的不同實作。Laravel 提供一個簡單又流利介面來定義此行為：

    use Illuminate\Support\Facades\Storage;
    use App\Http\Controllers\PhotoController;
    use App\Http\Controllers\VideoController;
    use Illuminate\Contracts\Filesystem\Filesystem;

    $this->app->when(PhotoController::class)
              ->needs(Filesystem::class)
              ->give(function () {
                  return Storage::disk('local');
              });

    $this->app->when(VideoController::class)
              ->needs(Filesystem::class)
              ->give(function () {
                  return Storage::disk('s3');
              });

<a name="tagging"></a>
### 標記

偶爾你可能需要解析特定「類別」的所有綁定。例如，也許你正在建立一個報表彙整器來接收一個不同 `Report` 介面實作的陣列 。在註冊 `Report` 實作之後，你可以使用 `tag` 方法為它們賦予一個標籤：

    $this->app->bind('SpeedReport', function () {
        //
    });

    $this->app->bind('MemoryReport', function () {
        //
    });

    $this->app->tag(['SpeedReport', 'MemoryReport'], 'reports');

一旦服務被標記之後，你可以簡單地透過 `tagged` 方法解析它們全部：

    $this->app->bind('ReportAggregator', function ($app) {
        return new ReportAggregator($app->tagged('reports'));
    });

<a name="resolving"></a>
## 解析

<a name="the-make-method"></a>
#### `make` 方法

你可以使用 `make` 方法從容器中解析出類別實例，`make` 方法接收你希望解析的類別或是介面的名稱：

    $api = $this->app->make('HelpSpot\API');

如果你的程式碼所在位置無法存取 `$app` 變數，可以使用此全域的輔助函數 `resolve`：

    $api = resolve('HelpSpot\API');

如果你有一些類別的依賴不能透過容器來解析，你可以將它們組成為關聯陣列傳入到 `makeWith` 方法來注入：

    $api = $this->app->makeWith('HelpSpot\API', ['id' => 1]);

<a name="automatic-injection"></a>
#### 自動注入

此外，也是最常用的，你可以簡單地在類別的建構子中對依賴「型別提示」來解析出容器中物件，包含[控制器](/laravel_tw/docs/5.5/controllers)、[事件監聽器](/laravel_tw/docs/5.5/events)、[隊列任務](/laravel_tw/docs/5.5/queues)、[中介層](/laravel_tw/docs/5.5/middleware)及其他等等。在實際情形中，這就是為何大部分的物件都是由容器中解析。

例如，你可以透過你應用程式控制器的建構子型別提示一個被定義的 Repository。Repository 將自動地被解析並注入到類別中：

    <?php

    namespace App\Http\Controllers;

    use App\Users\Repository as UserRepository;

    class UserController extends Controller
    {
        /**
         * 使用者 repository 的實作。
         */
        protected $users;

        /**
         * 建立新控制器實例。
         *
         * @param  UserRepository  $users
         * @return void
         */
        public function __construct(UserRepository $users)
        {
            $this->users = $users;
        }

        /**
         * 顯示給定 ID 的使用者。
         *
         * @param  int  $id
         * @return Response
         */
        public function show($id)
        {
            //
        }
    }

<a name="container-events"></a>
## 容器事件

當容器解析任何的類型物件時被呼叫。你可以使用 `resolving` 方法監聽這個事件：

    $this->app->resolving(function ($object, $app) {
        // 當容器解析任何的類型物件時被呼叫...
    });

    $this->app->resolving(HelpSpot\API::class, function ($api, $app) {
        // 當容器解析「HelpSpot\API」的類型物件時被呼叫...
    });

如你所見，被解析的物件會被傳遞至回呼函式中，在它被提供給它的消費者之前，允許你在物件上設定任何額外的屬性。

<a name="psr-11"></a>
## PSR-11

Laravel 的服務容器實作了 [PSR-11](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-11-container.md) 介面。所以，你可以注入符合 PSR-11 型別提示的容器介面來取得 Laravel 容器的實例：

    use Psr\Container\ContainerInterface;

    Route::get('/', function (ContainerInterface $container) {
        $service = $container->get('Service');

        //
    });

> {note} 如果介面或類別沒有順利綁定於容器中，就會呼叫 `get` 方法並拋出異常訊息。
