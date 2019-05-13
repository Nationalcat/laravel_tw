---
layout: post
title: mail
tag: 5.5
---
# 郵件

- [簡介](#introduction)
    - [驅動前提](#driver-prerequisites)
- [產生 Mailables](#generating-mailables)
- [撰寫 Mailables](#writing-mailables)
    - [設置寄件者](#configuring-the-sender)
    - [設置視圖](#configuring-the-view)
    - [視圖資料](#view-data)
    - [附件](#attachments)
    - [行內附件](#inline-attachments)
    - [自訂 SwiftMailer 訊息](#customizing-the-swiftmailer-message)
- [Markdown Mailables](#markdown-mailables)
    - [產生 Markdown Mailables](#generating-markdown-mailables)
    - [撰寫 Markdown 訊息](#writing-markdown-messages)
    - [自定元件](#customizing-the-components)
- [在瀏覽器中預覽 Mailables](#previewing-mailables-in-the-browser)
- [發送郵件](#sending-mail)
    - [隊列郵件](#queueing-mail)
- [郵件與本機端開發](#mail-and-local-development)
- [事件](#events)

<a name="introduction"></a>
## 簡介

Laravel 基於熱門的 [SwiftMailer](https://swiftmailer.symfony.com/) 函式庫提供了一個簡潔的 API。Laravel 為 SMTP、Mailgun、Mandrill、Amazon SES、PHP 的 `mail` 函式及 `sendmail` 提供驅動，讓你可以快速地以所選擇的本地或雲端服務開始寄送郵件。

<a name="driver-prerequisites"></a>
### 驅動前提

基於 API 的驅動，例如 Mailgun 或 SparkPost，通常比 SMTP 伺服器更簡單快速。如果可以，應該要使用其中一種驅動。所有的 API 驅動都需要 Guzzle HTTP 函式庫，可以用 Composer 套件管理器來安裝：

    composer require guzzlehttp/guzzle

#### Mailgun 驅動

要使用 Mailgun 驅動，必須先安裝 Guzzle，之後將 `config/mail.php` 設定檔中的 `driver` 選項設定為 `mailgun`。接下來，確認 `config/services.php` 設定檔包含下列選項：

    'mailgun' => [
        'domain' => 'your-mailgun-domain',
        'secret' => 'your-mailgun-key',
    ],

#### SparkPost 驅動

要使用 SparkPost 驅動，必須先安裝 Guzzle，之後將 `config/mail.php` 設定檔中的 `sparkpost` 選項設定為 `mailgun`。接下來，確認 `config/services.php` 設定檔包含下列選項： 

    'sparkpost' => [
        'secret' => 'your-sparkpost-key',
    ],

#### SES 驅動

要使用 Amazon SES 驅動，必須先安裝 PHP 的 Amazon AWS SDK。可以在 `composer.json` 檔案的 `require` 段落加入下面這一行並執行 `composer update` 命令來安裝此函式庫：

    "aws/aws-sdk-php": "~3.0"

接下來，將 `config/mail.php` 設定檔中的 `driver` 選項設定為 `ses`，然後確認 `config/services.php` 設定檔包含下列選項：

    'ses' => [
        'key' => 'your-ses-key',
        'secret' => 'your-ses-secret',
        'region' => 'ses-region',  // e.g. us-east-1
    ],

<a name="generating-mailables"></a>
## 產生 Mailables

在 Laravel 中，應用程式送出的每種類型的郵件都是以「mailable」類別表示。這些類別存放在 `app/Mail` 資料夾中。如果沒看到這個資料夾也不必擔心，因為在第一次用 `make:mail` 指令來建立 mailable 類別時會自動產生此資料夾：

    php artisan make:mail OrderShipped

<a name="writing-mailables"></a>
## 撰寫 Mailables

一個 mailable 類別的所有設定都是寫在 `build` 方法中。在此方法中，可以呼叫 `from`、`subject`、`view` 和 `attach` 等不同方法來設定郵件的展示和寄送。

<a name="configuring-the-sender"></a>
### 設置寄件者

#### 使用 `from` 方法

我們先來設定郵件的寄件者。也可以說是郵件「從」誰發出的。有兩個方式來設定寄件者。首先，可以在 mailable 類別的 `build` 方法中用 `from` 方法：

    /**
     * 建立訊息。
     *
     * @return $this
     */
    public function build()
    {
        return $this->from('example@example.com')
                    ->view('emails.orders.shipped');
    }

#### 使用全域的 `from` 位址

然而，如果全部應用程式的郵件都使用同一個「 from 」位址，要在每個你產生的 mailable 類別中都呼叫 `from` 方法會變得很冗餘。代替的做法是，可以在 `config/mail.php` 設定檔中指定一個全域的 `from` 位址：

    'from' => ['address' => 'example@example.com', 'name' => 'App Name'],

<a name="configuring-the-view"></a>
### 設置視圖

在 mailable 類別的 `build` 方法中，可以使用 `view` 方法來指定用來渲染郵件內容的模板。由於郵件一般是用 [Blade 模板](/laravel_tw/docs/5.5/blade) 來渲染內容，可以藉由 Blade 模板引擎完整的功能和便利性來構建郵件的 HTML：

    /**
     * 構建郵件訊息。
     *
     * @return $this
     */
    public function build()
    {
        return $this->view('emails.orders.shipped');
    }

> {tip} 你可能預期要建立一個 `resources/views/emails` 目錄來存放所有郵件模板；其實可以隨意放在 `resources/views` 目錄中的任何位置。

#### 純文字郵件

如果你想要定義純文字版本的郵件，可以用 `text` 方法。如果 `view` 方法，`text` 方法接受用來渲染郵件內容的模板名稱。你可以自由地定義訊息的 HTML 或純文字版本：

    /**
     * 構建郵件訊息。
     *
     * @return $this
     */
    public function build()
    {
        return $this->view('emails.orders.shipped')
                    ->text('emails.orders.shipped_plain');
    }

<a name="view-data"></a>
### 視圖資料

#### 透過公開屬性

一般來說，你會想要傳遞一些資料到視圖以在渲染郵件的 HTML 時使用。有兩種可以在視圖中使用資料的方式。首先，任何定義在 mailable 類別中的公開屬性都會自動傳遞給視圖。例如，可以傳資料進 mailable 類別的建構子並賦值給類別的公開屬性：

    <?php

    namespace App\Mail;

    use App\Order;
    use Illuminate\Bus\Queueable;
    use Illuminate\Mail\Mailable;
    use Illuminate\Queue\SerializesModels;

    class OrderShipped extends Mailable
    {
        use Queueable, SerializesModels;

        /**
         * order 實例。
         *
         * @var Order
         */
        public $order;

        /**
         * 建立新的訊息實例。
         *
         * @return void
         */
        public function __construct(Order $order)
        {
            $this->order = $order;
        }

        /**
         * 構建郵件訊息。
         *
         * @return $this
         */
        public function build()
        {
            return $this->view('emails.orders.shipped');
        }
    }

當資料賦值給公開屬性後，會自動傳遞給視圖，所以可以在 Blade 模板中像存取其他資料般存取它：

    <div>
        價格：{% raw %} {{ $order->price }} {% endraw %}
    </div>

#### 透過 `with` 方法：

如果想要在郵件的資料傳給模板前調整資料格式，可以透過 `with` 方法來手動傳遞資料給視圖。一般來說，你仍然會透過 mailable 類別的建構子來傳資料；不過應該要把屬性設定成 `protected` 或 `private` 才不會自動把資料傳給視圖。然後在呼叫 `with` 方法時，傳進想要提供給視圖使用的資料的陣列：

    <?php

    namespace App\Mail;

    use App\Order;
    use Illuminate\Bus\Queueable;
    use Illuminate\Mail\Mailable;
    use Illuminate\Queue\SerializesModels;

    class OrderShipped extends Mailable
    {
        use Queueable, SerializesModels;

        /**
         * order 實例。
         *
         * @var Order
         */
        protected $order;

        /**
         * 建立新的訊息實例。
         *
         * @return void
         */
        public function __construct(Order $order)
        {
            $this->order = $order;
        }

        /**
         * 構建郵件訊息。
         *
         * @return $this
         */
        public function build()
        {
            return $this->view('emails.orders.shipped')
                        ->with([
                            'orderName' => $this->order->name,
                            'orderPrice' => $this->order->price,
                        ]);
        }
    }

把資料傳給 `with` 方法後，便會自動傳遞給視圖，所以可以在 Blade 模板中像存取其他資料般存取它：

    <div>
        Price: {% raw %} {{ $orderPrice }} {% endraw %}
    </div>

<a name="attachments"></a>
### 附件

要在郵件中加入附件，在 mailable 類別的 `build` 方法中使用 `attach` 方法。`attach` 方法接受檔案的完整路徑作為第一個參數：

        /**
         * 構建郵件訊息。
         *
         * @return $this
         */
        public function build()
        {
            return $this->view('emails.orders.shipped')
                        ->attach('/path/to/file');
        }

附加檔案至訊息時，你也可以傳遞`陣列`給 `attach` 方法的第二個參數，以指定顯示名稱和／或 MIME 型別：

        /**
         * 構建郵件訊息。
         *
         * @return $this
         */
        public function build()
        {
            return $this->view('emails.orders.shipped')
                        ->attach('/path/to/file', [
                            'as' => 'name.pdf',
                            'mime' => 'application/pdf',
                        ]);
        }

#### 原始資料附件

`attachData` 方法可以用來把原始字串的位元組轉為附件。例如，如果在記憶體中產生了一份 PDF 並且想將其加入附件而不是寫進硬碟中，你便可以利用這個方法。`attachData` 方法接受原始資料位元組為第一個參數，檔案名稱為第二個參數，第三個參數則為其他設定的陣列：

        /**
         * 構建郵件訊息。
         *
         * @return $this
         */
        public function build()
        {
            return $this->view('emails.orders.shipped')
                        ->attachData($this->pdf, 'name.pdf', [
                            'mime' => 'application/pdf',
                        ]);
        }

<a name="inline-attachments"></a>
### 行內附件

在郵件中嵌入行內圖片一般是件很麻煩的事；然而 Laravel 提供一個便利的方法，讓你在電子郵件中附加圖像並取得適當的 CID。要嵌入行內圖片，在郵件視圖中使用 `$message` 變數的 `embed` 方法。Laravel 會自動讓所有郵件視圖都能使用 `$message` 變數，所以不用擔心需要手動傳遞這個變數：

    <body>
        這是一張圖片：

        <img src="{% raw %} {{ $message->embed($pathToFile) }} {% endraw %}">
    </body>

> {note} `$message` 變數無法在 markdown 訊息中使用。

#### 嵌入原始資料附件

若你已有希望嵌入郵件模板中的原始資料字串，你可以使用 `$message` 變數的 `embedData` 方法：

    <body>
        這是一張從原始資料來的圖片：

        <img src="{% raw %} {{ $message->embedData($data, $name) }} {% endraw %}">
    </body>

<a name="customizing-the-swiftmailer-message"></a>
### 自訂 SwiftMailer 訊息

`Mailable` 基礎類別的 `withSwiftMessage` 方法允許你註冊一個回呼，在送出訊息前會以原始的 SwiftMailer 訊息實例調用此回呼。這提供你一個在送出前自訂訊息的機會：

        /**
         * 構建郵件訊息。
         *
         * @return $this
         */
        public function build()
        {
            $this->view('emails.orders.shipped');

            $this->withSwiftMessage(function ($message) {
                $message->getHeaders()
                        ->addTextHeader('Custom-Header', 'HeaderValue');
            });
        }

<a name="markdown-mailables"></a>
## Markdown Mailables

Markdown Mailables 訊息允許你利用郵件通知中的預建模板和組件。由於訊息是以 Markdown 來撰寫，Laravel 能夠漂亮地渲染出漂亮且響應式的HTML模板，同時自動生成純文字的副本。

<a name="generating-markdown-mailables"></a>
### 產生 Markdown Mailables

要以對應的 Markdown 模板產生一個 mailable，可以使用 `make:mail` Artisan 指令的 `--markdwon` 選項：

    php artisan make:mail OrderShipped --markdown=emails.orders.shipped

接下來在 `build` 方法中設置 mailable 時，呼叫 `markdown` 方法而不是 `view` 方法。`markdown` 方法接受 Markdown 模板的名稱和非必要的模板中可使用的資料陣列：

    /**
     * 構建郵件訊息。
     *
     * @return $this
     */
    public function build()
    {
        return $this->from('example@example.com')
                    ->markdown('emails.orders.shipped');
    }

<a name="writing-markdown-messages"></a>
### 撰寫 Markdown 訊息

Markdown mailable 使用 Blade 元件跟 Markdown 語法的組合，讓你可以輕鬆構建郵件訊息，同時利用 Laravel 的預製元件：

    @component('mail::message')
    # 訂單出貨通知

    你的訂單已出貨！

    @component('mail::button', ['url' => $url])
    查看訂單
    @endcomponent

    感謝，<br>
    {% raw %} {{ config('app.name') }} {% endraw %}
    @endcomponent

> {tip} 在撰寫 Markdown 郵件時不要使用過多的縮排。Markdown 解析器會將縮排的內容以程式碼區塊呈現。

#### 按鈕元件（ button ）

按鈕元件會渲染出置中的按鈕連結。此元件接受兩個參數，`網址（ url ）` 和非必填的 `顏色（ color ）`。支援的顏色有`藍色（ blue ）`、`綠色（ green ）`和`紅色（ red ）`。可以增加任意數量的按鈕元件到訊息中：

    @component('mail::button', ['url' => $url, 'color' => 'green'])
    查看訂單
    @endcomponent

#### 面板元件（ panel ）

面板元件會在一個背景顏色與訊息稍微不同的面板中渲染出給定的文字區塊。讓你可以將注意力吸引到給定的文字區塊中：

    @component('mail::panel')
    這是面板的內容。
    @endcomponent

#### 表格元件（ table ）

表格元件讓你能把 Markdown 表格轉換成 HTML 表格。此元件接受 Markdown 表格為其內容。預設的 Markdown 表格對齊方向的語法來支援表列對齊方向：

    @component('mail::table')
    | Laravel    | 表格    | 範例   |
    | ---------- |:------:| ------:|
    | 第二列      | 置中    |    $10 |
    | 第三列      | 靠右    |    $20 |
    @endcomponent

<a name="customizing-the-components"></a>
### 自定元件

你可以把所有的 Markdown 郵件元件匯出到自己的應用程式中來做自定調整。要匯出元件，使用 `vendor:publish` Artisan 指令來發布 `laravel-mail` 資源檔標籤：

    php artisan vendor:publish --tag=laravel-mail

此指令會把 Markdown 郵件元件發布到 `resources/views/vendor/mail` 目錄中。`mail` 目錄包含 `html` 和 `markdown` 目錄，每個目錄都包含它們對每個可用組件的相應表示。在 `html` 目錄中的元件是用於產生你的電子信箱的 HTML 範本，而 `markdown` 目錄則適用於產生純文字的範本。你可以隨意自定這些組件。

#### 自定 CSS

匯出元件後，`resources/views/vendor/mail/html/themes` 目錄會包含一個 `default.css` 檔案。您可以在此文件中自定 CSS，且樣式將自動以行內的形式放在 Markdown 郵件的 HTML 表示中。

> {tip} 如果您想要為 Markdown 元件建立一個全新的主題，只需在 `html/themes` 目錄中撰寫一個新的 CSS 檔案，然後更改 `mail` 設定檔中的 `theme` 選項即可。

<a name="previewing-mailables-in-the-browser"></a>
## 在瀏覽器中預覽 Mailables

在設計 mailable 的模板時，像一般的 Blade 模板一樣，快速地在瀏覽器中預覽渲染出的 mailable 是很方便的。因此 Laravel 允許在路由的閉包或控制器中回傳 mailable。當回傳 mailable 時，會在瀏覽器中渲染並呈現，讓你不用發送給實際的郵件位址就可以快速地預覽設計結果：

    Route::get('/mailable', function () {
        $invoice = App\Invoice::find(1);

        return new App\Mail\InvoicePaid($invoice);
    });

<a name="sending-mail"></a>
## 發送郵件

用 `Mail` [facade](/laravel_tw/docs/5.5/facades) 的 `to` 方法來發送訊息。`to` 方法接受一個電子郵件位址、一個使用者實例、或使用者的集合。如果傳進一個物件或物件的集合，mailer 會自動以 `email` 和 `name` 屬性來設定收件者，所以要確定物件的這些屬性可以被使用。在指定收件者後，你可以傳一個 mailable 類別的實例給 `send` 方法：

    <?php

    namespace App\Http\Controllers;

    use App\Order;
    use App\Mail\OrderShipped;
    use Illuminate\Http\Request;
    use Illuminate\Support\Facades\Mail;
    use App\Http\Controllers\Controller;

    class OrderController extends Controller
    {
        /**
         * 訂單出貨。
         *
         * @param  Request  $request
         * @param  int  $orderId
         * @return Response
         */
        public function ship(Request $request, $orderId)
        {
            $order = Order::findOrFail($orderId);

            // 出貨...

            Mail::to($request->user())->send(new OrderShipped($order));
        }
    }

當然在發送訊息時你不只可以指定「 to 」收件者。以單一個鏈結的方法呼叫來隨意設定「 to 」、「 cc 」和「 bcc 」收件者：

    Mail::to($request->user())
        ->cc($moreUsers)
        ->bcc($evenMoreUsers)
        ->send(new OrderShipped($order));

<a name="queueing-mail"></a>
### 隊列郵件

#### 將郵件訊息加入隊列

由於寄送電子郵件訊息會大幅延長應用程式的回應時間，許多開發者選擇將郵件訊息加入隊列以在背景發送。Laravel 使用其內建的[統一的隊列 API](/laravel_tw/docs/5.5/queues)，讓你輕鬆地完成此工作。在定義訊息的收件者後使用 `Mail` facade 的 `queue` 方法來將郵件訊息加入隊列：

    Mail::to($request->user())
        ->cc($moreUsers)
        ->bcc($evenMoreUsers)
        ->queue(new OrderShipped($order));

此方法會自動將工作加入隊列，以便在背景發送郵件訊息。當然，在使用此功能前需要[設定你的隊列](/laravel_tw/docs/5.5/queues)。

#### 延遲的訊息隊列

如果希望延遲發送已加入隊列的電子郵件訊息，可以使用 `later` 方法。`later` 方法的第一個數接受 `DateTime` 實例來指定訊息應該被送出的時間。

    $when = Carbon\Carbon::now()->addMinutes(10);

    Mail::to($request->user())
        ->cc($moreUsers)
        ->bcc($evenMoreUsers)
        ->later($when, new OrderShipped($order));

#### 加入特定隊列

因為所有 `make:mail` 指令產生的 mailable 類別都可以利用 `Illuminate\Bus\Queueable` trait，你可以呼叫任何 mailable 類別實例的 `onQueue` 和 `onConnection` 方法，讓你可以指定訊息的連線和隊列名稱：

    $message = (new OrderShipped($order))
                    ->onConnection('sqs')
                    ->onQueue('emails');

    Mail::to($request->user())
        ->cc($moreUsers)
        ->bcc($evenMoreUsers)
        ->queue($message);

#### 預設加入隊列

如果有總是要加入隊列的 mailable 類別，可以在此類別上實作 `ShouldQueue` contract。現在即使在寄送郵件時呼叫 `send` 方法，此 mailable 仍然會被加入隊列，因為已實作此 contract，：

    use Illuminate\Contracts\Queue\ShouldQueue;

    class OrderShipped extends Mailable implements ShouldQueue
    {
        //
    }

<a name="mail-and-local-development"></a>
## 郵件與本機開發

當開發需要寄送郵件的應用程式時，你可能不想實際送出郵件到真正的電子郵件地址。Laravel 提供了幾種方法以在本機開發時「停止」將郵件訊息真正寄出。

#### 日誌驅動

`log` 郵件驅動會將所有的電子郵件訊息寫入日誌檔案，而不是發送郵件，以作為檢驗之用。若需要更多根據環境來設定應用程式的資訊，可參考[設定文件](/laravel_tw/docs/5.5/configuration#environment-configuration)。

#### 通用收件者

另一個 Lavavel 提供的解決方案是為框架寄出的所有電子郵件設定一個通用的收件者。如此一來，應用程式產生的所有電子郵件都會被送到一個特定的位址，而不是寄信時實際指定的收件人位址。可以透過 `config/mail.php` 設定檔的 `to` 選項來完成：

    'to' => [
        'address' => 'example@example.com',
        'name' => 'Example'
    ],

#### Mailtrap

最後，你可以使用像 [Mailtrap](https://mailtrap.io) 的服務以及 `smtp` 驅動來將郵件訊息寄到一個「假的」郵箱，而你可以在一個真的郵件客戶端檢視它們。這個方法的好處是讓你可以在 Mailtrap 的訊息檢閱器中實際查看最終的電子郵件。

<a name="events"></a>
## 事件

Laravel 會在發送郵件訊息的過程中觸發兩個事件。`MessageSending` 事件是在訊息被送出前觸發，`MessageSent` 則是在訊息送出後被觸發。切記，此事件只會在郵件發送時*觸發*，在隊列時不會。可以在 `EventServiceProvider` 註冊一個事件監聽器：

    /**
     * 應用程式的事件監聽器對應。
     *
     * @var array
     */
    protected $listen = [
        'Illuminate\Mail\Events\MessageSending' => [
            'App\Listeners\LogSendingMessage',
        ],
        'Illuminate\Mail\Events\MessageSent' => [
            'App\Listeners\LogSentMessage',
        ],
    ];
