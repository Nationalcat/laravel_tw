---
layout: post
title: providers
tag: 5.3
---
# 服務提供者（Provider）

- [簡介](#introduction)
- [編寫服務提供者](#writing-service-providers)
    - [註冊方法（register）](#the-register-method)
    - [啟動方法（boot）](#the-boot-method)
- [註冊提供者](#registering-providers)
- [緩載提供者](#deferred-providers)

<a name="introduction"></a>
## 簡介

服務提供者（Provider）是所有 Laravel 應用程式啟動的中心所在。你自己的應用程式，以及所有的 Laravel 核心服務，都是透過服務提供者啟動的。

但我們所說的「啟動」指的是什麼？一般而言，我們指的是**註冊**事物，包括註冊服務容器綁定、事件監聽器、中介層，甚至路由。服務提供者是設定你的應用程式的中心所在。

若你開啟包含於 Laravel 中的 `config/app.php` 此一檔案，你會看到 `providers` 陣列。這些是所有你的應用程式會載入的服務提供者類別。當然，它們之中有很多屬於「緩載」提供者，意味著除非真正需要它們所提供的服務，否則它們並不會在每一個請求中都被載入。

在這份概述中，你會學到如何編寫你自己的服務提供者，並將它們註冊於你的 Laravel 應用程式。

<a name="writing-service-providers"></a>
## 編寫服務提供者

所有的服務提供者都繼承了 `Illuminate\Support\ServiceProvider` 類別。大部分服務提供者都包含兩個方法：`register` 和 `boot`。在 `register` 方法中，你應該**只將事物綁定至[服務容器](/laravel_tw/docs/5.3/container)**之中。你永遠不該試圖在 `register` 方法中註冊任何事件監聽器、路由或任何其他功能。

Artisan 命令列介面可以很容易地透過 `make:provider` 指令產生新的提供者：

    php artisan make:provider RiakServiceProvider

<a name="the-register-method"></a>
### 註冊方法（register）

如同之前提到的，在 `register` 方法中，，你應該**只將事物綁定至[服務容器](/laravel_tw/docs/5.3/container)**之中。你永遠不該試圖在 `register` 方法中註冊任何事件監聽器、路由或任何其他功能。否則的話，你可能會意外地使用到由尚未載入的服務提供者所提供的服務。

讓我們來看看基本的服務提供者，在你的服務提供者中的任何方法，永遠可以使用 `$app` 屬性來存取服務容器：

    <?php

    namespace App\Providers;

    use Riak\Connection;
    use Illuminate\Support\ServiceProvider;

    class RiakServiceProvider extends ServiceProvider
    {
        /**
         * 在容器中註冊綁定
         *
         * @return void
         */
        public function register()
        {
            $this->app->singleton(Connection::class, function ($app) {
                return new Connection(config('riak'));
            });
        }
    }

此服務提供者只定義了一個 `register` 方法，並在服務容器中使用此方法定義了一份 `Riak\Connection` 的實作。若你不瞭解服務容器是如何運作的，去看看[相關文件](/laravel_tw/docs/5.3/container)。

<a name="the-boot-method"></a>
### 啟動方法（boot）

若欲在服務提供者中註冊一個視圖 composer，那該做些什麼呢？應在 `boot` 方法中完成。**此方法會在所有其他的服務提供者被註冊後才被呼叫**，意味著你可以取得已經被框架註冊的所有其他服務：

    <?php

    namespace App\Providers;

    use Illuminate\Support\ServiceProvider;

    class ComposerServiceProvider extends ServiceProvider
    {
        /**
         * Bootstrap any application services.
         *
         * @return void
         */
        public function boot()
        {
            view()->composer('view', function () {
                //
            });
        }
    }

#### 啟動方法依賴注入

你可以對服務提供者的 `boot` 方法作依賴的型別提式。[服務容器](/laravel_tw/docs/5.3/container)會自動注入你所需要的任何依賴：

    use Illuminate\Contracts\Routing\ResponseFactory;

    public function boot(ResponseFactory $response)
    {
        $response->macro('caps', function ($value) {
            //
        });
    }

<a name="registering-providers"></a>
## 註冊提供者

所有的服務提供者都在 `config/app.php` 此設定檔中被註冊。這個檔案包含了一個 `providers` 陣列，你可以在其中列出你所有服務提供者的類別名稱。此陣列預設會列出一組 Laravel 的核心服務提供者。這些提供者啟動了 Laravel 的核心元件，例如郵件寄送器、隊列、快取及其他等等。

註冊你的服務提供者，只需簡單地加進這個陣列中：

    'providers' => [
        // 其他的服務提供者

        App\Providers\ComposerServiceProvider::class,
    ],

<a name="deferred-providers"></a>
## 緩載提供者

若你的提供者**僅於**[服務容器](/laravel_tw/docs/5.3/container)中註冊綁定，你可以選擇延緩其註冊，直到真正需要其中已註冊的綁定。延緩像這樣的提供者載入可增進你的應用程式的效能，因為這樣就毋須每個請求都從檔案系統將其載入。

Laravel 編譯並儲存了一份清單，包括所有由延緩服務提供者所提供的服務，以及其服務提供者類別的名稱。因此，只有在當你企圖解析其中的服務時，Laravel 才會載入該服務提供者。

要延緩提供者載入，將 `defer` 屬性設定為 `true`，並定義一個 `provides` 方法。`provides` 方法會回傳提供者所註冊的服務容器綁定：

    <?php

    namespace App\Providers;

    use Riak\Connection;
    use Illuminate\Support\ServiceProvider;

    class RiakServiceProvider extends ServiceProvider
    {
        /**
         * 指定提供者載入是否延緩
         *
         * @var bool
         */
        protected $defer = true;

        /**
         * 註冊服務提供者
         *
         * @return void
         */
        public function register()
        {
            $this->app->singleton(Connection::class, function ($app) {
                return new Connection($app['config']['riak']);
            });
        }

        /**
         * 取得提供者所提供的服務
         *
         * @return array
         */
        public function provides()
        {
            return [Connection::class];
        }

    }
