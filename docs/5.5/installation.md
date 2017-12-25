---
layout: post
title: installation
---
# 安裝

- [安裝](#installation)
    - [系統需求](#server-requirements)
    - [安裝 Laravel](#installing-laravel)
    - [設定](#configuration)
- [Web 伺服器設定](#web-server-configuration)
    - [修飾 URLs](#pretty-urls)

<a name="installation"></a>
## 安裝

> {video} 你是位視覺化學習者？ [Laracasts](http://laravelfromscratch.com) 為入門 Laravel 框架的新手，提供了免費、深入的介紹。這裡是能夠帶你入門這套框架的好地方。

<a name="server-requirements"></a>
### 系統需求

Laravel 需要一些基本的系統需求。當然，這些基本需求都可以在 [Laravel Homestead](/docs/{{version}}/homestead) 虛擬機器環境內被滿足，十分推薦你使用 Homestead 作為你本機的 Laravel 開發環境。

不過，如果你不使用 Homestead，則需要確保你的伺服器符合以下的要求：

<div class="content-list" markdown="1">
- PHP >= 7.0.0
- OpenSSL PHP Extension
- PDO PHP Extension
- Mbstring PHP Extension
- Tokenizer PHP Extension
- XML PHP Extension
</div>

<a name="installing-laravel"></a>
### 安裝 Laravel

Laravel 使用了 [Composer](https://getcomposer.org) 來管理套件的相依性。所以，在使用 Laravel 之前，確保你已經在你的機器上安裝了 Composer。

#### 方法一：使用 Laravel Installer

首先，使用 Composer 下載 Laravel installer：

    composer global require "laravel/installer"

請確定把 `$HOME/.composer/vendor/bin` 目錄（實際的目錄路徑依據你的作業系統可能有所不同） 放置於環境變數 $PATH 裡，這樣你的系統才能夠找到並正確執行 `laravel` 這個指令。

一旦安裝完畢，可以使用 `laravel new` 建立全新的 Laravel 專案至指定的目錄。例如：`laravel new blog` 會建立名稱為 `blog` 的目錄，裡面包含新安裝的 Laravel 專案和相依程式碼：

    laravel new blog

#### 方法二：透過 Composer Create-Project

或者，你也可以透過 Composer 在命令列執行 `create-project` 指令安裝 Laravel ：

    composer create-project --prefer-dist laravel/laravel blog

#### 本地開發環境伺服器

如果你已經在本地端安裝好 PHP ，並且想要使用 PHP 內建的開發環境伺服器來啟用你的應用程式，可以透過 Artisan 指令 `serve` 。這個指令會啟動本地開發環境伺服器，你可以透過 `http://localhost:8000` 在本地端訪問:

    php artisan serve

當然，更健全的開發環境選項還是透過 [Homestead](/docs/{{version}}/homestead) 和 [Valet](/docs/{{version}}/valet)。

<a name="configuration"></a>
### 設定

#### Public 目錄

在安裝完 Laravel 後，你需要將你的網站伺服器根目錄指向 public 目錄。該目錄下的 index.php 程式將作為前端控制器，所有的 HTTP 請求都會透過它進入至你的應用程式。

#### 設定檔

所有 Laravel 框架的設定檔都位於 `config` 目錄下。每一個設定的選項均有詳細的說明，因此你可以輕鬆地瀏覽這些文件，並且熟悉這些選項及配置。

#### 目錄權限

安裝 Laravel 後，你必須對一些權限進行設定。目錄 `storage` 及 `bootstrap/cache` 內的子目錄必須是讓網頁伺服器可寫的，否則 Laravel 就無法正常執行。若是使用 [Homestead](/docs/{{version}}/homestead) 虛擬機器，這些權限預設已經被設定完成。

#### 應用程式金鑰

在安裝完 Laravel 後，必須做的下一步就是設定一組隨機字串作為應用程式金鑰。假設你是透過 Composer 或是 Laravel 安裝工具安裝 Laravel ，那麼這組應用程式金鑰已經透過 `php artisan key:generate` 指令幫你設定完成。

通常，這組金鑰應該有 32 字元的長度。這組金鑰可以在 `.env` 環境設定檔中設定。若你還沒將 `.env.example` 檔案重新命名成 `.env` ，那你應該現在就做。**如果應用程式金鑰沒有被設定的話，你的使用者 sessions 和其他加密的資料都是不安全的！**

#### 其他設定

Laravel 幾乎不需設定就可以馬上使用，你可以自由自在的開始開發！不過，你或許會想看看 `config/app.php` 檔案和相關的文件。其中包含了數個設定選項，像是 `timezone` 和 `locale`，你可以根據你的應用程式來做修改。

你可能也會想要設定一些 Laravel 附加的元件，像是：

<div class="content-list" markdown="1">
- [快取](/docs/{{version}}/cache#configuration)
- [資料庫](/docs/{{version}}/database#configuration)
- [Session](/docs/{{version}}/session#configuration)
</div>

<a name="web-server-configuration"></a>
## Web 伺服器設定

<a name="pretty-urls"></a>
### 修飾 URLs

#### Apache

Laravel 內包含了一個 `public/.htaccess` 檔案用於讓 URLs 路徑不帶有前端控制器 `index.php`。使用 Apache 部署 Laravel 前，請確認是否將 `mod_rewrite` 模組啟用，這樣 `.htaccess` 檔案才能夠正確的被伺服器正確解析。

如果 Laravel 專案內預先搭載的 `.htaccess` 檔案在你的 Apache 環境下不能使用，你可以嘗試這個方法：

    Options +FollowSymLinks
    RewriteEngine On

    RewriteCond %{REQUEST_FILENAME} !-d
    RewriteCond %{REQUEST_FILENAME} !-f
    RewriteRule ^ index.php [L]

#### Nginx

若你是使用 Nginx，可以透過在你的網站設定內加入以下的指令來導向所有的請求至 `index.php` 前端控制器：

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

當然，如果你是使用 [Homestead](/docs/{{version}}/homestead) 或 [Valet](/docs/{{version}}/valet)，修飾 URLs 功能已經自動地被設定完成。
