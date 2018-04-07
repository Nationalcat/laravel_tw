---
layout: post
title: container
---
# 服務容器

- [簡介](#introduction)
- [綁定](#binding)
    - [綁定介面至實作](#binding-interfaces-to-implementations)
    - [情境綁定](#contextual-binding)
    - [標記](#tagging)
- [解析](#resolving)
- [容器事件](#container-events)

<a name="introduction"></a>
## 簡介

Laravel 服務容器是管理類別依賴與執行依賴注入的強力工具。依賴注入是個花俏的名詞，事實上是指：類別的依賴透過建構子「注入」，或在某些情況下透過「setter」方法注入。

讓我們來看個簡單範例：

    <?php

    namespace App\Jobs;

    use App\User;
    use Illuminate\Contracts\Mail\Mailer;
    use Illuminate\Contracts\Bus\SelfHandling;

    class PurchasePodcast implements SelfHandling
    {
        /**
         * mailer 的實作。
         */
        protected $mailer;

        /**
         * 建立一個新實例。
         *
         * @param  Mailer  $mailer
         * @return void
         */
        public function __construct(Mailer $mailer)
        {
            $this->mailer = $mailer;
        }

        /**
         * 購買 podcast。
         *
         * @return void
         */
        public function handle()
        {
            //
        }
    }

在這範例中，購買 podcast 時，`PurchasePodcast` 任務會需要寄送 e-mails。因此，我們將**注入**能寄送 e-mails 的服務。由於服務被注入，我們能容易地切換成其它實作。當測試應用程式時，我們一樣能輕易地「mock」，或建立假的 mailer 實作。

在建置強大的應用程式，以及為 Laravel 核心貢獻時，須深入理解 Laravel 服務容器。

<a name="binding"></a>
## 綁定

幾乎所有的服務容器綁定都會在[服務提供者](/laravel_tw/docs/5.2/providers)中註冊，所以下方所有的例子將示範在該情境中使用容器。不過，如果類別沒有依賴任何的介面，那麼就沒有將類別綁定至容器中的必要。並不需要告訴容器如何建構這些物件，因為它會透過 PHP 的 reflection 自動解析「實際」的物件。

在服務提供者中，你總是可以透過 `$this->app` 實例變數取得容器。我們可以使用 `bind` 方法註冊一個綁定，傳遞我們希望註冊的類別或介面名稱，以及一個會回傳類別實例的`閉包`：

    $this->app->bind('HelpSpot\API', function ($app) {
        return new HelpSpot\API($app['HttpClient']);
    });

注意，我們會接收容器本身作為參數，傳入到解析器。我們可以使用容器來解析目標的綁定中，要傳入的下一層依賴。

#### 綁定一個單例（Singleton）

`singletion` 方法綁定一個只會被解析一次的類別或介面至容器中，且爾後的呼叫都會從容器中回傳相同的實例：

    $this->app->singleton('FooBar', function ($app) {
        return new FooBar($app['SomethingElse']);
    });

#### 綁定實例

你也可以使用 `instance` 方法，綁定一個已經存在的物件實例至容器中。爾後的呼叫都會從容器中回傳給定的實例：

    $fooBar = new FooBar(new SomethingElse);

    $this->app->instance('FooBar', $fooBar);

<a name="binding-interfaces-to-implementations"></a>
### 綁定介面至實作

服務容器有個非常強大的特色，就是能夠將給定的實作綁定至介面的功能。舉個例子，讓我們假設我們有個 `EventPusher` 介面及一個 `RedisEventPusher` 實作。一旦我們撰寫完該介面的 `RedisEventPusher` 實作，就可以如下將它註冊至服務容器：

    $this->app->bind('App\Contracts\EventPusher', 'App\Services\RedisEventPusher');

這麼做會告知容器當有個類別需要 `EventPusher` 的實作時，必須注入 `RedisEventPusher`。現在我們可以在建構子中型別提示 `EventPusher` 介面，或任何其他需要透過服務容器注入依賴的地方：

    use App\Contracts\EventPusher;

    /**
     * 建立一個新的類別實例。
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

有時候，你可能有兩個類別使用到相同介面，但你希望每個類別能注入不同實作。例如，當系統收到新訂單時，我們可能想透過 [PubNub](http://www.pubnub.com/) 來發送事件，而不是 Pusher。Laravel 提供一個簡單又流利介面來定義此行為：

    $this->app->when('App\Handlers\Commands\CreateOrderHandler')
              ->needs('App\Contracts\EventPusher')
              ->give('App\Services\PubNubEventPusher');

你甚至可以傳遞一個閉包至 `give` 方法：

    $this->app->when('App\Handlers\Commands\CreateOrderHandler')
              ->needs('App\Contracts\EventPusher')
              ->give(function () {
                      // Resolve dependency...
                  });

#### 綁定基本型別

有時你可能會需要在類別內注入一些類別，同時需要注入一些基本型別的值，像是 integer。你可以使用情境綁定注入任何類別需要的值：

    $this->app->when('App\Handlers\Commands\CreateOrderHandler')
              ->needs('$maxOrderCount')
              ->give(10);

<a name="tagging"></a>
### 標記

有些時候，可能需要解析某個「分類」的綁定。例如，你正在建置一個報表彙整器，要接收有多個不同 `Report` 介面實作的陣列。註冊完 Report 實作後，可以使用 `tag` 方法為它們賦予一個標籤：

    $this->app->bind('SpeedReport', function () {
        //
    });

    $this->app->bind('MemoryReport', function () {
        //
    });

    $this->app->tag(['SpeedReport', 'MemoryReport'], 'reports');

一旦服務被標記之後，你可以透過 `tagged` 方法很簡單的解析它們全部：

    $this->app->bind('ReportAggregator', function ($app) {
        return new ReportAggregator($app->tagged('reports'));
    });

<a name="resolving"></a>
## 解析

有幾種方式可以從容器中解析一些東西。首先，你可以使用 `make` 方法，它接收你希望解析的類別或是介面的名稱：

    $fooBar = $this->app->make('FooBar');

或者，你可以像陣列一樣使用容器，因為它實作了 PHP 的 `ArrayAccess` 介面：

    $fooBar = $this->app['FooBar'];

最後，但最重要是，你可以簡單地在類別的建構子對依賴使用「型別提示」，類別將會從容器中進行解析，包含[控制器](/laravel_tw/docs/5.2/controllers)、[事件監聽器](/laravel_tw/docs/5.2/events)、[隊列任務](/laravel_tw/docs/5.2/queues)、[中介層](/laravel_tw/docs/5.2/middleware)及其他等等。實際上，大部分的物件都是經由容器解析的。

容器會自動為類別注入解析出的依賴。舉個例子，你可以在控制器的建構子型別提示應用程式中定義好 repository 。repository 會自動被解析並注入至類別中：

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Routing\Controller;
    use App\Users\Repository as UserRepository;

    class UserController extends Controller
    {
        /**
         * user repository 實例。
         */
        protected $users;

        /**
         * 建立一個新的控制器實例。
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

每當服務容器解析一個物件時就會觸發事件。你可以使用 `resolving` 方法監聽這個事件：

    $this->app->resolving(function ($object, $app) {
        // 當容器解析任何型別的物件時會被呼叫...
    });

    $this->app->resolving(FooBar::class, function (FooBar $fooBar, $app) {
        // 當容器解析「FooBar」型別的物件時會被呼叫...
    });

如你所見，被解析的物件會被傳遞至回呼中，讓你可以在傳遞前設置任何額外的屬性至物件。
