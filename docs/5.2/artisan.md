---
layout: post
title: artisan
---
# Artisan 指令列

- [介紹](#introduction)
- [撰寫指令](#writing-commands)
    - [指令結構](#command-structure)
- [指令的輸入與輸出](#command-io)
    - [定義預期的輸入](#defining-input-expectations)
    - [取得輸入](#retrieving-input)
    - [為輸入加上提示](#prompting-for-input)
    - [輸出至畫面](#writing-output)
- [註冊指令](#registering-commands)
- [使用程式碼來呼叫指令](#calling-commands-via-code)

<a name="introduction"></a>
## 介紹

Artisan 是 Laravel 裡的一個指令列介面的名稱。當你在開發你的應用程式時，它提供了許多有用的指令來幫助你開發。它是基於由強大的 Symfony Console 元件來的。你可以使用 `list` 指令來列出所有可以使用的 Artisan 指令：

    php artisan list

每個指令也包含了「help」畫面，它會顯示並敘述指令可以使用的參數及選項。在指令前面加上 `help` 即可顯示：

    php artisan help migrate

<a name="writing-commands"></a>
## 撰寫指令

除了使用 Artisan 本身提供的指令外，你也可以建立自定的指令來進行作業。你可以將你的自定指令放在 `app/Console/Commands` 目錄下；當然，只要指令可以基於 `composer.json` 的設定自動載入，你可以自由選擇想要放置的地方。

要建立新的指令，可以使用 `make:console` Artisan 指令產生一個預設的腳本來作為起頭：

    php artisan make:console SendEmails

上面的這個指令會在 `app/Console/Commands/SendEmails.php` 建立一個類別。當建立指令時，`--command` 這個選項可以用來指定要使用的終端指令名稱：

    php artisan make:console SendEmails --command=emails:send

<a name="command-structure"></a>
### 指令結構

建立指令後，你應該填寫類別的 `signature` 和 `description` 這兩個屬性，它們會被顯示在 `list` 畫面：

當指令被執行的時候，`handle` 方法會被呼叫，因此你可以將任何的指令邏輯放置在這個方法中，讓我們來看看一個範例。

注意我們可以在建構子中注入任何需要的依賴，Laravel 的[服務容器](/laravel_tw/docs/5.2/container)將會自動注入任何型別提示的依賴到建構子中。為了更好的程式碼重用性，保持指令內容
輕量，並利用應用程式服務緩載來完成任務是一個好的實作方式。

    <?php

    namespace App\Console\Commands;

    use App\User;
    use App\DripEmailer;
    use Illuminate\Console\Command;

    class SendEmails extends Command
    {
        /**
         * 指令列的名稱及用法。
         *
         * @var string
         */
        protected $signature = 'email:send {user}';

        /**
         * 指令列的敘述。
         *
         * @var string
         */
        protected $description = 'Send drip e-mails to a user';

        /**
         * drip e-mail 服務。
         *
         * @var DripEmailer
         */
        protected $drip;

        /**
         * 創造新的指令實例。
         *
         * @param  DripEmailer  $drip
         * @return void
         */
        public function __construct(DripEmailer $drip)
        {
            parent::__construct();

            $this->drip = $drip;
        }

        /**
         * 執行指令。
         *
         * @return mixed
         */
        public function handle()
        {
            $this->drip->send(User::find($this->argument('user')));
        }
    }

<a name="command-io"></a>
## 指令的輸入與輸出

<a name="defining-input-expectations"></a>
### 定義預期的輸入

撰寫指令列時，經由參數或是選項取得使用者的輸入是很常見的。藉由使用指令的 `signature` 屬性，Laravel 讓你很方便的定義預期從使用者得到的輸入。`signature` 屬性允許你用單一、具表現力、與路由相似的語法，並用以定義指令的名字、參數及選項。

所有提供給使用者的參數及選項都在包在大括號中。如下方範例，此指令定義一個**必須的**參數：`user`：

    /**
     * 指令列的名稱及用法。
     *
     * @var string
     */
    protected $signature = 'email:send {user}';

你也可以使用讓參數是可選的並定義預設的值：

    // 可選的的參數...
    email:send {user?}

    // 可選的的參數及預設的值...
    email:send {user=foo}

選項，就跟參數一樣，也是一種使用者輸入，不過使用選項時，需要在選項名稱前加入兩個連字符號（`--`），我們可以在 signature 中這樣定義選項：

    /**
     * 指令列的名稱及用法。
     *
     * @var string
     */
    protected $signature = 'email:send {user} {--queue}';

在這個範例中，當呼叫 Artisan 指令時，`--queue` 這個選項可以被明確的指定。如果 `--queue` 被當成輸入時，這個選項的值會是 `true`，如果沒有指定時，這個選項的值將會是 `false`：

    php artisan email:send 1 --queue

你也可以在選項後面加上 `=`，表示選項需要明確指定值：

    /**
     * 指令列的名稱及用法。
     *
     * @var string
     */
    protected $signature = 'email:send {user} {--queue=}';

在這個範例中，使用者可以為這個選擇傳入一個值：

    php artisan email:send 1 --queue=default

你也可以指定預設值給選項：

    email:send {user} {--queue=default}

如果要在定義選項時給予簡寫，你可以在選項名稱之前指定簡寫，並使用 | 分隔符號將它與完整的選項名稱隔開：

    email:send {user} {--Q|queue}

如果你想定義預期輸入的參數或選項為陣列，可以使用 `*` 符號：

    email:send {user*}

    email:send {user} {--id=*}

#### 輸入的敘述

藉由加入冒號及敘述，你也可以為輸入參數及選項加上敘述：

    /**
     * 指令列的名稱及用法。
     *
     * @var string
     */
    protected $signature = 'email:send
                            {user : 使用者的 ID }
                            {--queue= : 這個工作是否該進入隊列}';

<a name="retrieving-input"></a>
### 取得輸入

執行指令時，你會需要取得接收的參數及選項。為了達到這個目的，你會需要使用 `argument` 及 `option` 方法：

    /**
     * 執行這個指令列。
     *
     * @return mixed
     */
    public function handle()
    {
        $userId = $this->argument('user');

        //
    }

如果你需要將所有的參數匯聚成一個`陣列`，只要不加參數呼叫 `argument` 即可：

    $arguments = $this->argument();

而取得選項就跟參數一樣簡單，除了使用的方法變為 `option`。就像 `argument` 方法一樣，你可以呼叫 `option` 不加任何參數，即可取得所有的選項並將之轉為一個`陣列`：

    // 取得特定的選擇
    $queueName = $this->option('queue');

    // 取得所有選擇
    $options = $this->option();

如果參數或選項不存在，將會回傳 `null`。

<a name="prompting-for-input"></a>
### 為輸入加上提示

除了顯示輸出，你也可以在指令執行期間，要求使用者輸入值。`ask` 方法將會用提供的問題來提示使用者，並且接受他們的輸入，接著回傳使用者的輸入回指令：

    /**
     * 執行這個指令列。
     *
     * @return mixed
     */
    public function handle()
    {
        $name = $this->ask('你是名字是?');
    }

`secret` 方法就如同 `ask` 方法一般，但是使用者的輸入將不會顯示在指令列。這個方法適合要求提供如密碼的敏感資訊時：

    $password = $this->secret('密碼是？');

#### 要求確認

如果你需要使用者做簡單的確認，你可以使用 `confirm` 方法。預設的情況時，這個方法會回傳 `false`。然而，如果使用者對這個提示輸入 `y`，那這個方法將會回傳 `true`：

    if ($this->confirm('你希望繼續嗎? [y|N]')) {
        //
    }

#### 讓使用者做選擇

`anticipate` 方法可被用於為可能的選擇提供自動完成。使用者仍可以選擇任何答案，不管這些選擇。

    $name = $this->anticipate('你的名字是?', ['Taylor', 'Dayle']);

如果你需要提供使用者一組事先定義的選擇，你可以使用 `choice` 方法。使用者會選擇答案的索引，但是回傳給你的會是答案的值。你可以設定回傳的預設值來防止沒有任何東西被選擇：

    $name = $this->choice('你的名字是?', ['Taylor', 'Dayle'], false);

<a name="writing-output"></a>
### 輸出至畫面

使用 `line`、`info`、`comment`、`question` 和 `error` 方法來傳送輸出到終端。每個方法都有適當的 ANSI 顏色來表達它們的目的。

使用 `info` 方法來傳送資訊訊息給使用者，並以綠色呈現在終端。

    /**
     * 執行這個指令列。
     *
     * @return mixed
     */
    public function handle()
    {
        $this->info('把我顯示在畫面上');
    }

使用 `error` 方法來傳送錯誤訊息給使用者，並以紅色呈現在終端。

    $this->error('有東西出問題了！');

如果你想顯示原本的控制列輸出，可以使用 `line` 方法。`line` 方法不會接收任何特殊的顏色：

    $this->line('把我顯示在畫面上');

#### 表格佈局

`table` 方法讓格式化多行與多列的資料變得簡單。只要傳送標頭和行到這個方法，寬跟高將會基於給的資料做動態的計算:

    $headers = ['Name', 'Email'];

    $users = App\User::all(['name', 'email'])->toArray();

    $this->table($headers, $users);

#### 進度條

對於需要長時間執行的任務，顯示進度指示器將會很有幫助。使用 output 物件，我們可以開始、前進、停止進度條，當開始執行時你需要定義總共有幾個階段，然後每階段完成後就讓進度條前進：

    $users = App\User::all();

    $bar = $this->output->createProgressBar(count($users));

    foreach ($users as $user) {
        $this->performTask($user);

        $bar->advance();
    }

    $bar->finish();

想得到更多資訊，請看看 [Symfony Progress Bar 元件的文件](http://symfony.com/doc/2.7/components/console/helpers/progressbar.html)。

<a name="registering-commands"></a>
## 註冊指令

一旦你的指令完成，你需要先向 Artisan 註冊它後才能使用。註冊的檔案為 `app/Console/Kernel.php`。

在這個檔案中，你會在 `commands` 屬性找到指令的清單。要註冊你的指令，只要簡單的在此清單加入類別的名稱。當 Artisan 啟動時，所有條列在這個屬性的指令，都會被[服務容器](/laravel_tw/docs/5.2/container)解析並向 Artisan 註冊：

    protected $commands = [
        Commands\SendEmails::class
    ];

<a name="calling-commands-via-code"></a>
## 使用程式碼呼叫指令

有時候你想在指令列介面外執行 Artisan 指令。例如，你希望在路由或控制器觸發 Artisan 指令。你只要在 `Artisan` facade 使用 `call` 方法做到。`call` 方法的第一個參數為指令的名稱，第二個參數為陣列型態的指令輸入。退出碼將會被回傳：

    Route::get('/foo', function () {
        $exitCode = Artisan::call('email:send', [
            'user' => 1, '--queue' => 'default'
        ]);

        //
    });

在 `Artisan` facade 使用 `queue` 方法，你甚至會將一堆 Artisan 指令放進隊列，好讓它們能在背景被你的[隊列作業器](/laravel_tw/docs/5.2/queues) 執行：

    Route::get('/foo', function () {
        Artisan::queue('email:send', [
            'user' => 1, '--queue' => 'default'
        ]);

        //
    });

如果你需要指定非接收字串選項的值，像是 `migrate:refresh` 指令的 `--force` 標記，你可以傳遞一個 `true` 或 `false` 的布林值：

    $exitCode = Artisan::call('migrate:refresh', [
        '--force' => true,
    ]);

### 在指令中呼叫其他指令

有時候，你會希望在指令中呼叫其他已存在的指令。你可以使用 `call` 方法來達成。`call` 方法接受指令名稱和指令參數的陣列：

    /**
     * 執行這個指令列。
     *
     * @return mixed
     */
    public function handle()
    {
        $this->call('email:send', [
            'user' => 1, '--queue' => 'default'
        ]);

        //
    }

如果你想要呼叫其他指令並忽視它所有的輸出，你可以使用 `callSilent` 指令。`callSilent` 方法有和 `call` 方法一樣的用法：

    $this->callSilent('email:send', [
        'user' => 1, '--queue' => 'default'
    ]);
