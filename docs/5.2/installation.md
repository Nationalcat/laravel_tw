---
layout: post
title: installation
---
# 安裝

- [安裝](#installation)
    - [伺服器需求](#server-requirements)
    - [安裝 Laravel](#installing-laravel)
    - [設定](#confiugration)

<a name="installation"></a>
## 安裝

<a name="server-requirements"></a>
### 伺服器需求

Laravel 框架有一些系統上的需求。當然，[Laravel Homestead](/laravel_tw/docs/5.2/homestead) 虛擬機器都能滿足這些需求，所以強烈的建議你使用 Homestead 作為本機 Laravel 開發環境。

不過，如果你不使用 Homestead，你會需要確保你的伺服器符合下列要求：

<div class="content-list" markdown="1">
- PHP >= 5.5.9
- OpenSSL PHP Extension
- PDO PHP Extension
- Mbstring PHP Extension
- Tokenizer PHP Extension
</div>

<a name="installing-laravel"></a>
### 安裝 Laravel

Laravel 使用 [Composer](http://getcomposer.org) 來管理相依性。所以，在使用 Laravel 之前，你必須確認電腦上是否安裝了 Composer。

#### 透過 Laravel 安裝工具

首先，使用 Composer 下載 Laravel 安裝包：

    composer global require "laravel/installer"

請確定把 `~/.composer/vendor/bin` 路徑放置於你的 PATH 裡，這樣你的系統才能找到 `laravel` 執行檔。

一旦安裝完成後，就可以使用 `laravel new` 指令在指定的目錄建立一份全新安裝的 Laravel。例如：`laravel new blog` 將會建立一個名稱為 `blog` 的目錄，裡面存放著全新安裝的 Laravel 和相依程式碼。這個安裝方法比透過 Composer 安裝速度快上許多：

    laravel new blog

#### 透過 Composer Create-Project

你也可以透過 Composer 在命令列執行 `create-project` 指令來安裝 Laravel：

    composer create-project --prefer-dist laravel/laravel blog

<a name="configuration"></a>
### 設定

所有 Laravel 框架的設定檔案都放置在 `config` 目錄下。每個選項都有說明，因此你可以輕鬆地瀏覽這些文件，並且熟悉這些選項配置。

#### 目錄權限

安裝 Laravel 之後，你必須設定一些權限。`storage` 和 `bootstrap/cache` 目錄中的目錄必須讓你的伺服器有寫入權限，否則 Laravel 就無法執行。如果你使用 [Homestead](/laravel_tw/docs/5.2/homestead) 虛擬機器，那麼這些權限應該已經被設定完成。

#### 應用程式金鑰

在你安裝完 Laravel 後，首先需要做的事情是設定一個隨機字串到應用程式金鑰。假設你是透過 Composer 或是 Laravel 安裝工具安裝 Laravel，那麼這個金鑰已經透過 `key:generate` 指令幫你設定完成。通常這個金鑰應該有 32 字元長。這個金鑰可以被設定在 `.env` 環境檔案中。如果你還沒將 `.env.example` 檔案重新命名為 `.env`，那麼你應該現在開始。**如果應用程式金鑰沒有被設定的話，你的使用者 Sessions 和其他的加密資料都是不安全的！**

#### 其他設定

Laravel 幾乎不需設定就可以馬上使用，你可以自由的開始開發！當然，你可以瀏覽 `config/app.php` 檔案和對應的文件。它包含數個選項，如`時區`和`語言環境`，你不仿根據你的應用程式來做修改。

你也可以設定 Laravel 的幾個附加元件，像是：

- [快取](/laravel_tw/docs/5.2/cache#configuration)
- [資料庫](/laravel_tw/docs/5.2/database#configuration)
- [Session](/laravel_tw/docs/5.2/session#configuration)

一旦 Laravel 安裝完成，你應該同時[設定本機環境](/laravel_tw/docs/5.2/configuration#environment-configuration)。
