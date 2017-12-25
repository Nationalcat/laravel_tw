---
layout: post
title: artisan
---
# Artisan 指令列

- [介紹](#introduction)
- [撰寫指令](#writing-commands)
    - [產生指令](#generating-commands)
    - [指令結構](#command-structure)
    - [閉包指令](#closure-commands)
- [定義預期的輸入](#defining-input-expectations)
    - [參數](#arguments)
    - [選項](#options)
    - [輸入陣列](#input-arrays)
    - [輸入說明](#input-descriptions)
- [指令 I/O](#command-io)
    - [取得輸入](#retrieving-input)
    - [互動式輸入](#prompting-for-input)
    - [自訂輸出](#writing-output)
- [註冊指令](#registering-commands)
- [使用程式碼呼叫指令](#programmatically-executing-commands)
    - [在指令中呼叫其他指令](#calling-commands-from-other-commands)

<a name="introduction"></a>
## 介紹

Artisan 是 Laravel 內建的指令集合，它能提供許多好用的指令來協助你開發程式。你可以使用 `list` 查詢更多指令：

    php artisan list

每個指令都有輔助說明，會告訴你有哪些參數及選項可以用。在需要查詢的指令前加上 `help` 即可顯示輔助說明內容:

    php artisan help migrate

#### Laravel REPL

所有 Laravel 的應用程式都可以使用 Tinker，是基於 [PsySH](https://github.com/bobthecow/psysh) 這個套件所提供的 REPL。Tinker 可以直接操控你的整個 Laravel 應用程式，包括 Eloquent ORM、任務、事件等。 執行 `tinker` 這個指令，即可進入 Tinker 環境：

    php artisan tinker

<a name="writing-commands"></a>
## 撰寫指令

除了 Laravel 提供的原生指令外，你也可以自訂指令。預設檔案路徑是在 `app/Console/Commands`。然而，只要指令可以被 Composer 載入，那你就可以任意的選擇檔案路徑。

<a name="generating-commands"></a>
### 產生指令

要產生一個新指令，請使用 `make:command`。該指令會在 `app/Console/Commands` 這個目錄中建立檔案。如果你的 Laravel 應用程式中沒有這個目錄，別擔心！當你第一次使用 `make:command` 時，會即時建立該目錄。產生的指令會包括所有指令中預設的屬性與方法：

    php artisan make:command SendEmails

<a name="command-structure"></a>
### 指令結構

產生新的指令後，應該先宣告 `signature` 和 `description` 的屬性內容，這會在使用 `list` 這個指令的時候顯示出來。 當指令被執行時，`handle` 方法會被呼叫，因此你可以將任何的指令邏輯放到該方法中。

> {tip} 為了讓程式碼更有效的複用，最好讓終端指令的程式碼保持輕量化，並讓它們緩載到應用程式服務的任務完成。在下列範例中，請注意！我們注入了一個服務類別來完成發送信件的「重任」。

讓我們看一個例子。請注意，我們可以在建構子中注入任何需要的依賴，Laravel 的[服務容器](/docs/{{version}}/container)將會自動注入任何型別提示的依賴到建構子中。

    <?php

    namespace App\Console\Commands;

    use App\User;
    use App\DripEmailer;
    use Illuminate\Console\Command;

    class SendEmails extends Command
    {
        /**
         * 指令列的名稱及用法
         *
         * @var string
         */
        protected $signature = 'email:send {user}';

        /**
         * 指令列的描述
         *
         * @var string
         */
        protected $description = 'Send drip e-mails to a user';

        /**
         * The drip e-mail service.
         *
         * @var DripEmailer
         */
        protected $drip;

        /**
         * 建立新的指令實例
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
         * 執行指令
         *
         * @return mixed
         */
        public function handle()
        {
            $this->drip->send(User::find($this->argument('user')));
        }
    }

<a name="closure-commands"></a>
### 閉包指令

基於閉包的指令提供了有別於使用類別定義終端指令的方法。簡單的說，路由閉包是另一種撰寫指令的方式。在 `app/Console/Kernel.php` 檔案的 `commands` 這個方法中，Laravel 會載入 `routes/console.php` 這個檔案：

    /**
     * 為應用程式註冊基於閉包的指令
     *
     * @return void
     */
    protected function commands()
    {
        require base_path('routes/console.php');
    }

即使這個檔案沒有定義 HTTP 路由，它仍可以透過路由終端定義到應用程式中。在這個檔案中，你可以使用 `Artisan::command` 這個方法定義所有基於閉包的路由。`command` 方法可以接受兩個參數：其一是[指令命名](#defining-input-expectations)，另一個取得指令參數與選項的閉包：

    Artisan::command('build {project}', function ($project) {
        $this->info("Building {$project}!");
    });

因為閉包綁定最底層的指令實例，所以你完全可以使用指令類別的所有輔助方法

#### 型別提示

除了接收指令參數與選項外，指令閉包還可以使用型別提示從[服務容器](/docs/{{version}}/container)中注入所需的任何依賴：

    use App\User;
    use App\DripEmailer;

    Artisan::command('email:send {user}', function (DripEmailer $drip, $user) {
        $drip->send(User::find($user));
    });

#### 撰寫閉包指令的描述

當定義一個基於閉包的指令時，你可以使用 `describe` 方法來新增指令的描述。這個描述會在你執行 `php artisan list` 或 `php artisan help` 時顯示：

    Artisan::command('build {project}', function ($project) {
        $this->info("Building {$project}!");
    })->describe('Build the project');

<a name="defining-input-expectations"></a>
## 定義預期的輸入

在撰寫終端指令時，通常會透過參數或選項來取得使用者輸入的指令。 Laravel 可以非常方便的使用 `signature` 屬性來定義你預期用戶輸入的內容。`signature` 屬性可以給你使用單一且可讀性高，還有類似路由的語法來定義名稱、參數和選項。

<a name="arguments"></a>
### 參數

所有使用者輸入的參數與選項都會在大括號中。在接下來的範例中，這個指令定義了一個**必要的**參數：`user`：

    /**
     * 指令列的命名和用法
     *
     * @var string
     */
    protected $signature = 'email:send {user}';

你也可以建立可選參數，並定義參數的預設值：

    // 可選參數...
    email:send {user?}

    // 帶有預設值的可選參數...
    email:send {user=foo}

<a name="options"></a>
### 選項

選項很類似參數，是用戶輸入的另一種方式。當指令列指定選項時，它們以兩個字符（`--`）作為前綴。有兩種類型的選項：可接受值和不可接受值。不接受值的選項又可作為布林值的「開關」。讓我們看一下這種類型選項的例子：

    /**
     * 指令列的命名和用法
     *
     * @var string
     */
    protected $signature = 'email:send {user} {--queue}';

在這個例子中，可以在執行 Artisan 指令中加入 `--queue`，如果有輸入 `--queue`，那麼將會回傳 `true`。除此之外，則回傳 `false`：

    php artisan email:send 1 --queue

<a name="options-with-values"></a>
#### 附值的選項

接著，讓我們看一下某個選項所期望的值。如果使用者必須為選項指定一個值，只需要在名稱後面加入 `=` 符號：

    /**
     * 指令列的命名和用法
     *
     * @var string
     */
    protected $signature = 'email:send {user} {--queue=}';

在這個例子中，使用者可以為這個選項輸入想要的值：

    php artisan email:send 1 --queue=default

你可以在選項名稱後指定預設值。如果使用者沒有傳送選項的值，將會使用預設值：

    email:send {user} {--queue=default}

<a name="option-shortcuts"></a>
#### 選項的簡寫

你只需要簡單的在選項名稱之前指定簡寫，並使用 `|` 分隔符號將它與完整的選項名稱隔開：

    email:send {user} {--Q|queue}

<a name="input-arrays"></a>
### 輸入陣列

如果你想定義預期輸入的參數或選項為陣列，你可以使用 `*` 符號。首先，讓我們先看一下一個陣列參數的實例：

    email:send {user*}

當呼叫該方法時，可以使用 `user` 參數給指令列。例如，以下的指令會設置 `user` 為 `['foo', 'bar']`:

    php artisan email:send foo bar

當定義期望輸入的參數或選項的值為陣列時，應在要個別輸入選項值的前綴:

    email:send {user} {--id=*}

    php artisan email:send --id=1 --id=2

<a name="input-descriptions"></a>
### 輸入說明

你可以透過冒號為輸入的參數和選項個別說明如何使用。如果你需要一點額外的空間來定義你的指令，可以隨意撰寫多行：

    /**
     * 命令列的命名和用法
     *
     * @var string
     */
    protected $signature = 'email:send
                            {user : The ID of the user}
                            {--queue= : Whether the job should be queued}';

<a name="command-io"></a>
## 指令 I/O

<a name="retrieving-input"></a>
### 取得輸入

當你的指令執行時，想必需要處理指令的參數與選項，那麼你可以使用 `argument` 和 `option` 這兩個方法：

    /**
     * 執行指令
     *
     * @return mixed
     */
    public function handle()
    {
        $userId = $this->argument('user');

        //
    }

如果你需要取得所有參數作為一個 `array`，呼叫 `arguments` 方法：

    $arguments = $this->arguments();

取得選項就和取得參數一樣簡單，你只需要使用 `option` 方法。要取得陣列的全部，直接使用 `options` 方法且不用加入任何參數:

    // 取得特定選項...
    $queueName = $this->option('queue');

    // 取得全部選項...
    $options = $this->options();

如果參數或選項不存在，將會回傳 `null`。

<a name="prompting-for-input"></a>
### 互動式輸入

除了顯示輸出外，你也可以在指令在執行期間，要求使用者輸入東西。`ask` 方法將會提供問題來詢問使用者，並且等待回覆與回傳使用者輸入的東西給指令：

    /**
     * 執行終端指令
     *
     * @return mixed
     */
    public function handle()
    {
        $name = $this->ask('What is your name?');
    }

`secret` 方法使用起來很像 `ask` 方法，但是使用者輸入的內容並不會顯示在指令列上。這個方法適合要求使用者提供密碼或其他敏感資訊：

    $password = $this->secret('What is the password?');

#### 要求確認

如果你需要使用者做簡單的確認，你可以使用 `confirm` 方法。預設的情況下，這個方法會回傳 `false`。然而，使用者輸入 `y` 或 `yes`，那麼這個方法才會回傳 `true`。

    if ($this->confirm('Do you wish to continue?')) {
        //
    }

#### 自動補完

`anticipate` 方法能預測並補齊使用者可能想輸入的內容。使用者仍然可以選擇任何的選項，不管自動補完是否有提示：

    $name = $this->anticipate('What is your name?', ['Taylor', 'Dayle']);

#### 多選題

如果你希望提供使用者以選擇題作答，你可以使用 `choice` 方法。你還可以設置預設值來回應使用者回答問題以外的東西。

    $name = $this->choice('What is your name?', ['Taylor', 'Dayle'], $default);

<a name="writing-output"></a>
### 自訂輸出

使用 `line`、`info`、`comment`、`question` 和 `error` 方法來傳送輸出到終端。每個方法都有適合的 ANSI 顏色來表達他們的目的。例如，我們要傳送一般資訊給使用者，建議使用 `info` 方法，這將會回傳綠字給終端：

    /**
     * 執行終端指令
     *
     * @return mixed
     */
    public function handle()
    {
        $this->info('Display this on the screen');
    }

使用 `error` 方法可以回傳錯誤訊息給使用者，並以紅字呈現：

    $this->error('Something went wrong!');

如果你只想要單純輸出文字到終端，可以使用 `line` 方法:

    $this->line('Display this on the screen');

#### 表格佈局

`table` 方法可以更輕鬆地格式化多行多列的資料，只需要傳送標題與行給這個方法。寬與高會根據資料進行動態調整：

    $headers = ['Name', 'Email'];

    $users = App\User::all(['name', 'email'])->toArray();

    $this->table($headers, $users);

#### 進度條

對於長時間執行的任務，顯示進度條將會很有幫助。使用輸出物件，我們可以開始、前進和停止進度條。當開始執行時你需要定義總共有幾個階段，然後在給個階段完成後就讓進度條前進：

    $users = App\User::all();

    $bar = $this->output->createProgressBar(count($users));

    foreach ($users as $user) {
        $this->performTask($user);

        $bar->advance();
    }

    $bar->finish();

更多進階選項，請點閱 [Symfony Progress Bar component documentation](https://symfony.com/doc/2.7/components/console/helpers/progressbar.html)。

<a name="registering-commands"></a>
## 註冊指令

由於 `load` 方法呼叫了在你的終端 kernel 的 `command` 方法，所有 `app/Console/Commands` 目錄下的所有指令都將會自動註冊到 Artisan。實際上，你可以自由地呼叫 `load` 方法來掃描 Artisan 指令的其他目錄：

    /**
     * 註冊 Artisan 指令
     *
     * @return void
     */
    protected function commands()
    {
        $this->load(__DIR__.'/Commands');
        $this->load(__DIR__.'/MoreCommands');

        // ...
    }

你還可以藉由類別名稱寫入 `app/Console/Kernel.php` 檔案的 `$command` 屬性來手動註冊命令。當 Artisan 啟動時，該屬性中列出的所有指令將由[服務容器](/docs/{{version}}/container)解析並註冊到 Artisan 指令上：

    protected $commands = [
        Commands\SendEmails::class
    ];

<a name="programmatically-executing-commands"></a>
## 使用程式碼呼叫指令

有時候你希望從終端機介面外執行 Artisan 指令。例如，你希望能從控制器或路由觸發 Artisan 指令。你可以使用 `Artisan` facade 的 `call` 方法做到。`call` 方法的第一個參數為指令名稱，第二個參數為陣列型態的指令輸入。退出碼將會被回傳：

    Route::get('/foo', function () {
        $exitCode = Artisan::call('email:send', [
            'user' => 1, '--queue' => 'default'
        ]);

        //
    });

在 `Artisan` facade 上使用 `queue` 方法，可以將 Artisan 指令放入[隊列](/docs/{{version}}/queues) 處理。在使用此方法前，請先確認隊列的設定，在執行隊列：

    Route::get('/foo', function () {
        Artisan::queue('email:send', [
            'user' => 1, '--queue' => 'default'
        ]);

        //
    });

你也可以在 Artisan 指令後面選擇你要使用的隊列驅動或任務：

    Artisan::queue('email:send', [
        'user' => 1, '--queue' => 'default'
    ])->onConnection('redis')->onQueue('commands');

#### 傳遞陣列

如果你需要接收陣列的選項，則可以簡單地將陣列傳給選項：

    Route::get('/foo', function () {
        $exitCode = Artisan::call('email:send', [
            'user' => 1, '--id' => [5, 13]
        ]);
    });

#### 傳遞布林值

如果你需要指定非接收字串選項的值，像是 `migrate:refresh` 指令的 `--force` 標記，你可以傳遞 `true` 或 `false`：

    $exitCode = Artisan::call('migrate:refresh', [
        '--force' => true,
    ]);

<a name="calling-commands-from-other-commands"></a>
### 在指令中呼叫其他指令

有時候，你希望從指令中呼叫其他已存在的指令。你可以使用 `call` 方法。 `call` 方法接受指令名稱和指令參數的陣列：

    /**
     * 執行終端指令
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

如果你想要呼叫其它指令並呼列它所有的輸出，你可以使用 `callSilent` 方法。`callSilent`和 `call` 方法使用方式一樣：

    $this->callSilent('email:send', [
        'user' => 1, '--queue' => 'default'
    ]);
