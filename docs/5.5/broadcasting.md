---
layout: post
title: broadcasting
tag: 5.5
---
# 廣播系統

- [介紹](#introduction)
    - [設定](#configuration)
    - [驅動需求](#driver-prerequisites)
- [概念簡述](#concept-overview)
    - [使用一個範例應用程式](#using-example-application)
- [定義廣播事件](#defining-broadcast-events)
    - [廣播名稱](#broadcast-name)
    - [廣播資料](#broadcast-data)
    - [廣播隊列](#broadcast-queue)
    - [廣播條件](#broadcast-conditions)
- [頻道授權](#authorizing-channels)
    - [定義授權的路由](#defining-authorization-routes)
    - [定義授權的回呼函式](#defining-authorization-callbacks)
- [廣播事件](#broadcasting-events)
    - [只廣播給其他人](#only-to-others)
- [接受廣播](#receiving-broadcasts)
    - [安裝 Laravel Echo](#installing-laravel-echo)
    - [監聽事件](#listening-for-events)
    - [退出頻道](#leaving-a-channel)
    - [命名空間](#namespaces)
- [Presence 頻道](#presence-channels)
    - [認證給 Presence 頻道](#authorizing-presence-channels)
    - [加入到 Presence 頻道](#joining-presence-channels)
    - [廣播到 Presence 頻道](#broadcasting-to-presence-channels)
- [客戶端事件](#client-events)
- [通知](#notifications)

<a name="introduction"></a>
## 介紹

在許多現代網頁應用程式中，WebSocket 被用來實作 realtime 與即時更新的使用者介面。當伺服器在更新某些資料時，會透過 WebSocket 的連線去處理使用者的訊息。相較於不斷地輪詢來取得後端資料的方式，這提供了更強大、更有效率的方式。

為了協助你建立這些類型的應用程式，Laravel 讓你可以輕鬆的廣播你的[事件](/laravel_tw/docs/5.5/events)到 WebSocket 連線。Laravel 允許你在廣播事件時，在伺服器端和客戶端的 JavaScript 應用程式之間共用相同的事件名稱。

> {tip} 再深入了解廣播之前，確認你已經讀過所有關於 Laravel 的[事件與監聽器](/laravel_tw/docs/5.5/events)的文件。

<a name="configuration"></a>
### 設定

所有關於事件廣播設定都存放在 `config/broadcasting.php` 這個設定檔。Laravel 支援幾個廣播用的驅動：[Pusher](https://pusher.com)、[Redis](/laravel_tw/docs/5.5/redis) 和用來本機開發與除錯的 `log`。此外，你可以使用 `null` 來禁用所有的廣播器。在 `config/broadcasting.php` 中，每個驅動的設定都有附上設定的範例供你參考。

#### 廣播的服務提供者

在廣播任何事件前，首先你需要註冊 `App\Providers\BroadcastServiceProvider`。在全新的 Laravel 應用程式中，你只需要在 `config/app.php` 設定檔中找到 `providers` 陣列，並取消對它們的註解。這個提供者可以讓你註冊廣播授權路由和回呼函式。

#### CSRF Token

[Laravel Echo](#installing-laravel-echo) 會需要存取當前 session 的 CSRF token 。你應該確認你的 `head` HTML 標籤裡是否有放入 `meta` 標籤來設定 CSRF token：

    <meta name="csrf-token" content="{% raw %} {{ csrf_token() }} {% endraw %}">

<a name="driver-prerequisites"></a>
### 驅動需求

#### Pusher

如果你透過 [Pusher](https://pusher.com) 來廣播事件，你應該使用 Composer 安裝 Pusher PHP SDK：

    composer require pusher/pusher-php-server "~3.0"

接下來，你需要設定你的 Pusher 憑證到 `config/broadcasting.php` 這個設定檔。這個文件已寫好 Pusher 設定範例，你只需要去修改預設的 Pusher 金鑰、密碼和應用程式 ID 。 `config/broadcasting.php` 裡面的 `Pusher` 設定允許你使用 `options` 來加入 Puhser 支援的額外功能選項，像是下面寫的：

    'options' => [
        'cluster' => 'eu',
        'encrypted' => true
    ],

在同時使用 Pushser 和 [Laravel Echo](#installing-laravel-echo) 的時候，你應該在 `resources/assets/js/bootstrap.js` 檔案中實體化 Echo 實例的時候，將指定 Pusher 作為所需的廣播器：

    import Echo from "laravel-echo"

    window.Pusher = require('pusher-js');

    window.Echo = new Echo({
        broadcaster: 'pusher',
        key: 'your-pusher-key'
    });

#### Redis

如果你使用 Redis，你應該安裝 Predis 套件：

    composer require predis/predis

Redis 廣播器會使用 Redis 發佈/訂閱 功能來廣播訊息。然而，你還是需要搭配 WebSocket 來接受來自 Redis 的訊息，並廣播它們到 WebSocket 的頻道

當 Redis 廣播器發布一個事件時，該事件會被發佈到指定的頻道上，裝載的資料會是 JSON 格式，並包含了事件名稱、 `data` 和使用者產生的事件 Socket ID（如果需要的話）：

#### Socket.IO

你將需要引入 Socket.IO JavaScript 客戶端函式庫到你應用程式的 HTML `head` 標籤。當啟動 Socket.IO 伺服器時，它會自動公開客戶端的 JavaScript 函式庫的 URL。例如，如果你執行的 Socket.IO 和網頁在同一個網域，你可以像下面這樣來讓前端存取你的 Socket.IO 的 JavaScript 函式庫：

    <script src="//{% raw %} {{ Request::getHost() }} {% endraw %}:6001/socket.io/socket.io.js"></script>

接著，你會需要指定 `socket.io` 和 `host` 連線來實例化 Echo。

    import Echo from "laravel-echo"

    window.Echo = new Echo({
        broadcaster: 'socket.io',
        host: window.location.hostname + ':6001'
    });

最後，你將要執行一個相容於 Socket.IO 的伺服器。Laravel 不會引入一個 Socket.IO 伺服器的實作；然而，一個受到社群驅使所開發的 Socket.IO 伺服器目前維護在 Github 的 [tlaverdure/laravel-echo-server](https://github.com/tlaverdure/laravel-echo-server) Repository。

#### Queue 先決條件

在廣播事件之前，你也許會需要設定和執行[隊列監聽器](/laravel_tw/docs/5.5/queues)。所有的事件廣播都會透過隊列任務，使你的程式回應時間不會受到太嚴重的影響。

<a name="concept-overview"></a>
## 概念簡述

Laravel 的事件廣播允許你使用基於驅動的 WebSockets 將後端的 Laravel 事件廣播給前端 JavaScript 應用程式。目前，Laravel 內建了 [Pusher](https://pusher.com) 和 Redis 的驅動。在前端使用 [Laravel Echo](#installing-laravel-echo) Javascript 的套件，可以更簡單的處理事件

事件通過「頻道」來廣播，頻道可以被指定為公開或私人的。你的應用程式的任何訪客都可以訂閱公共頻道，且不需要在認證身份或檢查授權。然而，如果為了訂閱一個私人頻道，使用者就必須認證身份與通過授權才可以監聽該頻道。

<a name="using-example-application"></a>
### 使用一個範例應用程式

在深入廣播事件的每個元件前，讓我們用電子商務作為較完整的例子。在這邊，我們不會討論如何設定 [Pusher](https://pusher.com) 或 [Laravel Echo](#installing-laravel-echo)，因為這些內容會在其他部分的技術文件做詳細討論。

在我們的應用程式中，假設我們有一個頁面，提供給使用者查看訂單的運送狀況。讓我們假設在應用程式處理更新出貨進度時，會觸發 `ShippingStatusUpdated` 的事件：

    event(new ShippingStatusUpdated($update));

####  `ShouldBroadcast` 介面

當使用者正在查看其中一個訂單時，我們並不想要讓他們透過重新整理這個頁面的方式來檢視狀態更新與否。相反的，我們想要在他們建立訂單時透過廣播去更新狀態。所以，我們需要在 `ShippingStatusUpdated` 事件中實作`ShouldBroadcast` 介面。這會讓 Laravel 在事件被觸發時，廣播這個事件：

    <?php

    namespace App\Events;

    use Illuminate\Broadcasting\Channel;
    use Illuminate\Queue\SerializesModels;
    use Illuminate\Broadcasting\PrivateChannel;
    use Illuminate\Broadcasting\PresenceChannel;
    use Illuminate\Broadcasting\InteractsWithSockets;
    use Illuminate\Contracts\Broadcasting\ShouldBroadcast;

    class ShippingStatusUpdated implements ShouldBroadcast
    {
        /**
         * 關於更新出貨進度的資訊。
         *
         * @var string
         */
        public $update;
    }

`ShouldBroadcast` 介面要求我們的事件去定義一個 `broadcastOn` 方法。這個方法負責回傳事件應該廣播到哪個頻道。已經在產生的事件類別上定義了這個方法，我們只需要填寫它的細節。我們希望訂單的建立者只能夠查看狀態更新，因此我們將在訂單相關的私人頻道上廣播該事件：

    /**
     * 取得事件應該被廣播的頻道。
     *
     * @return array
     */
    public function broadcastOn()
    {
        return new PrivateChannel('order.'.$this->update->order_id);
    }

#### 認證頻道

請記得，使用者需要被授權才能去監聽私人頻道。我們可以在 `routes/channels.php` 定義我們頻道的授權規則。在這個範例中，我們需要驗證任何在 `order.1` 的私人頻道上嘗試監聽的使用者是否為實際該訂單的持有人：

    Broadcast::channel('order.{orderId}', function ($user, $orderId) {
        return $user->id === Order::findOrNew($orderId)->user_id;
    });

`channel` 方法接受兩個參數：一個是頻道名稱，另一個是用來回傳 `true` 或 `false` 的回呼，這個回呼用來表示使用者是否被授權可以在頻道上監聽。

所有的授權回呼都會將當前認證的使用者作為第一個參數和任何額外的 wildcard 參數作為後續的參數。在這個範例中，我們使用 `{orderId}` 來表示頻道名稱的「ID」。

#### 監聽廣播事件

接著，就只剩下在 JavaScript 中監聽事件了。我們在這能使用 Laravel Echo。首先，我們會使用 `private` 方法去訂閱私人頻道。然後，我們可以使用 `listen` 方法來監聽 `ShippingStatusUpdated` 事件。在預設上，所有事件的公共屬性會被載入廣播事件中：

    Echo.private(`order.${orderId}`)
        .listen('ShippingStatusUpdated', (e) => {
            console.log(e.update);
        });

<a name="defining-broadcast-events"></a>
## 定義廣播事件

為了通知 Laravel 廣播一個給定的事件，在事件類別上實作 `Illuminate\Contracts\Broadcasting\ShouldBroadcast` 介面。這個介面已經透過框架引入所有被產生的事件類別，你可以輕鬆地新增它到任何的事件。

`ShouldBroadcast` 介面要求你實作一個方法：`broadcastOn`。`broadcastOn` 方法應該回傳需要被廣播的事件的頻道或是頻道陣列。頻道應該是 `Channel`、`PrivateChannel` 或是 `PresenceChannel` 的實例。頻道會有三種實例：`Channel`、`PrivateChannel` 和 `PresenceChannel`。`Channel` 實例代表任何使用者都可以訂閱的公共頻道，而 `PrivateChannels` 和 `PresenceChannels` 則代表需要[頻道授權](#authorizing-channels)的私人頻道：

    <?php

    namespace App\Events;

    use Illuminate\Broadcasting\Channel;
    use Illuminate\Queue\SerializesModels;
    use Illuminate\Broadcasting\PrivateChannel;
    use Illuminate\Broadcasting\PresenceChannel;
    use Illuminate\Broadcasting\InteractsWithSockets;
    use Illuminate\Contracts\Broadcasting\ShouldBroadcast;

    class ServerCreated implements ShouldBroadcast
    {
        use SerializesModels;

        public $user;

        /**
         * 建立一個新的事件實例。
         *
         * @return void
         */
        public function __construct(User $user)
        {
            $this->user = $user;
        }

        /**
         * 取得事件會被廣播的頻道。
         *
         * @return Channel|array
         */
        public function broadcastOn()
        {
            return new PrivateChannel('user.'.$this->user->id);
        }
    }

然後，你只需要和平常一樣[觸發事件](/laravel_tw/docs/5.5/events)。一旦事件被觸發，[隊列任務](/laravel_tw/docs/5.5/queues)會通過你指定的廣播驅動自動廣播事件。

<a name="broadcast-name"></a>
### 廣播名稱

預設上，Laravel 會使用事件類別名稱去廣播事件。然而，你可以在事件上定義 `broadcastAs` 方法來自訂廣播名稱：

    /**
     * 事件的廣播名稱。
     *
     * @return string
     */
    public function broadcastAs()
    {
        return 'server.created';
    }

如果你使用 `broadcastAs` 方法自訂廣播名稱，你應該確保在你註冊的監聽器加上前綴 `.`。這會告訴 Laravel Echo 不要把應用程式的命名空間加入到事件中：

    .listen('.server.created', function (e) {
        ....
    });


<a name="broadcast-data"></a>
### 廣播資料

當一個事件廣播時，所有 `public` 屬性的事件會自動地被序列，而且廣播被事件作為資料，允許讓你從 JavaScript 應用程式存取任何的公共資料。所以舉例來說，如果你的事件有一個 public `$user` 屬性，它包含了一個 Eloquent 模型，那麼事件的廣播資料會是：

    {
        "user": {
            "id": 1,
            "name": "Patrick Stewart"
            ...
        }
    }

然而，如果你希望對廣播資料有更細微的控制，你可以加入 `broadcastWith` 方法到你的事件中。這個方法會回傳一組陣列資料，並如你所希望的廣播事件資料：

    /**
     * 取得資料到廣播。
     *
     * @return array
     */
    public function broadcastWith()
    {
        return ['id' => $this->user->id];
    }

<a name="broadcast-queue"></a>
### 廣播隊列

預設上，每個廣播事件都放在預設隊列上，這些預設的指定隊列的連線在 `queue.php` 設定檔。你可以在你的事件類別上自訂定義 `broadcastQueue` 屬性來使用隊列。這個屬性會指定你希望在廣播的時候要用的隊列名稱：

    /**
     * 要放置事件的隊列名稱。
     *
     * @var string
     */
    public $broadcastQueue = 'your-queue-name';

如果你想要使用 `sync` 隊列而不是使用預設的隊列驅動來廣播你的事件，你可以實作 `ShouldBroadcastNow` 介面而不是 `ShouldBroadcast` 介面:

    <?php

    use Illuminate\Contracts\Broadcasting\ShouldBroadcastNow;

    class ShippingStatusUpdated implements ShouldBroadcastNow
    {
        //
    }
    
<a name="broadcast-conditions"></a>
### 廣播條件

有些時候，你只想在給定條件為 `true`時，才廣播該事件。你可以在你的事件類別上定義 `broadcastWhen` 方法，並增加一些你想要的條件陳述式：

    /**
     * 定義如果事件應該被廣播。
     *
     * @return bool
     */
    public function broadcastWhen()
    {
        return $this->value > 100;
    }

<a name="authorizing-channels"></a>
## 授權給頻道

私人頻道只允許你授權過的使用者才可以監聽頻道。這個過程是使用者發送一個 HTTP 請求到指定的 Laravel 的頻道名稱，並判斷該使用者是否可以監聽該頻道。當你在使用 [Laravel Echo](#installing-laravel-echo) 的時候，將會自行發送 HTTP 請求到想要訂閱的私人頻道。然而，你需要定義路由來回應這些請求。

<a name="defining-authorization-routes"></a>
### 定義授權的路由

幸好，Laravel 可以輕易地定義路由來回應頻道授權的請求。在你的 Laravel 應用程式內的 `BroadcastServiceProvider` 中，你將可以看到一個呼叫 `Broadcast::routes` 方法。這個方法將註冊 `/broadcasting/auth` 路由來處理授權請求：

    Broadcast::routes();

`Broadcast::routes` 方法會自行將它的路由放置 `web` 中介層群組中。然而，如果你想要自定一些屬性，你可以傳遞一組路由屬性的陣列到這個方法：

    Broadcast::routes($attributes);

<a name="defining-authorization-callbacks"></a>
### 定義授權的回呼函式

接下來，我們需要定義實際執行頻道授權的邏輯。這是在 Laravel 內建的 `routes/channels.php` 引入完成的。在這個檔案，你可以使用 `Broadcast::channel` 方法註冊頻道授權的回呼：

    Broadcast::channel('order.{orderId}', function ($user, $orderId) {
        return $user->id === Order::findOrNew($orderId)->user_id;
    });

`channel` 方法接受兩個參數：其一是頻道名稱，另一個是回傳 `true` 或 `false`，用來判斷使用者是否有授權監聽頻道的回呼。

所有的授權回呼都會將當前已認證使用者作為第一個參數，和任何其他萬元字元作為後續參數。在這個範例中，我們使用  `{orderId}` 來表示頻道名稱的「 ID 」。

#### 授權回呼模型綁定

就像 HTTP 路由，頻道路由也可以利用隱式和顯示[路由模型綁定](/laravel_tw/docs/5.5/routing#route-model-binding)。舉例來說，相對於接收字串或數字的 Order ID，你可以請求實際的 `Order` 模型實例：

    use App\Order;

    Broadcast::channel('order.{order}', function ($user, Order $order) {
        return $user->id === $order->user_id;
    });

<a name="broadcasting-events"></a>
## 廣播事件

如果你在一個事件類別上實作 `ShouldBroadcast` 介面，那麼你只需要使用 `event` 函式來觸發事件。事件發送器會注意到該事件有實作 `ShouldBroadcast` 介面，並將事件加入到隊列等待廣播：

    event(new ShippingStatusUpdated($update));

<a name="only-to-others"></a>
### 只廣播給其他人

當你利用事件廣播建立一個應用程式時，你可以用 `broadcast` 函式來替換 `event` 函式，像 `event` 函式一樣， `broadcast` 函式派發事件到你的伺服器端的監聽器：

    broadcast(new ShippingStatusUpdated($update));

然而， `broadcast` 函式還有個 `toOthers` 方法可以讓你把當前使用者從廣播接收名單中排除：

    broadcast(new ShippingStatusUpdated($update))->toOthers();

為了更好理解何種情況會運用到 `toOthers` 方法，讓我們設想一個任務清單的應用程式，使用者可以輸入任務名稱來建立新的任務。要建立一個任務，首先你的應用程式會送出一個建立新任務的請求到 `/task` 路由，接著會觸發事件並廣播給使用者送出請求的結果，回傳的內容會 JSON 格式表示。當你的 JavaScript 應用程式接收到路由回應的內容，它會直接加入新的任務到任務清單中，像是：

    axios.post('/task', task)
        .then((response) => {
            this.tasks.push(response.data);
        });

然而，請記得我們剛廣播了這個任務的新建立事件。如果你的 JavaScript 應用程式同時也在監聽同一件事，你會得到一組重複的內容在同一個任務清單上：一個來自路由的，另一個來自廣播。

這時你就可以使用 `toOthers` 方法來告訴廣播器不要再廣播給觸發事件的使用者來解決這個問題。

#### 設定

在你初始化 laravel-echo 實例的時候，socket ID 會被指定到該連線上。如果你是使用 [Vue](https://vuejs.org) 和 [Axios](https://github.com/mzabriskie/axios) 這個組合，socket ID 會自動增加 `X-Socket-ID` header 到每個送出的請求上。接著，當你呼叫 `toOthers` 方法時，Laravel 會從 header 提出 socket ID，並告訴廣播器不要廣播到同一個 socket ID 的連線上。

如果你不是使用 Vue 和 Axios，你需要自行設定 JavaScript 應用程式來送出 `X-Socket-ID` header。你可以使用 `Echo.socketId` 方法來取得 socket ID：

    var socketId = Echo.socketId();

<a name="receiving-broadcasts"></a>
## 接收廣播

<a name="installing-laravel-echo"></a>
### 安裝 Laravel Echo

Laravel Echo 是一個 JavaScript 程式庫，它可以透過 Laravel 無痛的訂閱頻道與監聽事件廣播。你可以透過 NPM 套件管理器來安裝 Echo。在這個範例中，我們也會安裝 `pusher-js` 套件，因為我們會使用 Pusher 來廣播：

    npm install --save laravel-echo pusher-js

一旦安裝好 Echo，你可以準備在你的 JavaScript 應用程式中建立全新的 Echo 實例。好消息是 Laravel 已為你寫好並放在 `resources/assets/js/bootstrap.js` 檔案的最底下：

    import Echo from "laravel-echo"

    window.Echo = new Echo({
        broadcaster: 'pusher',
        key: 'your-pusher-key'
    });

在使用 `pusher` 來建立 Echo 實例時，你還可以指定 `cluster` 以及是否需要加密連線：

    window.Echo = new Echo({
        broadcaster: 'pusher',
        key: 'your-pusher-key',
        cluster: 'eu',
        encrypted: true
    });

<a name="listening-for-events"></a>
### 監聽事件

一旦安裝並實例化了 Echo，你就可以準備監聽事件廣播。首先，使用 `channel` 方法來取得一個頻道實例，然後呼叫 `listen` 方法去監聽指定的事件：

    Echo.channel('orders')
        .listen('OrderShipped', (e) => {
            console.log(e.order.name);
        });

如果你想在私人頻道上監聽事件，可以使用 `private` 方法。你可以繼續鏈結呼叫的 `listen` 方法去監聽同個頻道上的多個事件：

    Echo.private('orders')
        .listen(...)
        .listen(...)
        .listen(...);

<a name="leaving-a-channel"></a>
### 離開頻道

想要離開頻道，你可以在你的 Echo 實例上呼叫 `leave` 方法：

    Echo.leave('orders');

<a name="namespaces"></a>
### 命名空間

你可能注意到上述範例中沒有為事件類別指定完整的命名空間。這是因為 Echo 會自動假設事件會在 `App\Events` 這個命名空間裡。話雖如此，你可以在實例化 Echo 的時候傳遞 `namespace` 設定選項來設定想要的命名空間：

    window.Echo = new Echo({
        broadcaster: 'pusher',
        key: 'your-pusher-key',
        namespace: 'App.Other.Namespace'
    });

另外，當你使用 Echo 訂閱它們的時候，你應該在事件類別加上前綴 `.`。這會允許你總是指定完整的類別名稱：

    Echo.channel('orders')
        .listen('.Namespace.Event.Class', (e) => {
            //
        });

<a name="presence-channels"></a>
## Presence 頻道

Presence 頻道建構在私人頻道的安全性上，同時多了一個知道誰被訂閱在頻道上的額外功能。這使得它可以輕鬆建立強大的協作應用程式功能，例如當使用者都在瀏覽相同頁面時，通知其他使用者。

<a name="authorizing-presence-channels"></a>
### 授權給 Presence 頻道

所有的 presence 頻道也算是私人頻道。因此，使用者必須[被授權才可存取他們](#authorizing-channels)。然而，在定義 presence 頻道授權回呼的時候，若有個使用者有被授權進入頻道，則回傳 `true`，反之，你應該回傳一組關於使用者資料的陣列。

授權回呼所回傳的資料會用在 JavaScript 應用程式中，由授權回呼回傳的資料將提供給 presence 通道事件偵聽器。。如果使用者未被授權允許加入 presence 頻道，你應該回傳 `false` 或 `null`：

    Broadcast::channel('chat.{roomId}', function ($user, $roomId) {
        if ($user->canJoinRoom($roomId)) {
            return ['id' => $user->id, 'name' => $user->name];
        }
    });

<a name="joining-presence-channels"></a>
### 加入到 Presence 頻道

要加入 presence 頻道，你可以使用 Echo 的 `join` 方法。 `join` 方法會回傳一個 `PresenceChannel` 實作，並且你還可以繼續加上 `listen` 方法，這會允許你使用訂閱 `here`、`joining` 和 `leaving` 事件。

    Echo.join(`chat.${roomId}`)
        .here((users) => {
            //
        })
        .joining((user) => {
            console.log(user.name);
        })
        .leaving((user) => {
            console.log(user.name);
        });

如果頻道連線成功，`here` 回呼就會立即執行，然後會接收到一組包含目前訂閱頻道的所有使用者的資訊。當一個新使用者加入頻道的時後會執行 `joining` 方法，而在使用者來開頻道的時候執行 `leaving` 方法。

<a name="broadcasting-to-presence-channels"></a>
### 廣播到 Presence 頻道

Presence 可以像私人或是公開頻道一樣接收事件。使用一個聊天室的範例，我們可能想要廣播 `NewMessage` 事件到聊天室的 presence 頻道。為此，我們將從事件的 `broadcastOn` 方法回傳一個 `PresenceChannel` 實例：

    /**
     * 取得應該廣播事件的頻道。
     *
     * @return Channel|array
     */
    public function broadcastOn()
    {
        return new PresenceChannel('room.'.$this->message->room_id);
    }

就像是公開或是私人頻道上的事件， presence 頻道上的事件可以使用 `broadcast` 函式來廣播。與其他事件一樣，你可以使用 `toOthers` 方法來將使用者從廣播接收名單中排除。

    broadcast(new NewMessage($message));

    broadcast(new NewMessage($message))->toOthers();

你可以透過 `listen` 方法來監聽 join 事件：

    Echo.join(`chat.${roomId}`)
        .here(...)
        .joining(...)
        .leaving(...)
        .listen('NewMessage', (e) => {
            //
        });

<a name="client-events"></a>
## 客戶端事件

有時候你可能希望向其他連接的客戶端廣播一個事件，而不觸發你的 Laravel 應用程式。這對於像是「正在輸入」的通知特別有用，你想要提醒應用程式的使用者，另一個使用者正在給定的視窗上輸入訊息。要廣播客戶端事件，你可以使用 Echo 的 `whisper` 方法：

    Echo.channel('chat')
        .whisper('typing', {
            name: this.user.name
        });

若要監聽客戶端事件，你可以使用 `listenForWhisper` 方法：

    Echo.channel('chat')
        .listenForWhisper('typing', (e) => {
            console.log(e.name);
        });

<a name="notifications"></a>
## 通知

事件廣播搭配[通知系統](/laravel_tw/docs/5.5/notifications)，可讓你的 JavaScript 應用程式在不需要重新整理頁面得情況下接收新的通知。首先，請先閱讀使用[廣播通知頻道](/laravel_tw/docs/5.5/notifications#broadcast-notifications)的文件。

如果你已設定通知系統到廣播頻道，你就可以使用 Echo 的 `notification` 方法來監聽廣播事件。但請記得，頻道名稱應該與接收通知的實例之類別名稱一致：

    Echo.private(`App.User.${userId}`)
        .notification((notification) => {
            console.log(notification.type);
        });

在這個範例中，凡是透過廣播頻道發送到 `App\User` 實例的所有通知都將會被回呼所接收。`App.User.{id}` 頻道的授權回呼已載入到預設的 `BroadcastServiceProvider`。
