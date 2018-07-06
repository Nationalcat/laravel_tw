---
layout: post
title: scheduling
tag: 5.5
---
# 任務排程

- [簡介](#introduction)
- [定義排程](#defining-schedules)
    - [把 Artisan 指令加入排程](#scheduling-artisan-commands)
    - [把隊列任務加入排程](#scheduling-queued-jobs)
    - [把 Shell 指令加入排程](#scheduling-shell-commands)
    - [排程頻率設置](#schedule-frequency-options)
    - [避免任務重複](#preventing-task-overlaps)
    - [維護模式](#maintenance-mode)
- [任務輸出](#task-output)
- [任務掛勾](#task-hooks)

<a name="introduction"></a>
## 簡介

過去開發者必須為每個需要排程的任務產生 Cron 項目。然而令人頭疼的是任務排程不受版本控制，且需要 SSH 到伺服器以增加 Cron 項目。

Laravel 指令排程器允許你清楚流暢地的在 Laravel 當中定義指令排程。使用排程器時，僅需要在伺服器上增加一條 Cron 項目即可。排程已經定義在 `app/Console/Kernel.php` 檔案的 `schedule` 方法中。為了方便你開始，該方法已包含一個簡單的範例。

### 啟動排程器

使用排程器時，只需要把下列 Cron 項目加進伺服器中即可。如果你不知道如何在伺服器中加入 Cron 項目，可以考慮使用類似 [Laravel Forge](https://forge.laravel.com) 可以管理 Cron 項目的服務：

    * * * * * php /path-to-your-project/artisan schedule:run >> /dev/null 2>&1

此 Cron 將於每分鐘呼叫 Laravel 指令排程器，開始執行 `schedule:run` 指令時，Laravel 會衡量你排定的任務並執行預定任務。

<a name="defining-schedules"></a>
## 定義排程

你可以將所有排程任務定義在 `App\Console\Kernel` 類別的 `schedule `方法中。一開始，讓我們看一個任務的排程範例。此範例中，我們將排定一個在午夜被呼叫的`閉包`。該`閉包`將執行清除某個資料表的資料庫查詢：

    <?php

    namespace App\Console;

    use DB;
    use Illuminate\Console\Scheduling\Schedule;
    use Illuminate\Foundation\Console\Kernel as ConsoleKernel;

    class Kernel extends ConsoleKernel
    {
        /**
         * 應用程式提供的 Artisan 指令。
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

<a name="scheduling-artisan-commands"></a>
### 把 Artisan 指令加入排程

除了呼叫閉包以外，也可以把 [Artisan 指令](/laravel_tw/docs/5.5/artisan) 和作業系統指令加入排程。例如，可以使用 `command` 方法來使用指令名稱或類別的 Artisan 指令加入排程：

    $schedule->command('emails:send --force')->daily();

    $schedule->command(EmailsCommand::class, ['--force'])->daily();

<a name="scheduling-queued-jobs"></a>
### 把隊列任務加入排程

`job` 方法可以用來把 [隊列任務](/laravel_tw/docs/5.5/queues) 加入排程。此方法提供提供一個方便的方式來把任務加入排程，而不是使用 `call` 方法來手動建立將任務加入隊列的閉包：

    $schedule->job(new Heartbeat)->everyFiveMinutes();

<a name="scheduling-shell-commands"></a>
### 把 Shell 指令加入排程

`exec` 方法可以用來對操作系統發出指定：

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
`->everyFifteenMinutes();`  |  於每十五分鐘執行該任務
`->everyThirtyMinutes();`  |  於每三十分鐘執行該任務
`->hourly();`  |  於每小時執行該任務
`->hourlyAt(17);`  |  於每小時的 17 分鐘後執行該任務
`->daily();`  |  於每天午夜執行該任務
`->dailyAt('13:00');`  |  於每天 13:00 執行該任務
`->twiceDaily(1, 13);`  |  於每天 1:00 及 13:00 執行該任務
`->weekly();`  |  於每週執行該任務
`->monthly();`  |  於每月執行該任務
`->monthlyOn(4, '15:00');`  |  於每月 4 號的 15:00 執行該任務
`->quarterly();` |  於每季執行該任務
`->yearly();`  |  於每年執行該任務
`->timezone('America/New_York');` | 設定時區

這些方法可以合併其他限制條件，藉以產生更精細的排程，如在一週的特定幾天執行排程。舉個例子，排定一個每週一的排程：

    // 在每個禮拜一的 13:00 執行一次...
    $schedule->call(function () {
        //
    })->weekly()->mondays()->at('13:00');

    // 在平日的 8:00 到 17:00 間每小時執行一次...
    $schedule->command('foo')
              ->weekdays()
              ->hourly()
              ->timezone('America/Chicago')
              ->between('8:00', '17:00');

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
`->between($start, $end);`  |  限制任務在 start 和 end 時間區間執行
`->when(Closure);`  |  限制任務基於一個為真驗證

#### 時間區間限制

`between` 方法可以用一天當中的時間區間來限制任務執行：

    $schedule->command('reminders:send')
                        ->hourly()
                        ->between('7:00', '22:00');

同樣 `unlessBetween` 方法可以限制在一段時間中不執行任務：

    $schedule->command('reminders:send')
                        ->hourly()
                        ->unlessBetween('23:00', '4:00');

#### 為真驗證限制

`when` 方法可以基於給定的為真驗證的執行結果來限制任務執行與否。換句話說，如果給定的`閉包`回傳 `true`，這個任務只要沒有其他的限制條件就會持續執行：

    $schedule->command('emails:send')->daily()->when(function () {
        return true;
    });

`skip` 方法可以視為跟 `when` 相反。如果 `skip` 方法回傳 `true`，排程任務就不會被執行：

    $schedule->command('emails:send')->daily()->skip(function () {
        return true;
    });

鏈結使用 `when` 方法時，排定的指令只有在所有的 `when` 條件回傳 `true` 的時候才執行。

<a name="preventing-task-overlaps"></a>
### 避免任務重複

預設情況下，即使之前相同的任務實例尚未結束，排定的任務仍會執行。為了避免這個問題，可以使用 `withoutOverlapping` 方法：

    $schedule->command('emails:send')->withoutOverlapping();

在這個範例，如果不在執行中，`emails:send` [Artisan 指令](/laravel_tw/docs/5.5/artisan) 將於每分鐘執行。當你有些任務的執行時間有大幅變化，且很難預測所需的時間時，`withoutOverlapping` 方法特別有幫助。

<a name="maintenance-mode"></a>
### 維護模式

Laravel 的排程任務在 [維護模式](/laravel_tw/docs/5.5/configuration#maintenance-mode) 下不會被執行，因為我們不希望你的任務會干擾任何伺服器上尚未完成的維護。然而，如果你想要強制任務在維護模式下仍會執行，可以使用 `evenInMaintenanceMode` 方法：

    $schedule->command('emails:send')->evenInMaintenanceMode();

<a name="task-output"></a>
## 任務輸出

Laravel 排程器為任務排程輸出提供許多方便的方法。首先，透過 `sendOutputTo` 方法可以發送輸出到一個檔案做為後續檢查：

    $schedule->command('emails:send')
             ->daily()
             ->sendOutputTo($filePath);

如果想將輸出附加到給定的檔案，可以使用 `appendOutputTo` 方法：

    $schedule->command('emails:send')
             ->daily()
             ->appendOutputTo($filePath);

透過 `emailOutputTo` 方法，你可以把輸出送到你所選的電子郵件。在將寄出任務輸出之前，你需要先設定 Laravel 的[電子郵件服務](/laravel_tw/docs/5.5/mail)：

    $schedule->command('foo')
             ->daily()
             ->sendOutputTo($filePath)
             ->emailOutputTo('foo@example.com');

> {note} `emailOutputTo`、`sendOutputTo` 和 `appendOutputTo` 方法只適用於 `command` 方法，並且不支援 `call` 方法。

<a name="task-hooks"></a>
## 任務掛勾

透過 `before` 與 `after` 方法，你能讓特定的代碼在任務完成之前及之後執行：

    $schedule->command('emails:send')
             ->daily()
             ->before(function () {
                 // 任務即將開始...
             })
             ->after(function () {
                 // 任務已完成...
             });

#### Ping URL

透過 `pingBefore` 與 `thenPing` 方法，排程器能自動的在一個任務完成之前或之後 ping 一個特定的 URL。此方法在你排定的任務進行或完成時，能有效的通知一個外部服務，例如 [Laravel Envoyer](https://envoyer.io)：

    $schedule->command('emails:send')
             ->daily()
             ->pingBefore($url)
             ->thenPing($url);

使用 `pingBefore($url)` 或 `thenPing($url)` 功能需要 Guzzle HTTP 函式庫。你可以用 Composer 套件管理器把 Guzzle 加到你的專案：

    composer require guzzlehttp/guzzle
