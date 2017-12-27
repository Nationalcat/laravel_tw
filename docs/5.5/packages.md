---
layout: post
title: packages
---
# 套件開發

- [介紹](#introduction)
    - [Facades 注意事項](#a-note-on-facades)
- [套件發現](#package-discovery)
- [服務提供者](#service-providers)
- [資源](#resources)
    - [設定](#configuration)
    - [遷移](#migrations)
    - [路由](#routes)
    - [語系](#translations)
    - [視圖](#views)
- [指令](#commands)
- [公開資源](#public-assets)
- [發佈檔案群組](#publishing-file-groups)

<a name="introduction"></a>
## 介紹

套件是 Laravel 新增功能的主要方式。例如像 [Carbon](https://github.com/briannesbitt/Carbon) 用於處理時間，或是像 [Behat](https://github.com/Behat/Behat) 這種完整的 BDD 測試框架。。

當然，會有不同種類的套件。有些套件是獨立套件，也就可以執行在任何 PHP 框架上。Carbon 和 Behat 就是獨立套件的例子。這些任何一個套件都可以被 Laravel 使用，只要你在 `composer.json` 檔案中引入它們就好。

另一方面，其他套件專門用在 Laravel。這些套件可以有路由、控制器、視圖和設定來專門增強 Laravel 應用程式。這份指南裡將主要以開發 Laravel 專屬的套件為目標進行說明。

<a name="a-note-on-facades"></a>
### Facades 注意事項

撰寫 Laravel 應用程式時，不論你是使用 Contract 或 Facade 都不會造成太大的差別，因為它們在本質上都提供了相同程度的可測試性。然而，撰寫套件時，你的套件通常無法使用 Laravel 的測試輔助函式。如果你想要撰寫套件測試就像在 Laravel 的環境中，你可以使用 [Orchestral Testbench](https://github.com/orchestral/testbench) 套件。

<a name="package-discovery"></a>
## 套件發現

在 Laravel 應用程式的 `config/app.php` 設定檔，`providers` 選項定義定義了應該被 Laravel 載入的服務提供者清單。當有人在安裝你的套件時，你通常會想要你的服務提供者能被包含到這個清單，而不是要求使用者手動新增你的服務提供者到該清單上。你可以在套件中 `composer.json` 的 `extra` 部分來定義該提供者。除了服務提供者，你也可以列出你想要註冊的任何 [facades](/laravel_tw/docs/5.5/facades)：

    "extra": {
        "laravel": {
            "providers": [
                "Barryvdh\\Debugbar\\ServiceProvider"
            ],
            "aliases": {
                "Debugbar": "Barryvdh\\Debugbar\\Facade"
            }
        }
    },

Laravel 一旦發現到你的套件設定，會在套件被安裝的時候自動註冊該服務提供者與 Facade，來為你的套件使用者打造一個方便的安裝體驗。

### 選擇套件發現

如果你是一個套件使用者，並想停用套件的發現的功能，你可以列出在應用程式的 `composer.json` 檔案 `extra` 部分中的套件清單：

    "extra": {
        "laravel": {
            "dont-discover": [
                "barryvdh/laravel-debugbar"
            ]
        }
    },

你可以在應用程式的 `dont-discover` 只是中使用 `*` 字元來為所有套件停用發現功能：

    "extra": {
        "laravel": {
            "dont-discover": [
                "*"
            ]
        }
    },

<a name="service-providers"></a>
## 服務提供者

[服務提供者](/laravel_tw/docs/5.5/providers)是你的套件與 Laravel 之間的連接點。服務提供者負責將事物綁定在 Laravel 的[服務容器](/laravel_tw/docs/5.5/container)，並告訴 Laravel 要在哪裡載入套件資源，像是視圖、設定和本地化檔案。

服務提供者繼承了 `Illuminate\Support\ServiceProvider` 類別，並包含了兩個方法：`register` 和 `boot`。基底的 `ServiceProvider` 類別會放在 `illuminate/support` Composer 套件中，你應該新增自己套件的依賴項目。想學習更多關於服務提供者的用途與結構，請查閱[服務物提供者的文件](/laravel_tw/docs/5.5/providers)。

<a name="resources"></a>
## 資源

<a name="configuration"></a>
### 設定

通常你會需要將套件的設定檔發佈到到應用程式本身的 `config` 目錄中。這可以讓套件使用者輕易的覆寫預設的設定選項。要讓你的設定檔可以被發佈，請從你的服務提供者的 `boot` 方法中呼叫 `publishes` 方法：

    /**
     * 執行服務註冊後啟動。
     *
     * @return void
     */
    public function boot()
    {
        $this->publishes([
            __DIR__.'/path/to/config/courier.php' => config_path('courier.php'),
        ]);
    }

現在，當套件使用者執行 Laravel 的 `vendor:publish` 指令時，你的檔案會被複製到指定的發佈位置。當然你的設定檔一旦被發佈，它的值就可以像任何其它設定檔案一樣被存取：

    $value = config('courier.option');

> {note} 你不應該在設定檔中設定閉包。當使用者執行 Artisan 的 `config:cache`指令時，它們無法如預期地被序列化。

#### 預設套件設定

你也可以將自己的套件設定檔與應用程式發佈的副本合併。這可以讓你的使用者只定義他們實際想要在發佈的副本上覆寫的選項。要合併設定，請在服務提供者的 `register` 方法中使用 `mergeConfigFrom` 方法：

    /**
     * 在容器中註冊綁定。
     *
     * @return void
     */
    public function register()
    {
        $this->mergeConfigFrom(
            __DIR__.'/path/to/config/courier.php', 'courier'
        );
    }

> {note} 這個方法只會合併設定陣列的第一層。如果部分使用者定義了多層的設定陣列，則不會遺失合併的選項。

<a name="routes"></a>
### 路由

如果你的套件具有路由，你可以使用 `loadRoutesFrom` 方法來載入它們。這個方法會自動確認應用程式的路由是否被快取，如果路由被快取的話，就不會載入路由檔案：

    /**
     * 執行服務註冊後啟動：
     *
     * @return void
     */
    public function boot()
    {
        $this->loadRoutesFrom(__DIR__.'/routes.php');
    }

<a name="migrations"></a>
### 遷移

如果你的套件具有[資料庫遷移](/laravel_tw/docs/5.5/migrations)，你可以使用 `loadMigrationsFrom` 方法來告訴 Laravel 該如何載入他們。 `loadMigrationsFrom` 方法接受套件的遷移路徑作為它的唯一參數：

    /**
     * 執行服務註冊後啟動。
     *
     * @return void
     */
    public function boot()
    {
        $this->loadMigrationsFrom(__DIR__.'/path/to/migrations');
    }

套件的遷移一旦被註冊，他們會自動在執行完 `php artisan migrate` 指令後運作。你不需要將它們導出到應用程式主要的 `database/migrate` 目錄。

<a name="translations"></a>
### 語系

如果你的套件具有[語系檔案](/laravel_tw/docs/5.5/localization)，你可以使用 `loadTranslationsFrom` 方法來告訴 Laravel 該如何載入它。例如，如果你的套件叫做 `courier`，你應該新增下列內容到你的服務提供者提供者的 `boot` 方法：

    /**
     * 執行服務註冊後啟動。
     *
     * @return void
     */
    public function boot()
    {
        $this->loadTranslationsFrom(__DIR__.'/path/to/translations', 'courier');
    }

套件語系是使用 `package::file.Line` 語法慣例來引用的。所以，你可以從 `messages` 檔案中載入 `courier` 套件的 `welcome` 這行，像是：

    echo trans('courier::messages.welcome');

#### 發佈語系

如果你想要發佈套件的語系到應用程式的 `resources/lang/vendor` 目錄，你可以使用服務提供者的 `publishes` 方法。`publishes` 方法接受一組套件路徑的陣列和他們想要發佈的位置。例如，要為 `courier` 套件發佈語系檔案，你可以效仿下列內容：

    /**
     * 執行服務註冊後啟動。
     *
     * @return void
     */
    public function boot()
    {
        $this->loadTranslationsFrom(__DIR__.'/path/to/translations', 'courier');

        $this->publishes([
            __DIR__.'/path/to/translations' => resource_path('lang/vendor/courier'),
        ]);
    }

現在，當套件的使用者執行 Laravel 的 Artisan `vendor:publish` 指令時，套件的語系會被發佈到指定的發佈位置。

<a name="views"></a>
### 視圖

要註冊套件的[視圖](/laravel_tw/docs/5.5/views)到 Laravel，你需要告訴 Laravel 視圖的位置在哪裡。你可以使用服務提供者的 `loadViewsFrom` 方法來做到。`loadViewsFrom` 方法接受兩個參數：視圖模板的路徑和套件名稱。例如，你的套件名稱是 `courier`，你可以在服務提供者的 `boot` 方法中新增下列內容：

    /**
     * 執行服務註冊後啟動。
     *
     * @return void
     */
    public function boot()
    {
        $this->loadViewsFrom(__DIR__.'/path/to/views', 'courier');
    }

套件視圖使用 `package::view` 語法慣例來引用。所以，一旦你的視圖路徑被註冊在服務提供者上，你可以從 `courier` 套件中載入 `admin` 視圖，像是：

    Route::get('admin', function () {
        return view('courier::admin');
    });

#### 覆寫套件視圖

當你在使用 `loadViewsFrom` 方法時，Laravel 實際會為你的視圖註冊兩個位置：應用程式的 `resources/views/vendor` 目錄和你指定的目錄。所以，使用 `courier` 範例，Laravel 首先會檢查開發者在 `resources/views/vendor/courier` 中是否提供了一個自訂的視圖版本。然後，如果視圖沒被自訂，Laravel 會搜尋你在呼叫 `loadViewsFrom` 中指定的套件視圖目錄。這麽做可以讓套件使用者容易的自訂或覆寫你的套件視圖。

#### 發佈視圖

如果你想要讓你的視圖可以發佈到應用程式的 `resources/views/vendor` 目錄，你可以使用服務提供者的 `publishes` 方法。`publishes` 方法接受一組視圖路徑的陣列和它們想要發佈的位置：

    /**
     * 執行服務註冊後啟動。
     *
     * @return void
     */
    public function boot()
    {
        $this->loadViewsFrom(__DIR__.'/path/to/views', 'courier');

        $this->publishes([
            __DIR__.'/path/to/views' => resource_path('views/vendor/courier'),
        ]);
    }

現在，當套件使用者執行 Laravel 的 Artisan `vendor:publish` 指令時，套件的視圖會被複製到指定發佈的位置。

<a name="commands"></a>
## 指令

要註冊你套件的 Artisan 指令到 Laravel 上，你可以使用 `commands` 指令。這個指令需要一組指令類別名稱的陣列。指令一旦被註冊，你可以使用 [Artisan CLI](/laravel_tw/docs/5.5/artisan) 來執行它們：

    /**
     * 啟動應用程式服務。
     *
     * @return void
     */
    public function boot()
    {
        if ($this->app->runningInConsole()) {
            $this->commands([
                FooCommand::class,
                BarCommand::class,
            ]);
        }
    }

<a name="public-assets"></a>
## 公開資源

你的套件可能會有 JavaScript、CSS 和圖像等資源。要發佈這些資源到應用程式的 `public` 目錄，請使用服務提供者的 `publishes` 方法。在這個範例中，我們還會新增一個 `public` 資源群組標籤，這可以被用於發佈相關資源群組

    /**
     * 執行服務註冊後啟動。
     *
     * @return void
     */
    public function boot()
    {
        $this->publishes([
            __DIR__.'/path/to/assets' => public_path('vendor/courier'),
        ], 'public');
    }

現在，當你的套件使用者執行 `vendor:publish` 指令時，你的資源會被複製到指定的發佈位置。由於你通常會需要在每次更新套件覆寫資源，你可以使用 `--force` 選項：

    php artisan vendor:publish --tag=public --force

<a name="publishing-file-groups"></a>
## 發佈檔案群組

你可能想要發佈套件資源群組和個別資源。例如，你可能想要讓你的使用者發佈你的套件設定檔，而不必強制發佈你的套件資源。你可以從套件的服務提供者中呼叫 `publishes` 方法來標記，藉此做到這一點。例如，讓我們使用標籤在套件服務提供者的 `boot` 方法中定義兩個發佈群組：

    /**
     * 執行服務註冊後啟動。
     *
     * @return void
     */
    public function boot()
    {
        $this->publishes([
            __DIR__.'/../config/package.php' => config_path('package.php')
        ], 'config');

        $this->publishes([
            __DIR__.'/../database/migrations/' => database_path('migrations')
        ], 'migrations');
    }

現在你的使用者可以在執行 `vendor:publish` 指令時依照它們引用的標籤來個別發佈這些群組：

    php artisan vendor:publish --tag=config
