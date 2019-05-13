---
layout: post
title: lifecycle
tag: 5.5
---
# 請求的生命週期

- [介紹](#introduction)
- [生命週期的概述](#lifecycle-overview)
- [把焦點放到服務提供者上](#focus-on-service-providers)

<a name="introduction"></a>
## 介紹

在「真實世界」中使用任何工具時，如果能夠理解工具運作的原理，會更加有自信心。開發應用程式這件事也不例外。當你理解開發工具的功能時，使用起來會更加得心應手。

本文件的目的在於為你提供更進階的 Laravel 框架運作原理。當你愈是瞭解整體框架，就越能降低「莫名其妙」的感覺，並且會讓你更有信心建構應用程式。如果你不明白所有的專業術語，別太難過！只要嘗試對眼前不懂的地方有個基本的理解，你的知識會隨著你瀏覽文件的其他章節時跟著成長。

<a name="lifecycle-overview"></a>
## 生命週期的概述

### 第一件事

`public/index.php` 這個檔案是對 Laravel 應用程式所有請求的進入點。所有請求都會透過你的網頁伺服器（Apache 或 Nginx）設定。`index.php` 檔案並沒有放入太多程式碼。更確切地說，它只是一個載入框架其他部分的起點。

`index.php` 檔案會去載入 Composer 產生的自動載入的定義，並接收一個從 `bootstrap/app.php` 腳本中的 Laravel 應用程式實例。Laravel 本身的第一個動作就是建立一個應用程式 / [服務容器](/laravel_tw/docs/5.5/container)的實例

### HTTP / 終端核心

接下來，會根據進入應用程式的請求類型將傳入的請求轉發到 HTTP 核心或控制器核心。這兩個核心是所有請求流動的中心位置。所以現在，我們就只要把焦點放在 HTTP 核心，而它的檔案就在 `app/Http/Kernel.php`。

HTTP 核心繼承了 `Illuminate\Foundation\Http\Kernel` 類別，這類別定義了一組在請求被處理前所要執行的 `bootstrappers` 陣列。這些啟動器設置了錯誤處理、設定記錄、[偵測應用程式環境](/laravel_tw/docs/5.5/configuration#environment-configuration)，並在請求被實際處理之前，先去執行其他需要被完成的任務。

HTTP 核心也定義了所有請求在被應用程式處理之前必須經過的 HTTP [中介層](/laravel_tw/docs/5.5/middleware) 清單。這些中介層會去處理 [HTTP session](/laravel_tw/docs/5.5/session) 的讀取與寫入，以及確認應用程式是否處於維護模式，還有[驗證 CSRF token](/laravel_tw/docs/5.5/csrf) 等等。

HTTP 核心的 `handle` 方法的署名方式是相當簡單的。接收一個 `Request`，並回傳一個 `Response`。就把核心想像成一個能代表整個應用程式的大黑盒。只要將 HTTP 請求丟進去，它就會丟回給你一個 HTTP 回應。

#### 服務提供者

最重要的核心啟動行為之一，就是載入應用程式的[服務提供者](/laravel_tw/docs/5.5/providers)。應用程式的所有服務提供者都被設定在 `config/app.php` 設定檔的 `providers` 陣列。首先，會去呼叫所有提供者的 `register` 方法，等到所有提供者都被註冊完，接著會呼叫  `boot` 方法。

服務提供者負責在啟動時載入所有的框架各式元件，像是資料庫、隊列、驗證和路由元件。也因為服務提供者啟動載入並設定框架提供的每個功能，所以會是整個 Laravel 啟動載入過程中最重要的一個環節。

#### 調度請求

應用程式一旦被啟動載入並註冊了所有的服務提供者，那麼 `Request` 就會被轉交給路由器來進行調度。路由器會將請求調度到路由或控制器，並執行任何特定的路由中介層。

<a name="focus-on-service-providers"></a>
## 把焦點放到服務提供者上

服務提供者是啟動 Laravel 應用程式的真正關鍵。先是建立應用程式實例，接著註冊服務提供者，最後將請求轉移到已啟動的應用程式。過程就是如此簡單！

深入了解 Laravel 應用程式是如何建構並透過服務提供者啟動是相當有意義的。當然，你的應用程式的預設服務提供者就被存放在 `app/Providers` 目錄中。

預設的 `AppServiceProvider` 內容幾乎是空的。這個提供者是新增應用程式的本身啟動和綁定服務容器的好地方。當然，對於大型應用程式來說，你可能會希望建立好幾個服務提供者，並為每個服務提供者建立更細緻的啟動類型。
