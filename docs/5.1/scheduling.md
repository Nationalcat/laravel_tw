---
layout: post
title: scheduling
---
# 任務排程

- [簡介](#introduction)
- [定義排程](#defining-schedules)
    - [排程頻率設置](#schedule-frequency-options)
    - [避免任務重複](#preventing-task-overlaps)
- [任務輸出](#task-output)
- [任務掛勾](#task-hooks)

<a name="introduction"></a>
## 簡介

在過去，開發者必須為每個需要排程的任務產生 Cron 項目。然而令人頭疼的是任務排程不受版本控制，並且你需要 SSH 到你的伺服器增加 Cron 項目。Laravel 指令排程器允許你清楚流暢的在 Laravel 當中定義指令排程，並且僅需要在你的伺服器上增加一條 Cron 項目即可。

你的排程已經定義在 `app/Console/Kernel.php` 檔案的 `schedule` 方法中。為了方便你開始，一個簡單的範例已經包含在該方法。你可以自由的增加排程到 `Schedule` 物件中。

### 啟動排程器

底下是唯一需要加入到伺服器的 Cron 項目：

    * * * * * php /path/to/artisan schedule:run >> /dev/null 2>&1

該 Cron 將於每分鐘呼叫 Laravel 指令排程器，接著 Laravel 會衡量你排定的任務並執行預定任務。

<a name="defining-schedules"></a>
## 定義排程

你可以將所有排定的任務定義在 `App\Console\Kernel` 類別的 `schedule` 方法中。一開始，讓我們看一個任務的排程範例。在該範例，我們將排定一個在午夜被呼叫的閉包。該閉包將執行清除某個資料表的資料庫查詢：

    <?php

    namespace App\Console;

    use DB;
    use Illuminate\Console\Scheduling\Schedule;
    use Illuminate\Foundation\Console\Kernel as ConsoleKernel;

    class Kernel extends ConsoleKernel
    {
        /**
         * 你的應用程式提供的 Artisan 指令。
         *
         * @var array
         */
        protected $commands = [
            \App\Console\Commands\Inspire::class,
        ];

        /**
         * 定義應用程式的指令排程。
         *
         * @param  \Illuminate\Console\Scheduling\Schedule  $schedule
         * @return void
         */
        protected function schedule(Schedule $schedule)
        {
            $schedule->call(function () {
                DB::table('recent_users')->delete();
            })->daily();
        }
    }

除了排定 `閉包` 呼叫，你還能排定 [Artisan 指令](/laravel_tw/docs/5.1/artisan) 以及作業系統指令。舉個例子，你可以使用 `command` 方法排定一個 Artisan 指令：

    $schedule->command('emails:send --force')->daily();

`exec` 指令可被用於發送指令到作業系統：

    $schedule->exec('node /home/forge/script.js')->daily();

<a name="schedule-frequency-options"></a>
### 排程頻率設置

當然，你可以針對你的任務分配多種排程計畫：

方法  | 描述
------------- | -------------
`->cron('* * * * * *');`  |  於自訂的 Cron 排程執行該任務
`->everyMinute();`  |  於每分鐘執行該任務
`->everyFiveMinutes();`  |  於每五分鐘執行該任務
`->everyTenMinutes();`  |  於每十分鐘執行該任務
`->everyThirtyMinutes();`  |  於每三十分鐘執行該任務
`->hourly();`  |  於每小時執行該任務
`->daily();`  |  於每天午夜執行該任務
`->dailyAt('13:00');`  |  於每天 13:00 執行該任務
`->twiceDaily(1, 13);`  |  於每天 1:00 及 13:00 執行該任務
`->weekly();`  |  於每週執行該任務
`->monthly();`  |  於每月執行該任務
`->yearly();`  |  於每年執行該任務

這些方法可以合併其他限制條件，藉以產生更精細的排程。例如在某週的某幾天執行排程。舉個例子，排定一個每週一的排程：

    $schedule->call(function () {
        // 在每個禮拜一的 13:00 跑一次...
    })->weekly()->mondays()->at('13:00');

下方列出額外的限制條件：

方法  | 描述
------------- | -------------
`->weekdays();`  |  限制任務在平日
`->sundays();`  |  限制任務在星期日
`->mondays();`  |  限制任務在星期一
`->tuesdays();`  |  限制任務在星期二
`->wednesdays();`  |  限制任務在星期三
`->thursdays();`  |  限制任務在星期四
`->fridays();`  |  限制任務在星期五
`->saturdays();`  |  限制任務在星期六
`->when(Closure);`  |  限制任務基於一個為真驗證

#### 為真驗證限制條件

`when` 方法可以被用於限制任務執行與否，基於給定一個為真驗證的執行結果。換句話說，如果給定的 `閉包` 回傳 `true`，這個任務將持續被執行只要沒有其他的限制條件。

    $schedule->command('emails:send')->daily()->when(function () {
        return true;
    });

當鏈結使用 `when` 方法，排定指令只有在所有的 `when` 條件回傳 `true` 的時候才執行。

<a name="preventing-task-overlaps"></a>
### 避免任務重複

預設情況，排定的任務將被執行，即便之前相同的任務主體仍未結束。為了避免這個問題，你可以使用 `withoutOverlapping` 方法：

    $schedule->command('emails:send')->withoutOverlapping();

在這個範例，如果非執行中，`emails:send` [Artisan 指令](/laravel_tw/docs/5.1/artisan) 將於每分鐘執行。當你有些超長執行時間的任務，並且無法預測所需的時間，`withoutOverlapping` 方法將特別有幫助。

<a name="task-output"></a>
## 任務輸出

Laravel 排程器為任務排程輸出提供許多便捷的方法。首先，透過 `sendOutputTo` 你可以發送輸出到單一檔案做為後續檢查：

    $schedule->command('emails:send')
             ->daily()
             ->sendOutputTo($filePath);

如果想將輸出附加到給定的檔案，你可以使用 `appendOutputTo` 方法：

    $schedule->command('emails:send')
             ->daily()
             ->appendOutputTo($filePath);

透過 `emailOutputTo` 方法，你可以發送輸出到你所選的電子郵件。注意，你必須先透過 `sendOutputTo` 方法輸出到一個檔案。同時，在將任務輸出發送到電子郵件之前，你需要先設定 Laravel 的[電子郵件服務](/laravel_tw/docs/5.1/mail)：

    $schedule->command('foo')
             ->daily()
             ->sendOutputTo($filePath)
             ->emailOutputTo('foo@example.com');

> ** 注意：** `emailOutputTo` 與 `sendOutputTo` 方法只適用於 `command` 方法，並且不支援 `call` 方法。

<a name="task-hooks"></a>
## 任務掛勾

透過 `before` 與 `after` 方法，你能讓特定的代碼在任務完成之前及之後執行：

    $schedule->command('emails:send')
             ->daily()
             ->before(function () {
                 // 任務將要開始...
             })
             ->after(function () {
                 // 任務已完成...
             });

#### Ping 網址

透過 `pingBefore` 與 `thenPing` 方法，排程器能自動的在一個任務完成之前或之後 ping 一個給定的網址。該方法在你排定的任務進行或完成時，能有效的通知一個外部服務，例如 [Laravel Envoyer](https://envoyer.io)：

    $schedule->command('emails:send')
             ->daily()
             ->pingBefore($url)
             ->thenPing($url);

使用 `pingBefore($url)` 或 `thenPing($url)` 功能需要 Guzzle HTTP 函式庫。你可以透過將下列增加到你的 `composer.json` 檔案，使 Guzzle 加入你的專案：

    "guzzlehttp/guzzle": "~5.3|~6.0"
