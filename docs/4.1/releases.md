---
layout: post
title: releases
tag: 4.1
---
# 發行版本說明

- [Laravel 4.1](#laravel-4.1)

<a name="laravel-4.1"></a>
## Laravel 4.1

### 完整更動列表

此發行版本的完整更動列表，可以在版本 4.1 的安裝中命令列執行 `php artisan changes` 取得，或者瀏覽 [Github 更動檔](https://github.com/laravel/framework/blob/4.1/src/Illuminate/Foundation/changes.json)中了解。其中只記錄了該版本比較主要的強化功能和更動。

### 新的 SSH 元件

一個全新的 `SSH` 元件在此發行版本中登場。此功能讓你可以輕易的 SSH 至遠端伺服器並執行命令。更多資訊，可以參閱 [SSH 元件文件](/docs/ssh)。

新的 `php artisan tail` 指令就是使用這個新的 SSH 元件。更多的資訊，請參閱 `tail` [指令集文件](http://laravel.com/docs/ssh#tailing-remote-logs)。

### Boris In Tinker

如果您的系統支援 [Boris REPL](https://github.com/d11wtq/boris)，`php artisan thinker` 指令將會使用到它。系統中也必須先行安裝好 `readline` 和 `pcntl` 兩個 PHP 套件。如果你沒這些套件，從 4.0 之後將會使用到它。

### Eloquent 強化

Eloquent 新增了新的 `hasManyThrough` 關係鏈。想要了解更多，請參見 [Eloquent 文件](/docs/eloquent#has-many-through)。

一個新的 `whereHas` 方法也同時登場，他將允許[檢索基於關係模型的約束](/docs/eloquent#querying-relations)。

### 資料庫讀寫分離

Query Builder 和 Eloquent 目前透過資料庫層，已經可以自動做到讀寫分離。更多的資訊，請參考[文件](/docs/database#read-write-connections)。

### 隊列排序

隊列排序已經被支援，只要在 `queue:listen` 命令後將隊列以逗號分隔送出。

### 失敗隊列作業處理

現在隊列將會自動處理失敗的作業，只要在 `queue:listen` 後加上 `--tries` 即可。更多的失敗作業處理可以參見 [隊列文件](/docs/queues#failed-jobs)。

### 緩存標籤

緩存“區塊”已經被“標籤”取代。緩存標籤允許你將多個“標籤”指向同一個緩存物件，而且可以清空所有被指定某個標籤的所有物件。更多使用緩存標籤資訊請見[緩存文件](/docs/cache#cache-tags)。

### 更具彈性的密碼提醒

密碼提醒引擎已經可以提供更強大的開發彈性，如：認證密碼，顯示狀態訊息等等。使用強化的密碼提醒引擎，更多的資訊[請參閱文件](/docs/security#password-reminders-and-reset)。

### 強化路由引擎

Laravel 4.1 擁有一個完全重新編寫的路由層。API 一樣不變。然而與 4.0 相比，速度快上 100%。整個引擎大幅的簡化，且對於路由表達式的編譯大大減少對 Symfony Routing 的依賴。

### 強化 Session 引擎

此發行版本中，我們亦發佈了全新的 Session 引擎。如同路由增進的部分，新的 Session 曾更加簡化且更快速。我們不再使用 Symfony 的 Session 處理工具，並且使用更簡單、更容易維護的客製化解法。


### Doctrine DBAL

如果你有在你的遷移中使用到 `renameColumn`，之後你必須在 `composer.json` 裡加 `doctrine/dbal` 進相依套件中。此套件不再預設包含在 Laravel 之中。
