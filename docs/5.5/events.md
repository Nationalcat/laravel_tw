---
layout: post
title: events
tag: 5.5
---
# 事件

- [介紹](#introduction)
- [註冊事件與監聽器](#registering-events-and-listeners)
    - [產生事件與監聽器](#generating-events-and-listeners)
    - [手動註冊事件](#manually-registering-events)
- [定義事件](#defining-events)
- [定義監聽器](#defining-listeners)
- [隊列事件監聽器](#queued-event-listeners)
    - [手動存取隊列](#manually-accessing-the-queue)
    - [處理失敗的任務](#handling-failed-jobs)
- [調度事件](#dispatching-events)
- [事件訂閱者](#event-subscribers)
    - [撰寫事件訂閱者](#writing-event-subscribers)
    - [註冊事件訂閱者](#registering-event-subscribers)

<a name="introduction"></a>
## 介紹

Laravel 的事件提供了簡單的觀察者實作，可以讓你訂閱並監聽各種發生在你的應用程式的事件。事件類別通常會存放在 `app/Events` 目錄，而它們的監聽器則存放在 `app/Listeners`。如果你沒在你的應用程式中看到這些目錄，別擔心！因為他們會在你使用 Artisan 指令列指令來產生事件和監聽器時被建立。

事件服務作為解耦應用程式的各種方面的好方法，因為單一個事件能有多個不依賴彼此的監聽器。例如，你可能希望在每次訂單出貨得時候就發送 Slack 通知給使用者。不是將你的訂單處理程式碼耦合到你的 Slack 通知的程式碼中，而是你能簡單的觸發 `OrderShipped` 事件，讓監聽器可以來接收並轉換成一個 Slack 通知。

<a name="registering-events-and-listeners"></a>
## 註冊事件與監聽器

Laravel 應用程式中引入的 `EventServiceProvider` 提供一個便捷的地方來註冊應用程式的所有事件監聽器。`listen` 屬性包含一組所有事件（鍵）和它們的監聽器（值）的陣列。當然，你可以盡量新增多個事件到這組陣列來滿足應用程式的需求。例如，讓我們新增 `OrderShipped` 事件：

    /**
     * 應用程式的事件監聽器映射。
     *
     * @var array
     */
    protected $listen = [
        'App\Events\OrderShipped' => [
            'App\Listeners\SendShipmentNotification',
        ],
    ];

<a name="generating-events-and-listeners"></a>
### 產生事件與監聽器

當然，為每個事件和監聽器手動建立檔案是件麻煩的事情。不過，你可以簡單的新增監聽器和事件到 `EventServiceProvider`，並使用 `event:generate` 指令。這個指令會產生在 `EventServiceProvider` 被列出的任何事件或監聽器。當然，已存在的事件和監聽器就不會被動到：

    php artisan event:generate

<a name="manually-registering-events"></a>
### 手動註冊事件

通常事件會透過 `EventServiceProvider` 的 `$listen`陣列來註冊。不過，你也可以在 `EventServiceProvider` 的 `boot` 方法手動以閉包的方式註冊事件：

    /**
     * 為你的應用程式註冊其他任何事件。
     *
     * @return void
     */
    public function boot()
    {
        parent::boot();

        Event::listen('event.name', function ($foo, $bar) {
            //
        });
    }

#### 使用萬用字元的事件監聽器

你甚至可以使用 `*` 作為萬用字元參數來註冊監聽器，可以讓你在同個監聽器上抓到多個事件。萬用字元的監聽器將接收到的名稱作為它們的第一個參數，並將整個事件資料的陣列作為第二個參數：

    Event::listen('event.*', function ($eventName, array $data) {
        //
    });

<a name="defining-events"></a>
## 定義事件

事件類別是保存與事件相關的資訊的資料容器。例如，假設我們產生的 `OrderShipped` 事件要去接收 [Eloquent ORM](/laravel_tw/docs/5.5/eloquent) 物件：

    <?php

    namespace App\Events;

    use App\Order;
    use Illuminate\Queue\SerializesModels;

    class OrderShipped
    {
        use SerializesModels;

        public $order;

        /**
         * 建立新事件實例。
         *
         * @param  Order  $order
         * @return void
         */
        public function __construct(Order $order)
        {
            $this->order = $order;
        }
    }

如你所見，這個事件類別不包含邏輯。它只是一個 `Order` 實例。如果事件物件是被 PHP 的 `serialize` 函式給序列化，那麼被事件使用的 `SerializesModels` trait 將會被優雅的序列化成任何 Eloquent 模型。

<a name="defining-listeners"></a>
## 定義監聽器

接著，讓我們看一下範例事件的監聽器。事件監聽器會在它們的 `handle` 方法接收事件實例。`event:generate` 指令會自動導入合適的事件類別並在 `handle` 方法注入事件。在 `handle` 方法中，你可以執行任何必要的操作來回應事件：

    <?php

    namespace App\Listeners;

    use App\Events\OrderShipped;

    class SendShipmentNotification
    {
        /**
         * Create the event listener.
         *
         * @return void
         */
        public function __construct()
        {
            //
        }

        /**
         * 處理事件。
         *
         * @param  OrderShipped  $event
         * @return void
         */
        public function handle(OrderShipped $event)
        {
            // 使用 $event->order 存取訂單...
        }
    }

> {tip} 事件監聽器也可以在建構子上注入任何需要的依賴。所有的事件監聽器都會透過 Laravel [服務容器](/laravel_tw/docs/5.5/container)來解析，所以依賴才會被自動注入。

#### 停止一個事件的傳播

有時候，你可能希望停止一個事件的傳播到其他的監聽器。你可以在監聽器的 `handle` 方法回傳 `false` 達到這項目的。

<a name="queued-event-listeners"></a>
## 隊列事件監聽器

使用隊列監聽器會是有助於你的隊列要執行一個要處理很久的任務，像是發送電子郵件或發出 HTTP 請求。在使用隊列監聽器之前，請先確認你的[隊列設定](/laravel_tw/docs/5.5/queues)，並在你的伺服器或本機開發環境啟動隊列監聽器。

要讓監聽器能夠被指定隊列，新增 `ShouldQueue` 介面到監聽器的類別。由 Artisan 指令的 `event:generate` 產生的監聽器會同時把這個介面導入當前的命名空間中，所以你可以馬上使用它：

    <?php

    namespace App\Listeners;

    use App\Events\OrderShipped;
    use Illuminate\Contracts\Queue\ShouldQueue;

    class SendShipmentNotification implements ShouldQueue
    {
        //
    }

當這個監聽器被一個事件呼叫時，他會被事件發送器使用 Laravel 的[隊列系統](/laravel_tw/docs/5.5/queues)自動加入隊列。如果在隊列中執行監聽器時沒導致例外，隊列任務會在完成任務後自動刪除。

#### 自訂隊列連線與隊列名稱

如果你希望事件監聽器能使用自訂隊列連線和隊列名稱，你可以在監聽器類別上定義 `$connection` 和 `$queue` 屬性：

    <?php

    namespace App\Listeners;

    use App\Events\OrderShipped;
    use Illuminate\Contracts\Queue\ShouldQueue;

    class SendShipmentNotification implements ShouldQueue
    {
        /**
         * 任務會發送到該連線名稱。
         *
         * @var string|null
         */
        public $connection = 'sqs';

        /**
         * 隊列會發送到該連線名稱。
         *
         * @var string|null
         */
        public $queue = 'listeners';
    }

<a name="manually-accessing-the-queue"></a>
### 手動存取隊列

如果你需要手動存取監聽器底層的隊列任務的 `delete` 和 `release` 方法，你可以使用 `Illuminate\Queue\InteractsWithQueue` trait 來做。這個 trait 預設會導入剛產生的監聽器中，並提供存取這些方法：

    <?php

    namespace App\Listeners;

    use App\Events\OrderShipped;
    use Illuminate\Queue\InteractsWithQueue;
    use Illuminate\Contracts\Queue\ShouldQueue;

    class SendShipmentNotification implements ShouldQueue
    {
        use InteractsWithQueue;

        /**
         * 處理事件。
         *
         * @param  \App\Events\OrderShipped  $event
         * @return void
         */
        public function handle(OrderShipped $event)
        {
            if (true) {
                $this->release(30);
            }
        }
    }

<a name="handling-failed-jobs"></a>
### 處理失敗的任務

有時隊列中的事件監聽器可能會失敗。如果隊列監聽器的嘗試執行超過了隊列器定義的最大嘗試次數，會在你的監聽器上呼叫 `failed` 方法。`failed` 方法會接收事件實例和導致失敗的例外：

    <?php

    namespace App\Listeners;

    use App\Events\OrderShipped;
    use Illuminate\Queue\InteractsWithQueue;
    use Illuminate\Contracts\Queue\ShouldQueue;

    class SendShipmentNotification implements ShouldQueue
    {
        use InteractsWithQueue;

        /**
         * 處理事件。
         *
         * @param  \App\Events\OrderShipped  $event
         * @return void
         */
        public function handle(OrderShipped $event)
        {
            //
        }

        /**
         * 處理失敗的任務
         *
         * @param  \App\Events\OrderShipped  $event
         * @param  \Exception  $exception
         * @return void
         */
        public function failed(OrderShipped $event, $exception)
        {
            //
        }
    }

<a name="dispatching-events"></a>
## 調度事件

想要調度一個事件，你可以傳入一個事件實例到 `event` 輔助函式。輔助函式會指派事件到它註冊的所有監聽器。因為 `event` 輔助函式是全域可用的，你可以在應用程式的任何地方呼叫它：

    <?php

    namespace App\Http\Controllers;

    use App\Order;
    use App\Events\OrderShipped;
    use App\Http\Controllers\Controller;

    class OrderController extends Controller
    {
        /**
         * 傳送給定的訂單。
         *
         * @param  int  $orderId
         * @return Response
         */
        public function ship($orderId)
        {
            $order = Order::findOrFail($orderId);

            // 訂單出貨的邏輯...

            event(new OrderShipped($order));
        }
    }

> {tip} 在測試的時候，它是有助於斷言某些沒有實際觸發監聽器的事件。Laravel 的[內建測試的輔助函式](/laravel_tw/docs/5.5/mocking#event-fake)將它設計得很簡單。

<a name="event-subscribers"></a>
## 事件訂閱者

<a name="writing-event-subscribers"></a>
### 撰寫事件訂閱者

事件訂閱者可以從類別中訂閱多個事件的類別，可讓你在單一類別中定義多個事件處理器。訂閱者會定義 `subscribe` 方法，該方法會被傳入事件指派器實例。你可以在給定指派器上呼叫 `listen` 方法來註冊事件監聽器：

    <?php

    namespace App\Listeners;

    class UserEventSubscriber
    {
        /**
         * 處理使用者登入事件。
         */
        public function onUserLogin($event) {}

        /**
         * 處理使用者登出事件。
         */
        public function onUserLogout($event) {}

        /**
         * 為訂閱者註冊監聽器。
         *
         * @param  Illuminate\Events\Dispatcher  $events
         */
        public function subscribe($events)
        {
            $events->listen(
                'Illuminate\Auth\Events\Login',
                'App\Listeners\UserEventSubscriber@onUserLogin'
            );

            $events->listen(
                'Illuminate\Auth\Events\Logout',
                'App\Listeners\UserEventSubscriber@onUserLogout'
            );
        }

    }

<a name="registering-event-subscribers"></a>
### 註冊事件訂閱者

撰寫訂閱者後，你就已經準備好註冊它與事件指派器。你可以使用在 `EventServiceProvider` 上的 `$subscribe` 屬性註冊訂閱者。例如，我們新增 `UserEventSubscriber` 到清單上：

    <?php

    namespace App\Providers;

    use Illuminate\Foundation\Support\Providers\EventServiceProvider as ServiceProvider;

    class EventServiceProvider extends ServiceProvider
    {
        /**
         * 應用程式的事件監聽器映射。
         *
         * @var array
         */
        protected $listen = [
            //
        ];

        /**
         * 註冊訂閱者類別。
         *
         * @var array
         */
        protected $subscribe = [
            'App\Listeners\UserEventSubscriber',
        ];
    }
