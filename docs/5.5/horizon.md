---
layout: post
title: horizon
tag: 5.5
---
# Laravel Horizon

- [介紹](#introduction)
- [安裝](#installation)
    - [設定](#configuration)
    - [儀表板認證](#dashboard-authentication)
- [執行 Horizon](#running-horizon)
    - [部署 Horizon](#deploying-horizon)
- [標籤](#tags)
- [通知](#notifications)
- [Metrics](#metrics)

<a name="introduction"></a>
## 介紹

Horizon 為你的 Laravel Redis 隊列而提供一個華麗的儀表板和程式碼驅動的設定。Horizon 可以讓你輕鬆愉快的監控隊列系統的關鍵指標，像是執行吞吐量、運行時間和任務失敗。

你所有的執行器設定都被儲存在一個簡單的設定檔中，這可以讓你的整個團隊都能在版控系統中一起維護與使用這個設定檔。

<a name="installation"></a>
## 安裝

> {note} 因為會使用到異步訊號處理，所以 Horizon 會要求 PHP 版本要 7.1 以上。

你可以使用 Composer 來安裝 Horizon 到 Laravel 專案中：

    composer require laravel/horizon

安裝 Horizon 之後，使用 Artisan 的 `vendor:publish` 指令來發佈其資源：

    php artisan vendor:publish --provider="Laravel\Horizon\HorizonServiceProvider"

<a name="configuration"></a>
### 設定

發佈了 Horizon 的資源後，它的主要設定檔會放置於 `config/horizon.php`。這個設定檔可以讓你設定執行器的選項，並且每個設定選項都包含其說明，所以請一定要徹底看過這個檔案。

#### Balance 選項

Horizon 提供你三種負載平衡方針的選擇： `simple`、`auto` 和 `false`。`simple` 方針是預設值，會在收到任務時均分給進程：

    'balance' => 'simple',

`auto` 方針會根據當前隊列的執行負載量來調整每個隊列的執行進程數量。例如，如果你的 `notifications` 隊列若有一千個等待任務，剛好 `render` 隊列又是空的，則 Horizon 會將 `notifications` 隊列分配給更多執行器，直到被處理完畢。當 `balance` 選項被設為 `false` 時，會使用預設的 Laravel 行為，這會根據設定中列出的順序來處理隊列。

<a name="dashboard-authentication"></a>
### 儀表板認證

Horizon 的儀表板路由為 `/horizon`。預設只讓你在 `local` 環境下存取這個儀表板。要為儀表板定義更具體的存取方針，你應該使用 `Horizon::auth` 方法。該 `auth` 方法接受一個應該回傳 `true` 或 `false` 的回呼，並指出使用者是否可以存取 Horizon 儀表板：

    Horizon::auth(function ($request) {
        // return true / false;
    });

<a name="running-horizon"></a>
## 執行 Horizon

一旦你在 `config/horizon.php` 設定檔中設定了你的執行器，你就可以使用 Artisan 的 `horizon` 指令來啟動 Horizon。這個指令會啟動所有被設定的執行器：

    php artisan horizon

你可以使用 Artisan 的 `horizon:pause` 和 `horizon:continue` 指令來暫停或繼續你的 Horizon 進程：

    php artisan horizon:pause

    php artisan horizon:continue

你可以使用 Artisan 的 `horizon：terminate` 指令來優雅的終止 Horizon 的主要進程。Horizon 會在當前正在處理的任務完成後退出：

    php artisan horizon:terminate

<a name="deploying-horizon"></a>
### 部署 Horizon

如果你要將 Horizon 部署到正式上線的伺服器，則應該設定一個進程監控器來監控 `php artisan horizon` 指令，並會在意外關閉時重新啟動它。當你要部署新的程式碼到伺服器時，你會需要終止 Horizon 的主要進程，以利於進程監控器在重新啟動時接收程式碼的變更。

你可以使用 Artisan 的 `horizon:terminate` 指令來優雅的終止 Horizon 主要的進程。Horizon 會在當前正在處理的任務完成後退出：

    php artisan horizon:terminate

#### Supervisor 設定

如果你是使用 Supervisor 進程監控器來管理 `horizon` 進程，那麼以下的設定檔案就足以應付了：

    [program:horizon]
    process_name=%(program_name)s
    command=php /home/forge/app.com/artisan horizon
    autostart=true
    autorestart=true
    user=forge
    redirect_stderr=true
    stdout_logfile=/home/forge/app.com/horizon.log

> {tip}如果你覺得管理伺服器很麻煩，可以考慮使用 [Laravel Forge](https://forge.laravel.com)。Forge 提供了執行一切現代化完整的 Laravel 應用程式的 Horizon 伺服器所要求的 PHP 7+。

<a name="tags"></a>
## 標籤

Horizon 可以讓你「標籤」任務，這包括寄信功能、事件廣播、通知和隊列事件監聽器。事實上，Horizon 會根據任務所附加的 Eloquent 模型來聰明的自動標記大部分任務：

    <?php

    namespace App\Jobs;

    use App\Video;
    use Illuminate\Bus\Queueable;
    use Illuminate\Queue\SerializesModels;
    use Illuminate\Queue\InteractsWithQueue;
    use Illuminate\Contracts\Queue\ShouldQueue;
    use Illuminate\Foundation\Bus\Dispatchable;

    class RenderVideo implements ShouldQueue
    {
        use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

        /**
         * video 實例。
         *
         * @var \App\Video
         */
        public $video;

        /**
         * 建立一個新的任務實例。
         *
         * @param  \App\Video  $video
         * @return void
         */
        public function __construct(Video $video)
        {
            $this->video = $video;
        }

        /**
         * 執行任務。
         *
         * @return void
         */
        public function handle()
        {
            //
        }
    }

如果這個任務有個 `id` 為 `1` 的 `App\Video` 實例隊列，它會自動去接收 `App\Video:1` 標籤。這是因為 Horizon 會檢查任何 Eloquent 模型的任務屬性。如果有找到 Eloquent 模型，Horizon 會使用模型的類別名稱與主鍵來智能標記任務：

    $video = App\Video::find(1);

    App\Jobs\RenderVideo::dispatch($video);

#### 手動標記

如果你想為一個可被隊列的物件手動定義標籤，你可以在該類別上定義 `tags` 方法：

    class RenderVideo implements ShouldQueue
    {
        /**
         * 取得任務會被分配到的標籤。
         *
         * @return array
         */
        public function tags()
        {
            return ['render', 'video:'.$this->video->id];
        }
    }

<a name="notifications"></a>
## 通知

> **注意：**在使用通知之前，你應該新增 Composer 的 `guzzlehttp/guzzle` 套件到專案中。在你要設定 Horizon 來發送 SMS 通知時，你也應該去查閱 [Nexmo 通知驅動的必要條件](https://laravel.com/docs/5.5/notifications#sms-notifications).

如果你想要當隊列中有一個需要被等待一段時間的任務就發出通知，請使用 `Horizon::routeMailNotificationsTo`、 `Horizon::routeSlackNotificationsTo` 和 `Horizon::routeSmsNotificationsTo` 方法。你可以從應用程式的 `AppServiceProvider` 中呼叫這些方法：

    Horizon::routeMailNotificationsTo('example@example.com');
    Horizon::routeSlackNotificationsTo('slack-webhook-url', '#channel');
    Horizon::routeSmsNotificationsTo('15556667777');

#### 設定等待時間的通知標準

你可以在 `config/horizon.php` 設定檔中設定「過長的等待時間」的標準秒數。在這個檔案中的 `waits` 設定選項可以讓你控制每個連接與隊列組合的過長的等待時間標準：

    'waits' => [
        'redis:default' => 60,
    ],

<a name="metrics"></a>
## Metrics

Horizon 引入一個 Metrics 儀表板，這可以為你提供任務、隊列等待時間和吞吐量的資訊。為了寫入這個儀表板，你應該透過應用程式的[任務排程器](/laravel_tw/docs/5.5/scheduling)來設定每五分鐘就執行一次 Horizon 的 Artisan 的 `snapshot` 指令：

    /**
     * 定義應用程式的指令排程。
     *
     * @param  \Illuminate\Console\Scheduling\Schedule  $schedule
     * @return void
     */
    protected function schedule(Schedule $schedule)
    {
        $schedule->command('horizon:snapshot')->everyFiveMinutes();
    }
