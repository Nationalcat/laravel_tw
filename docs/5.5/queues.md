---
layout: post
title: queues
tag: 5.5
---
# 隊列

- [簡介](#introduction)
    - [連接與隊列的比較](#connections-vs-queues)
    - [隊列驅動基本要求](#driver-prerequisites)
- [建立任務](#creating-jobs)
    - [產生任務類別](#generating-job-classes)
    - [類別結構](#class-structure)
- [執行任務](#dispatching-jobs)
    - [延遲執行](#delayed-dispatching)
    - [隊列任務鏈](#job-chaining)
    - [自訂隊列與連接](#customizing-the-queue-and-connection)
    - [指定最大任務嘗試次數 / 逾時](#max-job-attempts-and-timeout)
    - [限制執行比例](#rate-limiting)
    - [錯誤處理](#error-handling)
- [執行 Queue Worker](#running-the-queue-worker)
    - [隊列優先權](#queue-priorities)
    - [Queue Workers 與部署](#queue-workers-and-deployment)
    - [任務到期與逾時](#job-expirations-and-timeouts)
- [設定 Supervisor](#supervisor-configuration)
- [處理失敗的任務](#dealing-with-failed-jobs)
    - [清理失敗的任務](#cleaning-up-after-failed-jobs)
    - [任務處理失敗事件](#failed-job-events)
    - [重試失敗的任務](#retrying-failed-jobs)
- [任務事件](#job-events)

<a name="introduction"></a>
## 簡介

> {tip} Laravel 目前支援了 Horizon。為 Redis 運作的隊列提供一個美觀儀表板介面的設定系統。查看完整的 [Horizon 文件](/laravel_tw/docs/5.5/horizon) 以取得更多的資訊。

Laravel 隊列為各式各樣的隊列後端服務提供了一個統一的 API，像是 Beanstalk、Amazon SQS、Redis 甚至是關聯式資料庫。隊列允許你抽離需要花較多時間處理的任務，例如經過一段時間後寄送一封 email。抽離這些較為耗時的任務能夠明顯地為你的網站應用程式加快處理每一個網頁請求。

隊列的設定檔位於 `config/queue.php`。在這個檔案內你可以馬上找到隊列連接的相關驅動設定，包含框架、資料庫、[Beanstalkd](https://kr.github.io/beanstalkd/)、[Amazon SQS](https://aws.amazon.com/sqs/)、[Redis](https://redis.io) 和以本機同步驅動執行任務的設定。使用 `null` 隊列驅動則可以很容易的丟棄所有隊列任務。

<a name="connections-vs-queues"></a>
### 連接與隊列

在使用 Laravel 隊列之前，必須先明確了解「連接」與「隊列」之間的區別。在 `config/queue.php` 設定檔中有一個 `connections` 的設定選項，這個選項定義了連接隊列的後端服務，像是 Amazon SQS、Beanstalk 或是 Redis。每一個定義的「連接、，可以擁有多個不同的「隊列」，隊列內有許多的隊列任務，堆疊起來。

注意在 `queue` 設定檔中每個連接設定範例的 `queue` 屬性。將任何隊列任務送至該連接時，所有的隊列任務預設都會送到這個隊列內。換句話說，如果妳執行一個隊列任務時沒有指定任何的隊列，該隊列任務預設就會放置在設定檔裡 `queue` 屬性內預設定義的隊列內：

    // 這個隊列任務會被送至預設的隊列...
    Job::dispatch();

    // 這個隊列任務會被送至 "email" 隊列...
    Job::dispatch()->onQueue('emails');

一些應用程序可能無法滿足只使用單一個隊列，會需要推送任務至多個隊列內。針對應用程序需要設計哪些任務必須先執行或是切割任務時，推送任務至多個隊列的功能就特別好用。Laravel queue worker 允許你指定不同隊列執行任務的優先權。例如，若你推送任務至 `high` 隊列，你可以在執行 Queue worker 時給予較高的處理順序：

    php artisan queue:work --queue=high,default

<a name="driver-prerequisites"></a>
### 隊列驅動基本要求

#### 資料庫

若要使用 `database` 作為隊列驅動，你必須先建置好一個資料表用來放置任務。你可以使用 Artisan 指令 `queue:table` 來產生這個資料表。當資料表遷移成功被建立後，你可以使用 `migrate` 指令進行資料庫的資料表遷移更新：

    php artisan queue:table

    php artisan migrate

#### Redis

若要使 `redis` 作為隊列驅動，你必須在 `config/database.php` 設定檔內設定你的 Redis 資料庫連接。

若你的 Redis 隊列連接使用 Redis 叢集，你的隊列名稱必須包含 [key hash tag](https://redis.io/topics/cluster-spec#keys-hash-tags)。這個選項是必須的，用來確保指定隊列的所有 Redis keys 被放置在同一個 hash slot:

    'redis' => [
        'driver' => 'redis',
        'connection' => 'default',
        'queue' => '{default}',
        'retry_after' => 90,
    ],

#### 其他的隊列驅動基本要求

針對列出的隊列驅動，必須安裝以下對應的相依套件：

<div class="content-list" markdown="1">
- Amazon SQS: `aws/aws-sdk-php ~3.0`
- Beanstalkd: `pda/pheanstalk ~3.0`
- Redis: `predis/predis ~1.0`
</div>

<a name="creating-jobs"></a>
## 建立任務

<a name="generating-job-classes"></a>
### 產生任務類別

應用程式所有可放入隊列中執行的任務都被存放在 `app/Jobs` 目錄。如果 `app/Jobs` 目錄不存在，執行 Artisan 指令 `make:job` 同時會建立該目錄，你可以使用 Artisan CLI 產生一個新的隊列任務：

    php artisan make:job ProcessPodcast

產生的任務類別會實作 `Illuminate\Contracts\Queue\ShouldQueue` 介面，意味著 Laravel 執行該任務時會將該任務類別以非同步的方式推送至隊列。

<a name="class-structure"></a>
### 類別結構

任務類別的結構非常簡單，通常會包含一個 `handle` 方法，該方法會在任務被隊列執行時呼叫。為了理解，讓我們看一下任務類別的範例。在這個範例中，我們假裝我們管理一個公開的推播服務，該服務需要在公開推播時處理上傳的播放檔案：

    <?php

    namespace App\Jobs;

    use App\Podcast;
    use App\AudioProcessor;
    use Illuminate\Bus\Queueable;
    use Illuminate\Queue\SerializesModels;
    use Illuminate\Queue\InteractsWithQueue;
    use Illuminate\Contracts\Queue\ShouldQueue;
    use Illuminate\Foundation\Bus\Dispatchable;

    class ProcessPodcast implements ShouldQueue
    {
        use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

        protected $podcast;

        /**
         * 產生一個 Job 實例
         *
         * @param  Podcast  $podcast
         * @return void
         */
        public function __construct(Podcast $podcast)
        {
            $this->podcast = $podcast;
        }

        /**
         * 執行任務
         *
         * @param  AudioProcessor  $processor
         * @return void
         */
        public function handle(AudioProcessor $processor)
        {
            // 處理上傳的推播...
        }
    }

在這個範例中，我們能夠直接傳遞一個 [Eloquent 模型](/laravel_tw/docs/5.5/eloquent) 至隊列任務的建構子。因為任務類別使用 `SerializesModels` trait，當任務被執行時， Eloquent 模型會優雅的被序列話化和解序列化。如果你的隊列任務在建構子接收一個 Eloquent 模型，只有模型的識別子(identifier)會被序列化被放進隊列中。當任務真正被處理時，隊列系統會自動的重新從資料庫獲取完整的模型實例。整個過程對於你的應用程式是完全透明的，避免在序列化整個 Eloquent 模型實例時出現問題。

`handle` 方法會在任務執行是被呼叫。注意我們能夠在 `handle` 方法傳遞的參數宣告依賴類別，Laravel 提供 [服務容器](/laravel_tw/docs/5.5/container) 能夠自動的注入這些依賴類別。

> {note} 二進位資料，例如原始圖形內容，在傳遞至隊列任務時需要使用 `base64_encode` 函式進行傳遞。否則，該隊列任務在被放置進隊列時可能無法正確的序列化解析成 JSON 格式。

<a name="dispatching-jobs"></a>
## 執行任務 

當你撰寫完任務類別後，你可以呼叫類別內的 `dispatch` 方法執行任務。`dispatch` 方法的參數會被傳遞至任務類別的建構子中：

    <?php

    namespace App\Http\Controllers;

    use App\Jobs\ProcessPodcast;
    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;

    class PodcastController extends Controller
    {
        /**
         * 儲存一個新的推播
         *
         * @param  Request  $request
         * @return Response
         */
        public function store(Request $request)
        {
            // 建立推播...

            ProcessPodcast::dispatch($podcast);
        }
    }

<a name="delayed-dispatching"></a>
### 延遲執行

如果你想要延遲執行一個隊列任務，可以在執行隊列任務時使用 `delay` 方法。舉例來說，指定一個任務在十分鐘後執行：

    <?php

    namespace App\Http\Controllers;

    use App\Jobs\ProcessPodcast;
    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;

    class PodcastController extends Controller
    {
        /**
         * 儲存新的推播
         *
         * @param  Request  $request
         * @return Response
         */
        public function store(Request $request)
        {
            // 新增推播 ...

            ProcessPodcast::dispatch($podcast)
                    ->delay(now()->addMinutes(10));
        }
    }

> {note} The Amazon SQS queue service has a maximum delay time of 15 minutes.

<a name="job-chaining"></a>
### 隊列任務鏈

隊列任務鏈允許你指定一系列的隊列任務，並且依序的執行這些任務。如果隊列任務鏈中的其中一個工作失敗了，整個任務不會繼續被執行。你可以在任何被執行的隊列任務呼叫`withChain` 方法執行任務鏈：

    ProcessPodcast::withChain([
        new OptimizePodcast,
        new ReleasePodcast
    ])->dispatch();

<a name="customizing-the-queue-and-connection"></a>
### 自訂隊列與連結

#### 在特定的隊列中執行

透過推送並任務至不同的隊列，你可以對隊列任務進行「分類」，讓多個隊列任務之間分配不同的執行優先順序和 Queue Worker 的執行數量。記住，這樣的行為並不會將對列任務推送到隊列設定檔內定義的「連線」。而只會將隊列任務在單一連線內推送到指定的隊列。在執行隊列任務時呼叫 `onQueue` 方法能指定隊列：

    <?php

    namespace App\Http\Controllers;

    use App\Jobs\ProcessPodcast;
    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;

    class PodcastController extends Controller
    {
        /**
         * 儲存一個新推播
         *
         * @param  Request  $request
         * @return Response
         */
        public function store(Request $request)
        {
            // 建立推播...

            ProcessPodcast::dispatch($podcast)->onQueue('processing');
        }
    }

#### 在特定的連線中執行

如果你正嘗試使用多個隊列連線，你可以指定要將對列任務推送到哪個連線。在執行隊列任務時呼叫 `onConnection` 方法能指定隊列任務推送至特定的連線：

    <?php

    namespace App\Http\Controllers;

    use App\Jobs\ProcessPodcast;
    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;

    class PodcastController extends Controller
    {
        /**
         * 儲存一個新推播
         *
         * @param  Request  $request
         * @return Response
         */
        public function store(Request $request)
        {
            // 建立推播...

            ProcessPodcast::dispatch($podcast)->onConnection('sqs');
        }
    }

當然,你可以串連 `onConnection` 和 `onQueue` 方法指定任務的連線和隊列：

    ProcessPodcast::dispatch($podcast)
                  ->onConnection('sqs')
                  ->onQueue('processing');

<a name="max-job-attempts-and-timeout"></a>
### 指定最大任務嘗試次數 / 逾時

#### 最大嘗試次數 

指定最大任務嘗試次數的一種方法是透過在執行 Artisan 指令時啟用 `--tries` 選項：

    php artisan queue:work --tries=3

不過，更精細的方式則是在任務類別內定義最大的嘗試次數。如果任務類別內指定了最大嘗試次數，在執行上述 Artisan 命令時類別內指定的最大次數優先於命令列指定的次數：

    <?php

    namespace App\Jobs;

    class ProcessPodcast implements ShouldQueue
    {
        /**
         * 隊列任務最大的嘗試次數
         *
         * @var int
         */
        public $tries = 5;
    }

<a name="time-based-attempts"></a>
#### 基於計時的嘗試

作為替代單純定義任務失敗時的最大嘗試次數的另一個方式，你可以定義任務的逾時時間，這讓任務可以在指定的時間內可以被重試無數次。在隊列任務類別內新增 `retryUntil` 方法來定義任務的最大逾時時間：

    /**
     * 預估該隊列任務多久會逾時
     *
     * @return \DateTime
     */
    public function retryUntil()
    {
        return now()->addSeconds(5);
    }

> {tip} 你也可以在你的隊列事件監聽類別內定義 `retryUntil` 方法。

#### 逾時

> {note} `timeout` 功能在 PHP 7.1+ 版本及 `pcntl` PHP 擴充元件均已優化。

同理，執行 Artisan 命令時使用 `--timeout` 選項能夠指定任務的最大秒數：

    php artisan queue:work --timeout=30

然而，你或許也想要在任務類別內定義任務允許被執行的最大秒數，如果在任務內指定了逾時，其優先權高於在 Artisan 命令列指定的秒數：

    <?php

    namespace App\Jobs;

    class ProcessPodcast implements ShouldQueue
    {
        /**
         * 任務在多久的時間內允許執行的最大秒數
         *
         * @var int
         */
        public $timeout = 120;
    }

<a name="rate-limiting"></a>
### 限制執行比例

> {note} 使用這個功能的前提是你的應用程式有根 [Redis 伺服器](/laravel_tw/docs/5.5/redis) 進行互動

如果你的應用程式與 Redis 有所互動，你可以藉由時間或是併發次數限制隊列任務的執行。這個功能能夠協助你的隊列任務在與一些具備請求限制的 APIs 互動進行比例限制，舉例來說，使用 `throttle` 方法能夠限制該類型的任務只能在每 60 秒內執行 10 次。若無法鎖定，通常會將任務釋放回隊列以便稍後進行重試：

    Redis::throttle('key')->allow(10)->every(60)->then(function () {
        // 任務邏輯...
    }, function () {
        // 無法獲得鎖定...

        return $this->release(10);
    });

> {tip} 在上面的例子中， `key` 可能是任何的獨一無二的字串用來識別你想要限制執行比例的任務類型。比如，你可能會想要以類別名稱和對應操作 Eloquent 類別的 ID 作為鍵值。

另外，你可以指定同時處理隊列任務的 Queue worker 最大數量，這在限制隊列任務只能一次存取單一資源時特別有用。舉例來說，使用 `funnel` 方法能夠限制該類型的任務在同一時間內只能一次被單一個 Queue worker 處理：

    Redis::funnel('key')->limit(1)->then(function () {
        // 任務邏輯...
    }, function () {
        // 無法獲得鎖定...

        return $this->release(10);
    });

> {tip} 在限制執行比利時，很難得知任務真正所需的最大嘗試次數，因此，同時限制執行比例和[逾時嘗試](#time-based-attempts)十分有效。

<a name="error-handling"></a>
### 錯誤處理

如果執行隊列任務時一個例外狀況被拋出，該任務會自動的被釋放回隊列並且重新嘗試執行。該任務會在應用程式允許的最大嘗試次數內繼續地被執行和釋放回隊列，可藉由 `queue:work` Artisan 命令的 `--tries` 選項定義任務的最大嘗試次數。另外，也可以在任務類別內定義最大嘗試次數，更多關於執行隊列 Queue worker [請詳閱下列的資訊](#running-the-queue-worker)。

<a name="running-the-queue-worker"></a>
## 執行 Queue Worker

Laravel 內建了 Queue worker 能夠處理被推送進隊列內的新任務。你可以使用 `queue:work` Artisan 命令來執行 Queue worker。注意一旦 `queue:work` 被啟動，會持續的執行直到你手動停止或是你終端機：

    php artisan queue:work

> {tip} 為了確保 `queue:work` 程序持續地在背景執行，你可以使用一些方法來監看這個程序，像是 [Supervisor](#supervisor-configuration) 來確保 Queue worker 不會在執行過程中被終止。

記住，Queue workers 是常駐程序並且會將應用程式的狀態儲存在記憶體空間中。一旦啟動後，當你的程式碼有變更時並不會被更新，所以，在部署階段，記得[重啟你的 Queue worker](#queue-workers-and-deployment).

#### 處理單一任務

`--once` 選項可以被用來指派 Queue worker 僅能從隊列中處理單一個任務：

    php artisan queue:work --once

#### 指定連線 & 隊列

你或許會想要指定 Queue worker 處理的連線對象，你可以將`config/queue.php` 設定檔內定義的其中一個連線名稱傳遞至 `work` 命令：

    php artisan queue:work redis

你甚至可以進一步自訂 Queue worker 只對該連線只處理特定的隊列。舉例來說，如果你所有的 Email 任務均在 `redis` 連線裡的 `emails` 隊列中，你可以使用以下命令來啟動一個 Queue worker 只處理該隊列的工作：

    php artisan queue:work redis --queue=emails

#### 資源分配考量

Queue workers 守護進程並不會在每個任務被執行前 "重啟" 整個框架。因此，你應該在任何任務結束後將任何高用量的資源釋放。比如，如果你正在使用 GD 函式庫處理影像，你應該在影像處理完畢後使用 `imagedestroy` 來釋放你的記憶體。

<a name="queue-priorities"></a>
### 隊列優先權

有時候你可能會想要決定每個隊列處理的優先順序，舉例來說，在你的 `config/queue.php` 檔案中你可以為 `redis` 連線將預設的 `queue` 定義為 `low`。不過，你偶爾會想要推送任務至 `high` 高優先隊列，像是：

    dispatch((new Job)->onQueue('high'));

為了啟用一個 Queue worker 並確保所有名為 `high` 的隊列能夠在名為 `low` 的隊列任務被處理前有較高的優先順序，必須以逗點分隔的方式依順序將隊列名稱傳遞至 `work` 命令：

    php artisan queue:work --queue=high,low

<a name="queue-workers-and-deployment"></a>
### Queue Workers & 部署

因為 Queue worker 屬於常駐形態的程序，並不會因為你的程式碼變動而自我重啟。所以，一個使用 Queue worker 的應用程序最簡單的部署方式就是在部署階段重啟 Queue worker。你可以藉由 `queue:restart` 命令優雅的重啟 Queue worker：

    php artisan queue:restart

該命令會指示所有的 Queue worker 等到目前處理的任務結束後優雅的 "終止" ，並且不會遺失掉任何一個隊列任務。因為 Queue worker 會等到 `queue:restart` 指令被執行後才會終止，你可以藉由 [Supervisor](#supervisor-configuration) 這樣的程序管理來自動重啟你的 Queue worker 。

> {tip} 隊列使用[快取](/laravel_tw/docs/5.5/cache)儲存重啟訊號，所以你必須在使用這個功能前確保應用程式的快取驅動正確的被設定

<a name="job-expirations-and-timeouts"></a>
### 任務到期與逾時

#### 任務逾期

在 `config/queue.php` 設定檔內，每個隊列連線都定義一個 `retry_after` 選項。這個選項指定了該隊列連線任務被處理後應等待的重試秒數，舉例來說，如果 `retry_after` 選項的值被設定為 `90`，該任務會在被處理後等待 90 秒再被釋放回隊列中而不是被刪除。通常，你應該合理地設定 `retry_after` 最大值使你的任務能夠妥善地被處理。

> {note} 唯一不適用 `retry_after` 選項設定的隊列連線為 Amazon SQS，SQS 本身具有我重試的機制，並且會基於 AWS 管理主控台內的 [Default Visibility Timeout](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/AboutVT.html) 設定進行自我重試。

#### Queue Worker 逾時

`queue:work` Artisan 命令具備 `--timeout` 選項，`--timeout` 選項指定了 Laravel 隊列主要程序會在 Queue worker 強制殺掉該處理程序前決定一個任務程序需要花多長時間等待。有時候子程序會因為種種因素 "凍結" ，像是呼叫外部 HTTP 連線沒有回應。`--timeout` 選項會在超出指定執行時間限制時移除被凍結的程序：

    php artisan queue:work --timeout=60

`retry_after` 設定選項和 `--timeout` CLI 選項有所不同，但是同時使用能確保隊列任務不會遺失，並且執行成功的隊列任務只能被執行一次。

> {note} `--timeout` 值應該至少比 `retry_after` 選項所設定的值稍微短些，這能確保一個 Queue worker 處理對應的任務時能夠在任務程序被重試前真正被終止。如果你的 `--timeout` 選項比你所設定的的 `retry_after` 設定值來得長，你的隊列任務可能會被處理兩次。

#### Queue Worker 閒置間隔

當一個任務被放置至隊列中，Queue worker 會不間斷的在隊列間持續處理任務。然而，`sleep` 選項設定了 Queue worker 在沒有新的任務被推送進隊列時應該「閒置」多久，處於閒置狀態時，Queue worker 並不會處理任何閒置期間新進的隊列任務，直到 Queue worker 回復工作狀態時才會被處理。

    php artisan queue:work --sleep=3

<a name="supervisor-configuration"></a>
## 設定 Supervisor

#### 安裝 Supervisor

Supervisor 是一個 Linux 系統內的程序監看工具，同時能夠自動重啟失敗的 `queue:work` 程序。在 Ubuntu 發行版中你可以使用以下指令安裝 Supervisor：

    sudo apt-get install supervisor

> {tip} 設定 Supervisor 聽起來很麻煩嗎？可以使用 [Laravel Forge](https://forge.laravel.com)，內建已經自動地為你的 Laravel 專安安裝並設定好 Supervisor。

#### Supervisor 設定檔

Supervisor 設定檔通常會位於 `/etc/supervisor/conf.d` 目錄。在這個目錄內，你可以建立不限數量的設定檔案來引導 Supervisor 如何監看你的程序。舉例來說，建立一個 `laravel-worker.conf` 檔案用於啟動並監看 `queue:work` 程序：

    [program:laravel-worker]
    process_name=%(program_name)s_%(process_num)02d
    command=php /home/forge/app.com/artisan queue:work sqs --sleep=3 --tries=3
    autostart=true
    autorestart=true
    user=forge
    numprocs=8
    redirect_stderr=true
    stdout_logfile=/home/forge/app.com/worker.log

在這個範例，`numprocs` 指令會引導 Supervisor 啟動並監看 8 個 `queue:work` 程序，並自動重啟失敗的程序。當然，你應該更改 `command` 選項內部分 `queue:work sqs` 設定，以達到你預期的隊列連線設定。

#### 啟動 Supervisor

一旦設定檔案被建立，你可以使用以下指令更新及啟動 Supervisor：

    sudo supervisorctl reread

    sudo supervisorctl update

    sudo supervisorctl start laravel-worker:*

更多訊息，詳見 [Supervisor 參考文件](http://supervisord.org/index.html)。

<a name="dealing-with-failed-jobs"></a>
## 處理失敗的任務

有時候隊列中的任務執行失敗，別擔心，事情發生總有不如預期。Laravel 內建了方便的方法能夠指定隊列任務的最大嘗試次數。當一個隊列任務超過最大嘗試次數時，會新增至 `failed_jobs` 資料表。為了透過建立資料庫遷移產生 `failed_jobs` 資料表，你可以使用 `queue:failed-table` 指令：

    php artisan queue:failed-table

    php artisan migrate

接下來，執行 [隊列 worker](#running-the-queue-worker)，你應該在執行 `queue:work` 指令時指定 `--tries` 選項指定最大的錯誤嘗試次數。如果 `--tries` 選項沒有被指定，失敗的隊列任務會不斷的進行錯誤重試：

    php artisan queue:work redis --tries=3

<a name="cleaning-up-after-failed-jobs"></a>
### 清理失敗的任務

你可以直接地在你的任務類別中定義一個 `failed` 方法，能夠在任務發生處理失敗時執行特定的清理動作。這也是一個在任務類別內能處理警告寄送給你的使用者或是還原任何隊列任務操作的絕佳位置。`Exception` 類別使得隊列任務在發生處理失敗時能夠呼叫 `failed` 方法：

    <?php

    namespace App\Jobs;

    use Exception;
    use App\Podcast;
    use App\AudioProcessor;
    use Illuminate\Bus\Queueable;
    use Illuminate\Queue\SerializesModels;
    use Illuminate\Queue\InteractsWithQueue;
    use Illuminate\Contracts\Queue\ShouldQueue;

    class ProcessPodcast implements ShouldQueue
    {
        use InteractsWithQueue, Queueable, SerializesModels;

        protected $podcast;

        /**
         * 建立一個新的任務實例
         *
         * @param  Podcast  $podcast
         * @return void
         */
        public function __construct(Podcast $podcast)
        {
            $this->podcast = $podcast;
        }

        /**
         * 執行任務
         *
         * @param  AudioProcessor  $processor
         * @return void
         */
        public function handle(AudioProcessor $processor)
        {
            // 處理上傳的推播...
        }

        /**
         * 任務處理失敗
         *
         * @param  Exception  $exception
         * @return void
         */
        public function failed(Exception $exception)
        {
            // 給使用者寄送處理失敗通知等...
        }
    }

<a name="failed-job-events"></a>
### 任務處理失敗事件

若你想要註冊一個在任務失敗時會被呼叫的事件，你可以使用 `Queue::failing` 方法，這個事件是一個非常好的方式能夠藉由 Email 或是 [HipChat](https://www.hipchat.com) 來通知你的團隊。舉例來說，我們會想要從 Laravel 內建的 `AppServiceProvider` 類別內，利用這個事件內附加一個回呼函式：

    <?php

    namespace App\Providers;

    use Illuminate\Support\Facades\Queue;
    use Illuminate\Queue\Events\JobFailed;
    use Illuminate\Support\ServiceProvider;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * 引導任何應用服務
         *
         * @return void
         */
        public function boot()
        {
            Queue::failing(function (JobFailed $event) {
                // $event->connectionName
                // $event->job
                // $event->exception
            });
        }

        /**
         * 註冊該服務提供者
         *
         * @return void
         */
        public function register()
        {
            //
        }
    }

<a name="retrying-failed-jobs"></a>
### 重試失敗的任務

利用 `queue:failed` Artisan 命令能列出所有被儲存至 `failed_jobs` 資料表內的失敗任務：

    php artisan queue:failed

`queue:failed` 命令會列任務 ID、連線名稱、隊列名稱和失敗時間。隊列 ID 在重試任務時會被使用，例如，使用以下命令用以重試一個 ID 為 `5` 的任務：

    php artisan queue:retry 5

執行 `queue:retry` 命令並配合 `all` 作為 ID 選項，重試所有的失敗任務：

    php artisan queue:retry all

假如你想要刪除一個失敗的任務，你可以使用 `queue:forget` 命令：

    php artisan queue:forget 5

使用 `queue:flush` 命令能清除所有失敗的任務：

    php artisan queue:flush

<a name="job-events"></a>
## 任務事件

在 `Queue` [facade](/laravel_tw/docs/5.5/facades) 使用 `before` 及 `after` 方法，你能指定回呼(callbacks)函式，在執行處理該隊列任務前或後執行對應的動作。這些回呼函式能夠完美的執行額外的事件記錄或是為儀表板提供統計資訊。通常會搭配 [service provider](/laravel_tw/docs/5.5/providers) 用於呼叫這些方法。舉例來說，你可以使用 Laravel 內建的 `AppServiceProvider`:

    <?php

    namespace App\Providers;

    use Illuminate\Support\Facades\Queue;
    use Illuminate\Support\ServiceProvider;
    use Illuminate\Queue\Events\JobProcessed;
    use Illuminate\Queue\Events\JobProcessing;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * 引導應用服務
         *
         * @return void
         */
        public function boot()
        {
            Queue::before(function (JobProcessing $event) {
                // $event->connectionName
                // $event->job
                // $event->job->payload()
            });

            Queue::after(function (JobProcessed $event) {
                // $event->connectionName
                // $event->job
                // $event->job->payload()
            });
        }

        /**
         * 註冊服務提供者
         *
         * @return void
         */
        public function register()
        {
            //
        }
    }

使用 `Queue` [facade](/laravel_tw/docs/5.5/facades) 的 `looping` 方法，你能夠藉由定義回呼函式，在 worker 嘗試從隊列中獲取任務執行一些工作。舉例來說，你可以註冊一個閉包以還原前一個任務執行時拜留下的資料庫交易紀錄：

    Queue::looping(function () {
        while (DB::transactionLevel() > 0) {
            DB::rollBack();
        }
    });
