---
layout: post
title: notifications
tag: 5.5
---
# 通知

- [介紹](#introduction)
- [建立通知](#creating-notifications)
- [寄送通知](#sending-notifications)
    - [使用 Notifiable Trait](#using-the-notifiable-trait)
    - [使用 Notification Facade](#using-the-notification-facade)
    - [指派遞送頻道](#specifying-delivery-channels)
    - [通知隊列](#queueing-notifications)
    - [隨需通知](#on-demand-notifications)
- [信件通知](#mail-notifications)
    - [格式化信件訊息](#formatting-mail-messages)
    - [自訂收件者](#customizing-the-recipient)
    - [自訂標題](#customizing-the-subject)
    - [自訂信件模板](#customizing-the-templates)
- [Markdown 通知信件格式](#markdown-mail-notifications)
    - [產生訊息](#generating-the-message)
    - [撰寫訊息](#writing-the-message)
    - [自訂元件](#customizing-the-components)
- [資料庫通知](#database-notifications)
    - [預先準備](#database-prerequisites)
    - [格式化資料庫通知](#formatting-database-notifications)
    - [存取通知](#accessing-the-notifications)
    - [標記通知為已讀](#marking-notifications-as-read)
- [廣播通知](#broadcast-notifications)
    - [預先準備](#broadcast-prerequisites)
    - [格式化廣播通知](#formatting-broadcast-notifications)
    - [監聽通知](#listening-for-notifications)
- [SMS 通知](#sms-notifications)
    - [預先準備](#sms-prerequisites)
    - [格式化 SMS 通知](#formatting-sms-notifications)
    - [自訂 "From" 號碼](#customizing-the-from-number)
    - [路由 SMS 通知](#routing-sms-notifications)
- [Slack 通知](#slack-notifications)
    - [預先準備](#slack-prerequisites)
    - [格式化 Slack 通知](#formatting-slack-notifications)
    - [Slack 附件](#slack-attachments)
    - [路由 Slack 通知](#routing-slack-notifications)
- [通知事件](#notification-events)
- [自訂頻道](#custom-channels)

<a name="introduction"></a>
## 介紹

除了支援[寄送 email](/laravel_tw/docs/5.5/mail)，Laravel 提供了多種寄送通知的支援，包含信件、 SMS (透過 [Nexmo](https://www.nexmo.com/))、[Slack](https://slack.com)，通知也能儲存在資料庫內以在你的網頁介面上顯示。

通常，通知訊息都很簡短，訊息會用來通知你的使用者某些應用程式的資訊。舉例來說，如果你正撰寫一個帳單應用程式，你或許會通過 Email 及 SMS 頻道寄送一個「發票支付」通知給你的使用者。

<a name="creating-notifications"></a>
## 建立通知

在 Laravel ，每則通知皆以使用單一類別（通常放置在 `app/Notifications` 目錄）來表示。如果你在你的應用程式專案底下沒有看到這個目錄，別擔心，在你執行 Artisan 命令 `make:notification` 會被建立：

    php artisan make:notification InvoicePaid

這個命令會放置一個新的通知類別在你的 `app/Notifications` 目錄。每個通知類別包含了 `via` 方法以及數個訊息構建方法（像是 `toMail` 和 `toDatabase`） 用以轉換通知至特定頻道的訊息。

<a name="sending-notifications"></a>
## 寄送通知

<a name="using-the-notifiable-trait"></a>
### 使用 Notifiable Trait

通知可以使用兩種方式寄送：使用 `Notifiable` trait 內的 `notify` 方法或使用 `Notification` [facade](/laravel_tw/docs/5.5/facades)。首先，這邊探索一下使用 trait 的範例：

    <?php

    namespace App;

    use Illuminate\Notifications\Notifiable;
    use Illuminate\Foundation\Auth\User as Authenticatable;

    class User extends Authenticatable
    {
        use Notifiable;
    }

這個 trait 被預設的 `App\User` 模型使用，並且包含了一個用於寄送通知的方法：`notify`。`notify` 方法會用來接收一個通知實例：

    use App\Notifications\InvoicePaid;

    $user->notify(new InvoicePaid($invoice));

> {tip} 記住，你也能在任何模型內使用 `Illuminate\Notifications\Notifiable` trait，並不受限只在 `User` 模型內使用。

<a name="using-the-notification-facade"></a>
### 使用 Notification Facade

另外，你也能透過 `Notification` [facade](/laravel_tw/docs/5.5/facades) 傳送通知。當您需要將通知發送到多個可通知的實體（例如用戶集合）時，這非常有用。為了在發送通知時使用 facade ，需傳遞所有可通知實體和通知實例至 `send` 方法：

    Notification::send($users, new InvoicePaid($invoice));

<a name="specifying-delivery-channels"></a>
### 指派遞送頻道

每個通知類別都有一個 `via` 方法用於判別要將通知寄送哪個頻道。換句話說，通知可以被寄送至 `mail`、`database`、`broadcast`、`nexmo` 和 `slack` 頻道。

> {tip} 如果你想要使用其他的遞送方式，像是 Telegram 或 Pusher ，可以參考社群維護的 [Laravel Notification Channels 網站](http://laravel-notification-channels.com)。

`via` 方法接收了一個 `$notifiable` 實例，該實例屬於寄送通知的類別。你可以使用 `$notifiable` 用於指定用於接收訊息的頻道：

    /**
     * 取得通知的遞送頻道
     *
     * @param  mixed  $notifiable
     * @return array
     */
    public function via($notifiable)
    {
        return $notifiable->prefers_sms ? ['nexmo'] : ['mail', 'database'];
    }

<a name="queueing-notifications"></a>
### 通知隊列

> {note} 在使用推知隊列之前，你應該先設定你的隊列並啟用[一個 worker](/laravel_tw/docs/5.5/queues)。

寄送通知可能十分花費時間，特別是頻道需要呼叫外部的 API 用於寄送通知。為了加速應用程式的回應時間，你可以在你的通知類別內新增 `ShouldQueue` 介面和 `Queueable` trait 。若你使用 `make:notification` 命令產生通知類別，該介面和 trait 已經在類別內自動被引用，你可以在通知類別內立即使用：

    <?php

    namespace App\Notifications;

    use Illuminate\Bus\Queueable;
    use Illuminate\Notifications\Notification;
    use Illuminate\Contracts\Queue\ShouldQueue;

    class InvoicePaid extends Notification implements ShouldQueue
    {
        use Queueable;

        // ...
    }

一旦 `ShouldQueue` 介面被新增至通知類別內，你可以一如往常的呼叫通知類別。Laravel 會自動的偵測類別內的 `ShouldQueue` 介面自動的以隊列的方式遞送通知：

    $user->notify(new InvoicePaid($invoice));

如果你想要延遲遞送通知，你可以在初始化通知時使用 `delay` 方法建立通知鏈：

    $when = Carbon::now()->addMinutes(10);

    $user->notify((new InvoicePaid($invoice))->delay($when));

<a name="on-demand-notifications"></a>
### 隨需通知

有時後你的應用程式需要寄送通知至並不是以 "user" 類別儲存的使用者。使用 `Notification::route` 方法，你能在寄送通知前指定臨時的通知路由資訊：

    Notification::route('mail', 'taylor@laravel.com')
                ->route('nexmo', '5555555555')
                ->notify(new InvoicePaid($invoice));

<a name="mail-notifications"></a>
## 信件通知

<a name="formatting-mail-messages"></a>
### 格式化信件訊息

如果一個通知支援以信件的格式寄送，你能在通知的類別內定義一個 `toMail` 方法。該發法會接收一個 `$notifiable` 實體並且返回一個 `Illuminate\Notifications\Messages\MailMessage` 實例。信件訊息可能包含多行的文字及 "call to action" 等，讓我們來看看一個 `toMail` 方法的範例：

    /**
     * 返回通知信件的顯示格式
     *
     * @param  mixed  $notifiable
     * @return \Illuminate\Notifications\Messages\MailMessage
     */
    public function toMail($notifiable)
    {
        $url = url('/invoice/'.$this->invoice->id);

        return (new MailMessage)
                    ->greeting('Hello!')
                    ->line('One of your invoices has been paid!')
                    ->action('View Invoice', $url)
                    ->line('Thank you for using our application!');
    }

> {tip} 注意這裏我們在 `message` 方法內使用 `$this->invoice->id` 取得收據資訊。你能在通知類別建構子內傳遞通知所需要的任何資料用以產生訊息。

在這個範例中，我們呼叫了 greeting 方法、action 方法和傳遞一行文字至 line 方法。這些方法由 `MailMessage` 物件提供，簡化並快速的格式化信件的訊息。該信件頻道會轉換訊息元件至易讀且響應式的 HTML 信件範本，與純文字相應，以下為一個使用 `mail` 頻道產生的 email：

<img src="https://laravel.com/assets/img/notification-example.png" width="551" height="596">

> {tip} 當寄送一個 Email 通知時，確保你已經在 `config/app.php` 設定檔內設置 `name`。這個設定會在信件通知訊息的標頭和結尾被使用到。

#### 其他的通知格式選項

除了在通知類別內使用字串一行一行的定義文字，你也可以使用 `view` 方法來指定一個自訂的模板用來渲染通知信件：

    /**
     * 取得通知信件的樣式
     *
     * @param  mixed  $notifiable
     * @return \Illuminate\Notifications\Messages\MailMessage
     */
    public function toMail($notifiable)
    {
        return (new MailMessage)->view(
            'emails.name', ['invoice' => $this->invoice]
        );
    }

此外，你可以在 `toMail` 方法內回傳一個 [mailable 物件](/laravel_tw/docs/5.5/mail)：

    use App\Mail\InvoicePaid as Mailable;

    /**
     * 取得通知信件的樣式
     *
     * @param  mixed  $notifiable
     * @return Mailable
     */
    public function toMail($notifiable)
    {
        return (new Mailable($this->invoice))->to($this->user->email);
    }

<a name="error-messages"></a>
#### 錯誤訊息

有些通知會用來提醒使用者某些錯誤資訊，像是付款失敗。你可以在建立你的通知訊息時利用 `error` 方法在信件資訊內指出相關的錯誤資訊。當在信件訊息內使用 `error` 方法，call to action 的按鈕則會顯示為紅色而不是藍色：

    /**
     * 取得通知信件的樣式
     *
     * @param  mixed  $notifiable
     * @return \Illuminate\Notifications\Message
     */
    public function toMail($notifiable)
    {
        return (new MailMessage)
                    ->error()
                    ->subject('Notification Subject')
                    ->line('...');
    }

<a name="customizing-the-recipient"></a>
### 自訂收件者

當透過 `mail` 頻道傳送通知時，該通知系統會自動的查看通知實體的 `email` 屬性。你可以透過在實體內定義 `routeNotificationForMail` 方法用來自訂遞送的 email 位置：

    <?php

    namespace App;

    use Illuminate\Notifications\Notifiable;
    use Illuminate\Foundation\Auth\User as Authenticatable;

    class User extends Authenticatable
    {
        use Notifiable;

        /**
         * 路由信件頻道的通知
         *
         * @return string
         */
        public function routeNotificationForMail()
        {
            return $this->email_address;
        }
    }

<a name="customizing-the-subject"></a>
### 自訂標題

預設信件的標題為通知的類別名稱並格式化標題的字串，例如，你的通知類別名為 `InvoicePaid`，則該信件的預設標題則會是 `Invoice Paid`。所以如果你想要指定一個明確的信件標題，你可以在建立你的訊息時呼叫 `subject` 方法：

    /**
     * 取得通知信件的樣式
     *
     * @param  mixed  $notifiable
     * @return \Illuminate\Notifications\Messages\MailMessage
     */
    public function toMail($notifiable)
    {
        return (new MailMessage)
                    ->subject('Notification Subject')
                    ->line('...');
    }

<a name="customizing-the-templates"></a>
### 自訂信件模板

你可以透過發佈通知套件的資源，修改信件通知使用的 HTML 和純文字模板。運行完以下指令後，信件通知的模板就會被放置在 `resources/views/vendor/notifications` 目錄：

    php artisan vendor:publish --tag=laravel-notifications

<a name="markdown-mail-notifications"></a>
## Markdown 通知信件格式

使用 Markdown 信件通知格式讓你能享有預先建立好信件通知的好處，同時有充分的彈性讓你容易撰寫較長的自訂訊息。因為訊息使用 Markdown 格式撰寫，Laravel 能協助你將訊息渲染至美觀的響應式 HTML 模板，同時也能自動的產生相應的純文字格式。

<a name="generating-the-message"></a>
### 產生訊息

你可以使用 Artisan 命令 `make:notification` 其中的 `--markdown` 選項，以產生對應的 Markdown 通知模板：

    php artisan make:notification InvoicePaid --markdown=mail.invoice.paid

如同其他信件通知，使用 Markdown 模板的通知都需要在通知類別內定義一個 `toMail` 方法。你可以使用 `markdown` 方法指定 Markdown 模板的名稱，而不需要使用 `line` 和 `action` 方法來建立通知：

    /**
     * 取得通知信件的樣式
     *
     * @param  mixed  $notifiable
     * @return \Illuminate\Notifications\Messages\MailMessage
     */
    public function toMail($notifiable)
    {
        $url = url('/invoice/'.$this->invoice->id);

        return (new MailMessage)
                    ->subject('Invoice Paid')
                    ->markdown('mail.invoice.paid', ['url' => $url]);
    }

<a name="writing-the-message"></a>
### 撰寫訊息

Markdown 信件通知格式利用 Blade 元件來組合 Markdown 標記語言，這使得你可以輕鬆組合通知訊息同時能使用 Laravel 內建的方法：

    @component('mail::message')
    # Invoice Paid

    Your invoice has been paid!

    @component('mail::button', ['url' => $url])
    View Invoice
    @endcomponent

    Thanks,<br>
    {% raw %} {{ config('app.name') }} {% endraw %}
    @endcomponent

#### Button 元件

按鈕元件會渲染一個置中的按鈕連結，這個元件接受兩個參數，一個 `url` 以及一個可選的 `color` 可接受 `blue` 、`green` 和 `red` 作為傳入值。你可以新增多個按鈕元件至你的通知內：

    @component('mail::button', ['url' => $url, 'color' => 'green'])
    View Invoice
    @endcomponent

#### Panel 元件

Panel 元件可以用來渲染一整個區塊的文字，Panel 區塊內的文字會跟整個通知內容有著不同的背景顏色。這個元件允許你可以用來為特定區塊的文字吸引注意：

    @component('mail::panel')
    This is the panel content.
    @endcomponent

#### Table 元件

Table 元件允許你轉換 Markdown 表格至 HTML 格式的表格，這個元件接受 Markdown 表格的內容。同時支持使用預設的 Markdown 表格對齊符號來對齊表格欄位：

    @component('mail::table')
    | Laravel       | Table         | Example  |
    | ------------- |:-------------:| --------:|
    | Col 2 is      | Centered      | $10      |
    | Col 3 is      | Right-Aligned | $20      |
    @endcomponent

<a name="customizing-the-components"></a>
### 自訂元件

你可以為你的應用程式打造自訂的 Markdown 通知元件，若你要發佈該元件，可以使用 Artisan 命令 `vendor:publish` 和標籤 `laravel-mail` 發佈你的元件：

    php artisan vendor:publish --tag=laravel-mail

這個命令會發佈 Markdown 信件元件至 `resources/views/vendor/mail` 目錄，`mail` 目錄會包含 `html` 和 `markdown` 目錄，其中包括個別可用元件的獨立樣式，你可以依照你的使用需求任意的自訂這些元件。

#### 自訂 CSS

在發佈元件後，`resources/views/vendor/mail/html/themes` 目錄會包含一個 `default.css` 檔案，你可以自訂這個檔案內的 CSS 樣式，該樣式會自動的套用在 Markdown 通知內的 HTML 中。

> {tip} 如果你想要為 Markdown 元件建立全新的樣式，你可以輕易的在 `html/themes` 目錄內撰寫新的 CSS 檔案，並且更改 `mail` 設定檔案內的 `theme` 選項。

<a name="database-notifications"></a>
## 資料庫通知

<a name="database-prerequisites"></a>
### 預先準備

`database` 通知頻道在資料表中存放了通知資訊，該資料包含了像是通知類別和自訂的 JSON 資料用以描述通知訊息等資訊。

你可以透過查詢資料表在你的使用者應用程式介面中以顯示通知，然而，在你開始查詢這些資料前，你需要先建立一個資料表以紀錄這些通知。你可以透過 `notifications:table` 命令產生相對應的資料表結構並且遷移你的資料庫：

    php artisan notifications:table

    php artisan migrate

<a name="formatting-database-notifications"></a>
### 格式化資料庫通知

若你的資料表內有存放通知，你可以在通知類別中定義 `toDatabase` 或 `toArray` 方法。這些方法會接受一個 `$notifiable` 實例並會返回一個 PHP 陣列，該回傳的陣列會被編碼成 JSON 格式並且會被存放在 `notifications` 資料表中的 `data` 欄位。讓我們來看以下一個 `toArray` 的例子：

    /**
     * 取得以陣列表示方式的通知
     *
     * @param  mixed  $notifiable
     * @return array
     */
    public function toArray($notifiable)
    {
        return [
            'invoice_id' => $this->invoice->id,
            'amount' => $this->invoice->amount,
        ];
    }

#### `toDatabase` Vs. `toArray`

`toArray` 方法同時被 `broadcast` 頻道用來判斷哪些資料應該在你的 JavaScript 用戶端推播。若你想要有兩種不同的陣列用以表示 `database` 和 `broadcast` 頻道，你應該定義一個 `toDatabase` 方法而不是 `toArray` 方法。

<a name="accessing-the-notifications"></a>
### 存取通知

一旦通知被存放在資料庫中，你需要一個簡易的方式已在你的可通知實例中存取。`Illuminate\Notifications\Notifiable` Trait 包含了 Laravel 預設的 `App\User` 模型，涵蓋了一個 `notifications` Eloquent 關聯用以回傳通知實例。為了取得這些通知，你可以透過這些 Eloquent 提供的方法進行存取，預設，通知會用 `created_at` 時間戳記進行排序：

    $user = App\User::find(1);

    foreach ($user->notifications as $notification) {
        echo $notification->type;
    }

若你想要取得僅「未讀」的通知，你可以透過 `unreadNotifications` 關聯，同樣地，該通知會用 `created_at` 時間戳記進行排序:

    $user = App\User::find(1);

    foreach ($user->unreadNotifications as $notification) {
        echo $notification->type;
    }

> {tip} 為了在你的 JavaScript 用戶端存取通知，你應該在你的應用程式定義一個通知控制器用以回傳一個通知實例，比如，當前的使用者資訊。然後，透過在你的 JavaScript 用戶端傳遞 HTTP 請求至該控制器的 URI。

<a name="marking-notifications-as-read"></a>
### 標記通知為已讀

一般來說，你會想要標記一則使用者看過的通知為「已讀」。`Illuminate\Notifications\Notifiable` Trait 提供了一個 `markAsRead` 方法，用以更新資料庫記錄中的 `read_at` 欄位：

    $user = App\User::find(1);

    foreach ($user->unreadNotifications as $notification) {
        $notification->markAsRead();
    }

然而，並非以迴圈的方式標註每一則通知為已讀，你可以直接透過使用 `markAsRead` 方法在一個通知類別集合中標記這些通知：

    $user->unreadNotifications->markAsRead();

你可能也會想要使用大規模更新的查詢方式以標記所有通知為已讀而不需要從資料庫中取得這些資訊進行操作：

    $user = App\User::find(1);

    $user->unreadNotifications()->update(['read_at' => Carbon::now()]);

當然，你可能會想要透過 `delete` 方法將這些通知從資料庫中完整的刪除：

    $user->notifications()->delete();

<a name="broadcast-notifications"></a>
## 廣播通知

<a name="broadcast-prerequisites"></a>
### 預先準備

在開始廣播通知前，同樣的，你需要配置 Laravel 提供的[事件廣播](/laravel_tw/docs/5.5/broadcasting) 服務，事件廣播提供一個方法能夠讓你的 JavaScript 用戶端能與伺服器端觸發的事件進行互動。

<a name="formatting-broadcast-notifications"></a>
### 格式化廣播通知

`broadcast` 頻道的廣播通知使用 Laravel 的 [事件廣播](/laravel_tw/docs/5.5/broadcasting) 服務，允許你的 JavaScript 用戶端能夠即時的捕捉通知訊息。如果你想要讓通知支援使用廣播通知，你需要在通知類別中定義一個 `toBroadcast` 方法。該方法會接收一個 `$notifiable` 實體並且回傳 `BroadcastMessage` 實例。回傳的資料會使用 JSON 格式傳遞至你的 JavaScript 用戶端。讓我們來看一個使用 `toBroadcast` 方法的範例：

    use Illuminate\Notifications\Messages\BroadcastMessage;

    /**
     * 轉換可廣播通知的訊息
     *
     * @param  mixed  $notifiable
     * @return BroadcastMessage
     */
    public function toBroadcast($notifiable)
    {
        return new BroadcastMessage([
            'invoice_id' => $this->invoice->id,
            'amount' => $this->invoice->amount,
        ]);
    }

#### 廣播隊列設定

所有的廣播的訊息都會以隊列方式進行廣播，如果你想要設定進行廣播操作時隊列連線的相關配置獲釋設定隊列名稱，你可使用 `BroadcastMessage` 的 `onConnection` 和 `onQueue` 方法：

    return (new BroadcastMessage($data))
                    ->onConnection('sqs')
                    ->onQueue('broadcasts');

> {tip} 除了你指定的訊息之外，廣播通知也會包含一個  `type` 欄位用以包含通知類別的名稱。

<a name="listening-for-notifications"></a>
### 監聽通知

通知會透過以 `{notifiable}.{id}` 慣用表示方式的私人頻道進行廣播，所以，如果你想要針對使用 ID 為 `1` 的 `App\User` 實例進行廣播，該通知會使用 `App.User.1` 的私人頻道進行傳遞。當使用 [Laravel Echo](/laravel_tw/docs/5.5/broadcasting)，你可以輕鬆的利用 `notifications` 輔助函式進行該頻道的監聽：

    Echo.private('App.User.' + userId)
        .notification((notification) => {
            console.log(notification.type);
        });

#### 自訂該通知頻道

如果你想要自訂用哪個頻道用來接收通知通知實體，你可以在通知實體內定義一個 `receivesBroadcastNotificationsOn` 方法：

    <?php

    namespace App;

    use Illuminate\Notifications\Notifiable;
    use Illuminate\Broadcasting\PrivateChannel;
    use Illuminate\Foundation\Auth\User as Authenticatable;

    class User extends Authenticatable
    {
        use Notifiable;

        /**
         * 該使用者用來接收通知的頻道
         *
         * @return string
         */
        public function receivesBroadcastNotificationsOn()
        {
            return 'users.'.$this->id;
        }
    }

<a name="sms-notifications"></a>
## SMS 通知

<a name="sms-prerequisites"></a>
### 預先準備

 Laravel 支持使用 [Nexmo](https://www.nexmo.com/) 傳送 SMS 通知訊息。在透過 Nexmo 傳送通知前，你需要使用 Composer 安裝 `nexmo/client` 套件並且在 `config/services.php` 設定檔內新增部分設定，你可以複製以下的範例設定檔：

    'nexmo' => [
        'key' => env('NEXMO_KEY'),
        'secret' => env('NEXMO_SECRET'),
        'sms_from' => '15556666666',
    ],

`sms_from` 選項可以設定你的 SMS 訊息會由哪個電話號碼進行傳送，你會需要在 Nexmo 的主控台介面中產生出一組電話號碼。

<a name="formatting-sms-notifications"></a>
### 格式化 SMS 通知

若要將一則通知利用 SMS 方式進行傳送，你需要在通知類別中定義 `toNexmo` 方法。該方法允許接受一個 `$notifiable` 實體並且會回傳一個 `Illuminate\Notifications\Messages\NexmoMessage` 實例：

    /**
     * 轉換通知為 Nexmo / SMS
     *
     * @param  mixed  $notifiable
     * @return NexmoMessage
     */
    public function toNexmo($notifiable)
    {
        return (new NexmoMessage)
                    ->content('Your SMS message content');
    }

#### Unicode 內容

如果你的 SMS 訊息會包含 unicode 字元，你可以在建構 `NexmoMessage` 實例時呼叫 `unicode` 方法：

    /**
     * 轉換通知為 Nexmo / SMS
     *
     * @param  mixed  $notifiable
     * @return NexmoMessage
     */
    public function toNexmo($notifiable)
    {
        return (new NexmoMessage)
                    ->content('Your unicode message')
                    ->unicode();
    }

<a name="customizing-the-from-number"></a>
### 自訂 "From" 號碼

若你想要在傳送通知時使用不同於 `config/services.php` 設定檔內指定的電話號碼，你可以在 `NexmoMessage` 實例內使用 `from` 方法：

    /**
     * 轉換通知為 Nexmo / SMS
     *
     * @param  mixed  $notifiable
     * @return NexmoMessage
     */
    public function toNexmo($notifiable)
    {
        return (new NexmoMessage)
                    ->content('Your SMS message content')
                    ->from('15554443333');
    }

<a name="routing-sms-notifications"></a>
### 路由 SMS 通知

當透過 `nexmo` 頻道傳送通知時，該通知會自動的檢查通知實體的 `phone_number` 屬性。如果你想要自訂通知目標傳送對象的電話號碼，你可以在通知實體中定義一個 `routeNotificationForNexmo` 方法：

    <?php

    namespace App;

    use Illuminate\Notifications\Notifiable;
    use Illuminate\Foundation\Auth\User as Authenticatable;

    class User extends Authenticatable
    {
        use Notifiable;

        /**
         * 路由通知至 Nexmo 頻道
         *
         * @return string
         */
        public function routeNotificationForNexmo()
        {
            return $this->phone;
        }
    }

<a name="slack-notifications"></a>
## Slack 通知

<a name="slack-prerequisites"></a>
### 預先準備

在開始使用 Slack 傳送通知前，你需要先透過 Composer 安裝 Guzzle HTTP 函式庫：

    composer require guzzlehttp/guzzle

你也會需要設定一個 ["Incoming Webhook"](https://api.slack.com/incoming-webhooks) 整合至你的 Slack team。這項整合會提供一組 URL，你可以利用這個設置來 [路由 Slack 通知](#routing-slack-notifications).

<a name="formatting-slack-notifications"></a>
### 格式化 Slack 通知

若要讓通知訊息支援由 Slack 傳遞，你可以透過在你的通知類別中定義一個 `toSlack` 方法，該方法會接收一個 `$notifiable` 實例並且會回傳一個 `Illuminate\Notifications\Messages\SlackMessage` 實例。Slack 訊息可以包含純文字內容或是涵蓋附加文字或資料的「附件」，讓我們來看一下一個簡單的 `toSlack` 範例：

    /**
     * 轉換 Slack 通知格式
     *
     * @param  mixed  $notifiable
     * @return SlackMessage
     */
    public function toSlack($notifiable)
    {
        return (new SlackMessage)
                    ->content('One of your invoices has been paid!');
    }

In this example we are just sending a single line of text to Slack, which will create a message that looks like the following:

<img src="https://laravel.com/assets/img/basic-slack-notification.png">

#### 自定義傳送 & 接收對象

你可以使用 `from` 和 `to` 方法來自訂傳送和接收對象，`from` 方法接受使用使用者名稱和 emoji 符號，同時 `to` 方法接受使用頻道或使用者名稱：

    /**
     * 轉換 Slack 通知格式
     *
     * @param  mixed  $notifiable
     * @return SlackMessage
     */
    public function toSlack($notifiable)
    {
        return (new SlackMessage)
                    ->from('Ghost', ':ghost:')
                    ->to('#other')
                    ->content('This will be sent to #other');
    }

你也可以使用一張圖片作為訊息的 Logo 而不是使用 emoji 符號：

    /**
     * 轉換 Slack 通知格式
     *
     * @param  mixed  $notifiable
     * @return SlackMessage
     */
    public function toSlack($notifiable)
    {
        return (new SlackMessage)
                    ->from('Laravel')
                    ->image('https://laravel.com/favicon.png')
                    ->content('This will display the Laravel logo next to the message');
    }

<a name="slack-attachments"></a>
### Slack 附件

你或許會想要附加「附件」至 Slack 訊息中，附件提供了比純文字更豐富的格式選擇。在這個範例中，我們會傳送一個關於應用程式發生例外的錯誤通知，包含了一個關於這個例外發生原因的詳細資訊連結：

    /**
     * 轉換 Slack 通知格式
     *
     * @param  mixed  $notifiable
     * @return SlackMessage
     */
    public function toSlack($notifiable)
    {
        $url = url('/exceptions/'.$this->exception->id);

        return (new SlackMessage)
                    ->error()
                    ->content('Whoops! Something went wrong.')
                    ->attachment(function ($attachment) use ($url) {
                        $attachment->title('Exception: File Not Found', $url)
                                   ->content('File [background.jpg] was not found.');
                    });
    }

上述的範例會產生一則如以下的 Slack 訊息：

<img src="https://laravel.com/assets/img/basic-slack-attachment.png">

附件同時允許你可以指定一個要傳遞給使用者相關資料的陣列，這些資料可以使用表列式的方式使其易讀：

    /**
     * 轉換 Slack 通知格式
     *
     * @param  mixed  $notifiable
     * @return SlackMessage
     */
    public function toSlack($notifiable)
    {
        $url = url('/invoices/'.$this->invoice->id);

        return (new SlackMessage)
                    ->success()
                    ->content('One of your invoices has been paid!')
                    ->attachment(function ($attachment) use ($url) {
                        $attachment->title('Invoice 1322', $url)
                                   ->fields([
                                        'Title' => 'Server Expenses',
                                        'Amount' => '$1,234',
                                        'Via' => 'American Express',
                                        'Was Overdue' => ':-1:',
                                    ]);
                    });
    }

上述的範例會產生一則如以下的 Slack 訊息：

<img src="https://laravel.com/assets/img/slack-fields-attachment.png">

#### Markdown 內容附件

若你的附件中含有 Markdown 的內容，你可以使 `markdown` 方法指示 Slack 解析並且以 Markdown 格式顯示附件中的文字。該方法可接受的值為： `pretext`、 `text`、 和 / 或 `fields`。若想要獲取更多關於 Slack 附件的格式，請參考 [Slack API 文件](https://api.slack.com/docs/message-formatting#message_formatting)：

    /**
     * 轉換為 Slack 通知
     *
     * @param  mixed  $notifiable
     * @return SlackMessage
     */
    public function toSlack($notifiable)
    {
        $url = url('/exceptions/'.$this->exception->id);

        return (new SlackMessage)
                    ->error()
                    ->content('Whoops! Something went wrong.')
                    ->attachment(function ($attachment) use ($url) {
                        $attachment->title('Exception: File Not Found', $url)
                                   ->content('File [background.jpg] was *not found*.')
                                   ->markdown(['text']);
                    });
    }

<a name="routing-slack-notifications"></a>
### 路由 Slack 通知

要路由 Slack 通知至對應的位置，可以在你的通知實例中定義 `routeNotificationForSlack` 方法，並在該方法中回傳對應的 webhook URL。Webhook URLs 可以透過在你的 Slack team 內新增一個 "Incoming Webhook" 服務：

    <?php

    namespace App;

    use Illuminate\Notifications\Notifiable;
    use Illuminate\Foundation\Auth\User as Authenticatable;

    class User extends Authenticatable
    {
        use Notifiable;

        /**
         * 路由通知至 Slack 頻道
         *
         * @return string
         */
        public function routeNotificationForSlack()
        {
            return $this->slack_webhook_url;
        }
    }

<a name="notification-events"></a>
## 通知事件

當一則通知被遞送 `Illuminate\Notifications\Events\NotificationSent` 事件會被通知系統觸發，其中包含 "notifiable" 實例和通知實例本身。你可以在你的 `EventServiceProvider` 中為該事件註冊監聽器：

    /**
     * 該事件監聽器對應至你的應用程式
     *
     * @var array
     */
    protected $listen = [
        'Illuminate\Notifications\Events\NotificationSent' => [
            'App\Listeners\LogNotification',
        ],
    ];

> {tip} 在你的 `EventServiceProvider` 註冊監聽器後，可以使用 `event:generate` Artisan 命令快速的產生監聽器類別。

在事件監聽器內，你可以透過存取事件的 `notifiable` 、 `notification` 和 `channel` 屬性以取得更多通知的接收對象和通知本身

    /**
     * 處理監聽事件
     *
     * @param  NotificationSent  $event
     * @return void
     */
    public function handle(NotificationSent $event)
    {
        // $event->channel
        // $event->notifiable
        // $event->notification
    }

<a name="custom-channels"></a>
## 自訂頻道

Laravel 附帶幾種通知頻道，然而，你可能會想要撰寫你自己的驅動用以遞送通知至不同的頻道。Laravel 使其變得容易，開始前，定義一個類別包含 `send` 方法，該方法需要接受兩個參數一個 `$notifiable` 和一個 `$notification`：

    <?php

    namespace App\Channels;

    use Illuminate\Notifications\Notification;

    class VoiceChannel
    {
        /**
         * 遞送至特定的頻道
         *
         * @param  mixed  $notifiable
         * @param  \Illuminate\Notifications\Notification  $notification
         * @return void
         */
        public function send($notifiable, Notification $notification)
        {
            $message = $notification->toVoice($notifiable);

            // 遞送通知至 $notifiable 實例...
        }
    }

一旦你的通知頻道類別被定義，你可以輕鬆的透過 `via` 方法回傳你的任何通知類別名稱：:

    <?php

    namespace App\Notifications;

    use Illuminate\Bus\Queueable;
    use App\Channels\VoiceChannel;
    use App\Channels\Messages\VoiceMessage;
    use Illuminate\Notifications\Notification;
    use Illuminate\Contracts\Queue\ShouldQueue;

    class InvoicePaid extends Notification
    {
        use Queueable;

        /**
         * 取得通知頻道
         *
         * @param  mixed  $notifiable
         * @return array|string
         */
        public function via($notifiable)
        {
            return [VoiceChannel::class];
        }

        /**
         * 處理呼叫 toVoice 通知中的邏輯
         *
         * @param  mixed  $notifiable
         * @return VoiceMessage
         */
        public function toVoice($notifiable)
        {
            // ...
        }
    }
