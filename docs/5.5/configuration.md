---
layout: post
title: configuration
---
# Configuration

- [介紹](#introduction)
- [環境設定](#environment-configuration)
    - [取得環境設定](#retrieving-environment-configuration)
    - [確認目前環境](#determining-the-current-environment)
- [存取設定值](#accessing-configuration-values)
- [快取設定](#configuration-caching)
- [維護模式](#maintenance-mode)

<a name="introduction"></a>
## 介紹

Laravel 框架的所有設定檔都存放在 `config` 目錄中。每個選項都有所屬的文件，你可以隨意看看這些文件，並熟悉這些可用的選項。

<a name="environment-configuration"></a>
## 環境設定

根據應用程式所執行的環境給予適合的設定是有助於開發的。例如，你可能希望針對開發環境和正式環境使用不同的快取驅動。

為了做到這件事，Laravel 透過 Vance Lucas 的 [DotEnv](https://github.com/vlucas/phpdotenv) PHP 函式庫來達到這項需求。在你新安裝的 Laravel 中，其根目錄會有 `.env.example` 的檔案。如果你是透過 Composer 安裝 Laravel 的話，檔案會自動重新命名為  `.env`。如果沒有的話，你需要手動重新命名 `.env.example` 檔案。

你的 `.env` 檔案不應該被提交到版本控制上，因為每個開發或正式環境未必會使用相同的環境設定。除此之外，如果有入侵者想嘗試存取你的版本控制系統，這會造成資安漏洞，因為所有敏感資訊都會被外流。

如果你是團隊開發，你可能希望應用程式能保留 `.env.example` 檔案。可以在範例設定檔放置預設值，這樣團隊的其他開發者能清楚地知道執行應用程式需要哪些環境設定值。 你也可以建立 `.env.testing` 檔案。在執行 PHPUnit 測試或執行 Artisan 指令並搭配 `--env=testing` 選項的時候，該檔案會覆蓋 `.env` 檔案內的環境設定值。

> {tip} 在你的 `.env` 檔案內的任何變數都能被外部環境變數覆蓋，像是系統層級或伺服器層級的環境變數。

<a name="retrieving-environment-configuration"></a>
### 取得環境設定

當你的應用程式收到請求時，`.env` 檔案的所有變數會被載到 $_ENV 這個 PHP 超級全域變數。然而，你可以使用 `env` 輔助函式去接收來自你的設定檔裡的一些變數。實際上，如果你有先去看 Laravel 設定檔，你會注意到有幾個選項已經使用了這個輔助函式：

    'debug' => env('APP_DEBUG', false),

傳遞給 `env` 函式的第二個值是「預設值」，該值會用在沒有環境變數存在的情況下給定預設值。

<a name="determining-the-current-environment"></a>
### 確認目前環境

目前應用程式的環境透過從你的  `.env` 檔案中的 `APP_ENV` 變數去判定的。你可以透過在 `App` [facade](/laravel_tw/docs/5.5/facades) 上的 `environment` 方法來存取這個值：

    $environment = App::environment();

你也可以傳遞參數到 `environment` 方法來檢查給定的值是否與環境變數一致。如果給定的值與條件上的環境變數之值相同，該方法就會回傳 `true`：

    if (App::environment('local')) {
        // 環境是本機端
    }

    if (App::environment(['local', 'staging'])) {
        // 環境是本機端或是 Stage...
    }

> {tip} 目前應用程式環境檢測可以被伺服器層級的 `APP_ENV` 環境變數覆蓋。這有助於你在共享不同的環境設定給相同的應用程式上，因此你可以在伺服器的設定中設置主機環境來配合給定的環境。

<a name="accessing-configuration-values"></a>
## 存取設定值

你可以在你的應用程式任何地方輕易地使用全域 `config` 輔助函式來存取你的設定值。可以使用「點」來存取設定值，這其中還包括你想要存取的檔案和選項名稱。也可以指定想要的值，或者如果設定的選項不存就回傳預設值：

    $value = config('app.timezone');

要在執行時去給予設定的值，請傳遞一組陣列給 `config` 輔助函式：

    config(['app.timezone' => 'America/Chicago']);

<a name="configuration-caching"></a>
## 快取你的設定

想讓你的應用程式更快一些，你應該使用 Artisan 的 `config:cache` 指令把全部的設定檔快取到一個檔案。這會將你應用程式中的所有設定選項合併成單一個檔案，讓框架能更快速的載入。

通常來說，你應該將執行 `php artisan config:cache` 指令當作正式產品部署的例行公事。該指令不應該執行在本地開環境，因為設定選項會因開發求而經常的更動。

> {note} 如果你在部署階段執行 `config:cache` 指令，你應該確定你只從設定檔中呼叫 `env` 函式。

<a name="maintenance-mode"></a>
## 維護模式

當你的應用程式正在維護時，所有傳遞至應用程式的請求都會顯示自訂的視圖。在你要更新或進行維護的時候，能輕易的「關閉」你的應用程式。預設的中介層堆疊會為你的應用程式檢核是否正在使用維護模式。如果應用程式進入維護模式時，`MaintenanceModeException` 會拋出 503 HTTP 狀態碼。

若要啟動維護模式，可以簡單地執行 Artisan 的 `down` 指令：

    php artisan down

你也可以給 `down` 指令提供 `message` 和 `retry` 選項。`message` 選項的值可用於顯示或記錄自訂的訊息。而 `retry` 選項的值將用來當 HTTP 標頭 `Retry-After` 的值：

    php artisan down --message="Upgrading Database" --retry=60

若要關閉維護模式，可以使用 `up` 指令：

    php artisan up

> {tip} 維護模式預設的模板放在 `resources/views/errors/503.blade.php`。你可以根據應用程式的需求來自由的修改此視圖。

#### 維護模式與隊列

當你的應用程式正處於維護模式時，[隊列中的任務](/laravel_tw/docs/5.5/queues)將不會被處理。直到應用程式離開維護模式才會繼續處理任務。

#### 維護模式的替代方案

因為維護模式需要你的應用程式幾秒鐘的停機時間，所以你如果想要達到零停機時間的部署，你可以考慮使用像是 [Envoyer](https://envoyer.io) 之類的第三方服務。
