---
layout: post
title: configuration
tag: 4.0
---
# 設定

- [介紹](#introduction)
- [環境設定](#environment-configuration)
- [維護模式](#maintenance-mode)

<a name="introduction"></a>
## 介紹

Laravel 的所有設定存放在 `app/config` 目錄中，檔案中的每一個選項都有做成文件供參考，可以翻閱文件選擇適合你熟悉的選項

有時你或許需要在執行時能存取到這些設定，你可以使用 `Config` 這個類別去存取設定:

**存取一個選項設定值**

	Config::get('app.timezone');

當這個選項值沒有被設定時，你或許可以指定一個預設的值當作預設選項:

	$timezone = Config::get('app.timezone', 'UTC');

注意到 "點(dot)" 的語法格式可以用來存取不同的檔案，你可以在執行時設定選項值為:

**設定一個選項值**

	Config::set('database.default', 'sqlite');

設定一個選項值僅會在當次請求中生效，並不會再之後的請求中生效。

<a name="environment-configuration"></a>
## 環境設定

不同的設定值，在不同的正在執行的應用環境，往往是有幫助的，舉例來說，你可以希望在 本地開發機器 與產品伺服器 使用不同的快取機制，這使用執行環境(environment)的配置的設定後，是很容易做到的。

簡單的在 `config` 目錄建立與執行環境(environment)相符的檔案名稱，像是 `local`，下一步建立你想要在這個環境覆蓋過去的設定，舉例來說，為了複寫本地端環境的快取機制，你可以在 `app/config/local` 建立 `cache.php` 的檔案，檔案內容如下:

	<?php

	return array(

		'driver' => 'file',

	);

> `注意:` 不要使用 'testing' 當作環境名稱，這個是保留給單元測試用

over the base files.請注意，你不需要在設定檔中指定 _每一個_ 選項，你只需要指定你想覆蓋的選項名稱即可，環境設置將會使用瀑布流(cascade)的方式去複寫這些在原始檔案的設定

接下來我們必須告訴框架，需要執行哪一個環境(environment)，預設的環境都是為 `production`，但是你或許安裝前在 `bootstrap/start.php` 檔案中設定其他的執行環境，在這個檔案中你將會找到 `$app->detectEnvironment` 的呼叫，這個陣列傳遞給這個方法去決定現在要執行的環境，你可能添加需要的其他環境及機器名稱。

    <?php

    $env = $app->detectEnvironment(array(

        'local' => array('your-machine-name'),

    ));

在這個範例中，'local' 是運行環境的名稱而 'your-machine-name' 是你的服務器的主機名稱。在 Linux 和 Mac 上，你可以透過命令列執行 `hostsname` 查到你的主機名稱。

您也可以通過一個封閉的detectEnvironment方法，讓你實現你自己的環境檢測： 你也可以透過一個 `封閉(Closure)` 的 `detectEnvironment` 方法，讓你完成你自己的環境檢測。

你可以透過 `environment` 方法存取現在的應用環境:

**存取現在的應用環境**

	$environment = App::environment();

<a name="maintenance-mode"></a>
## 維護模式

當你的應用程式在維護模式 (maintenance mode)，應用程式中所有的路由將會顯示一個自訂的 view ，呼叫 `app/start/global.php` 檔案中的 `App::down` 可以讓你容易的在你需要更新時，"關閉" 你的應用程式，這個回應會傳送給所有使用者，告知你的應用程式正在維護中。

開啟維護模式 (maintenance mode) 只需要在 Artisan 指令中執行 `down` 指令即可:

	php artisan down

關閉維護模式 (maintenance mode)時，使用 `up` 指令即可:

	php artisan up

為了在維護模式 (maintenance mode) 時顯示一個自訂的 view，你可以在 `app/start/global.php` 檔案中加入下列程式:

	App::down(function()
	{
		return Response::view('maintenance', array(), 503);
	});
