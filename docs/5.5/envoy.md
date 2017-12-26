# Envoy Task Runner

- [介紹](#introduction)
    - [安裝](#installation)
- [撰寫任務](#writing-tasks)
    - [建立](#setup)
    - [變數](#variables)
    - [Story](#stories)
    - [多個伺服器](#multiple-servers)
- [執行任務](#running-tasks)
    - [確認任務執行](#confirming-task-execution)
- [通知](#notifications)
    - [Slack](#slack)

<a name="introduction"></a>
## 介紹

[Laravel Envoy](https://github.com/laravel/envoy) 提供了簡潔、輕量的語法，定義在遠端伺服器執行的共同任務。使用 Blade 風格的語法，你可以簡單的設置部署任務，執行 Artisan 指令或是更多。目前，Envoy 只支援 Mac 及 Linux 作業系統。

<a name="installation"></a>
### 安裝

首先，使用 Composer 的 `global require` 指令安裝 Envoy：

    composer global require laravel/envoy

由於全域的 Composer 函式庫有時會導致套件版本衝突，你或許會考慮使用 `cgr`，這可直接替換 `composer global require` 指令。`cgr` 程式庫的安裝介紹能在 [GitHub 上找到](https://github.com/consolidation-org/cgr)。

> {note} 確認 `~/.composer/vendor/bin` 目錄有放置在 PATH 中，以便在終端機執行 `envoy` 指令時可以找到可執行的 `envoy`。

#### 更新 Envoy

你也可以使用 Composer 來維持最新的 Envoy。使用 `composer global update` 指令會更新所有 Composer 全域安裝的套件：

    composer global update

<a name="writing-tasks"></a>
## 撰寫任務

你所有的 Envoy 任務必須定義在專案根目錄的 `Envoy.blade.php` 檔案中。這裡有個範例可以幫助你瞭解：

    @servers(['web' => ['user@192.168.1.1']])

    @task('foo', ['on' => 'web'])
        ls -la
    @endtask

如你所見，`@servers` 的陣列被定義在檔案的起始，讓你可以在宣告任務時，在 `on` 選項裡參照這些伺服器。在你的 `@task` 宣告裡，你必須放置當任務執行時想要在遠端伺服器執行的 Bash 程式碼。

你可以將伺服器的 IP 指定為 `127.0.0.1` 來強制腳本在本機執行：

    @servers(['localhost' => '127.0.0.1'])

<a name="setup"></a>
### 建立

有時，你可能想在執行任務前執行一些 PHP 程式碼。你可以使用 ```@setup``` 指令在 Envoy 檔案裡宣告變數及執行一般的 PHP 程式：

    @setup
        $now = new DateTime();

        $environment = isset($env) ? $env : "testing";
    @endsetup

如果你在執行任務之前需要其他 PHP 檔案，則可以在 `Envoy.blade.php` 檔案的最上方使用 `@include` 指令：

    @include('vendor/autoload.php')

    @task('foo')
        # ...
    @endtask

<a name="variables"></a>
### 變數

如果有需要，你可以使用指令列來將可選的值傳入 Envoy 任務：

    envoy run deploy --branch=master

你可以透過 Blade 的 「echo」 語法來存取任務中的選項。當然，你也可以在任務中使用 `if` 語句和迴圈。例如，讓我們執行 `git pull` 指令之前，先來驗證 `$branch` 變數的存在：

    @servers(['web' => '192.168.1.1'])

    @task('deploy', ['on' => 'web'])
        cd site

        @if ($branch)
            git pull origin {% raw %} {{ $branch }} {% endraw %}
        @endif

        php artisan migrate
    @endtask

<a name="stories"></a>
### Story

Story 會通過一個單一便捷的名稱來設定一組任務，使你合併既小又重要的任務。例如，`deploy` story 會在定義中去監聽任務名稱並執行 `git` 和 `composer` 任務：

    @servers(['web' => '192.168.1.1'])

    @story('deploy')
        git
        composer
    @endstory

    @task('git')
        git pull origin master
    @endtask

    @task('composer')
        composer install
    @endtask

一旦 Story 已便寫完畢，你可以執行它就像一般任務一樣：

    envoy run deploy

<a name="multiple-servers"></a>
### 多個伺服器

Envoy 可以讓你輕鬆的在多個伺服器上執行。首先，增加額外的伺服器至你的 `@server` 宣告。每個伺服器必須分配一個唯一的名稱。一旦你已經定義好額外的伺服器，就會在任務宣告的 `on` 陣列中列出這些伺服器：

    @servers(['web-1' => '192.168.1.1', 'web-2' => '192.168.1.2'])

    @task('deploy', ['on' => ['web-1', 'web-2']])
        cd site
        git pull origin {% raw %} {{ $branch }} {% endraw %}
        php artisan migrate
    @endtask

#### 平行執行

預設的任務會在每個伺服器上連續執行。也就是說，一個任務必須在第一個伺服器完成執行後，才會在第二台伺服器上執行。如果你想在多個伺服器上同時執行任務，只要簡單的在任務宣告裡加上 `parallel` 選項：

    @servers(['web-1' => '192.168.1.1', 'web-2' => '192.168.1.2'])

    @task('deploy', ['on' => ['web-1', 'web-2'], 'parallel' => true])
        cd site
        git pull origin {% raw %} {{ $branch }} {% endraw %}
        php artisan migrate
    @endtask

<a name="running-tasks"></a>
## 執行任務

要執行 `Envoy.blade.php` 檔案中地義的任務或 Story，請先執行 Envoy 的執行指令，並傳入你想要執行的任務或 Story 名稱。Envoy 會執行該任務並顯示任務執行時的伺服器輸出。

    envoy run task

<a name="confirming-task-execution"></a>
### 確認任務執行

如果你希望在伺服器上執行給定任務之前提醒你確認，你應該將 `confirm` 指令新增到你的任務宣告中。這個選項對於不可逆的操作特別有幫助：

    @task('deploy', ['on' => 'web', 'confirm' => true])
        cd site
        git pull origin {% raw %} {{ $branch }} {% endraw %}
        php artisan migrate
    @endtask

<a name="notifications"></a>
<a name="hipchat-notifications"></a>
## 通知

<a name="slack"></a>
### Slack

執行每個任務後，Envoy 還支援發送通知給 [Slack](https://slack.com)。`@slack` 指令接受一個 Slack hook URL 和頻道名稱。你可以通過 Slack 控制面板中建立「傳入 WebHook」集成來存取你的 webhook URL。你應該把整個 webhook 網址傳給 @slack 指令：

    @finished
        @slack('webhook-url', '#bots')
    @endfinished

你可以提供下列其中一個頻道參數：

<div class="content-list" markdown="1">
- 發通知給一個頻道：`#channel`
- 發通知給一個使用者：`@user`
</div>
