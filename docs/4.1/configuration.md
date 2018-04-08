---
layout: post
title: configuration
tag: 4.1
---
# 設定

- [簡介](#introduction)
- [環境設定](#environment-configuration)
- [Provider Configuration](#provider-configuration)
- [Protecting Sensitive Configuration](#protecting-sensitive-configuration)
- [維護模式](#maintenance-mode)

<a name="introduction"></a>
## 簡介

所有關於 Laravel 框架的設定文件都被放置在 `app/config` 目錄下。每個文件裡的所有選項都有文件，因此你可以輕鬆地察看這些文件，並且熟悉這些選項配置。

有時候，你可能在運行時需要存取這些設定值，你可以使用 `Config` 類別：

**存取一個選項的值**

	Config::get('app.timezone');

如果選項值不存在，你可以指定一個預設值：

	$timezone = Config::get('app.timezone', 'UTC');


**設定選項值**

注意，"點"式語法可以用來存取不同設定文件裡的選項。你還可以在運行階段更改設定值:

	Config::set('database.default', 'sqlite');

在運行階段設定的選項值只在該次請求中有效，不會對其他的請求造成影響。

<a name="environment-configuration"></a>
## 環境配置

通常應用程式常常需要根據不同的運行環境而有不同的配置設定值。例如，你會希望在你的本地開發機器上會有與正式環境不同的緩存驅動（cache driver），透過設定檔案，這是非常容易達成的。

在 `config` 目錄下建立與環境名稱相同的目錄，例如 `local`。接下來，創建你想要覆寫的設定文件，並且設定該環境所希望的設定值。例如，你要在 `app/config/local` 建立 `cache.php` 檔案，內容如下：

	<?php

	return array(

		'driver' => 'file',

	);

> **註:** 請勿使用 'testing' 當作環境名稱，它是專門為單元測試保留的。

注意，你不需要為基本設定文件中的_所有_選項設定選項值，只需要指定你需要覆蓋的配置選項即可。環境配置文件將會以 "cascade" 的方式覆蓋基本設定文件。

接下來，我們需要讓框架知道如何確認其運行環境。預設環境是 `production`。然而，你可以在安裝目錄下的 `bootstrap/start.php` 文件中設定其他環境。在該文件中，你可以找到 `$app->detectEnvironment` 函式。該陣列將會用來偵測當前的運行環境。你可以根據你的需求增加環境或者是機器名稱。

    <?php

    $env = $app->detectEnvironment(array(

        'local' => array('your-machine-name'),

    ));

在這個範例中，'local' 是運行環境的名稱而 'your-machine-name' 是你的服務器的主機名稱。在 Linux 和 Mac 上，你可以透過命令列執行 `hostsname` 查到你的主機名稱。

如果你想要更靈活的環境偵測方式，可以傳遞一個 `閉包（Closure）` 給 `detectEnvironment` 函式，這樣你就可以按照你想要的方式偵測了：

	$env = $app->detectEnvironment(function()
	{
		return $_SERVER['MY_LARAVEL_ENV'];
	});

**存取目前的運行環境**

你也可以透過 `environment` 函式來取的目前運行階段的環境：

	$environment = App::environment();

你也可以傳遞參數至 `environment` 函式中，來確認目前的環境是否與參數相符合：

	if (App::environment('local'))
	{
		// 當環境為 local 時
	}

	if (App::environment('local', 'staging'))
	{
		// 環境為 local 或 staging
	}

<a name="provider-configuration"></a>
### Provider Configuration

When using environment configuration, you may want to "append" environment [service providers](/docs/ioc#service-providers) to your primary `app` configuration file. However, if you try this, you will notice the environment `app` providers are overriding the providers in your primary `app` configuration file. To force the providers to be appended, use the `append_config` helper method in your environment `app` configuration file:

	'providers' => append_config(array(
		'LocalOnlyServiceProvider',
	))

<a name="protecting-sensitive-configuration"></a>
## Protecting Sensitive Configuration

For "real" applications, it is advisable to keep all of your sensitive configuration out of your configuration files. Things such as database passwords, Stripe API keys, and encryption keys should be kept out of your configuration files whenever possible. So, where should we place them? Thankfully, Laravel provides a very simple solution to protecting these types of configuration items using "dot" files.

First, [configure your application](/docs/configuration#environment-configuration) to recognize your machine as being in the `local` environment. Next, create a `.env.local.php` file within the root of your project, which is usually the same directory that contains your `composer.json` file. The `.env.local.php` should return an array of key-value pairs, much like a typical Laravel configuration file:

	<?php

	return array(

		'TEST_STRIPE_KEY' => 'super-secret-sauce',

	);

All of the key-value pairs returned by this file will automatically be available via the `$_ENV` and `$_SERVER` PHP "superglobals". You may now reference these globals from within your configuration files:

	'key' => $_ENV['TEST_STRIPE_KEY']

Be sure to add the `.env.local.php` file to your `.gitignore` file. This will allow other developers on your team to create their own local environment configuration, as well as hide your sensitive configuration items from source control.

Now, On your production server, create a `.env.php` file in your project root that contains the corresponding values for your production environment. Like the `.env.local.php` file, the production `.env.php` file should never be included in source control.

> **Note:** You may create a file for each environment supported by your application. For example, the `development` environment will load the `.env.development.php` file if it exists.

<a name="maintenance-mode"></a>
## 維護模式

當你的應用程式處於維護模式時，所有的路由都會指向一個自定的視圖。當你要更新或進行維護作業時，“關閉”整個網站是很簡單的。`App::down` 函式已經定義在你的 `app/start/global.php` 檔案中。他將會在你的應用程式處於維護模式時將執行該函式，展現在用戶前。

啟用維護模式，只要執行 Artisan 指令 'down'：

	php artisan down

關閉維護模式，只要執行 Artisan 指令 'up'：

	php artisan up

如果你想要客製化維護模式的視圖，你只需要增加下面內容至應用程式裡的 `app/start/global.php` 檔案中：

	App::down(function()
	{
		return Response::view('maintenance', array(), 503);
	});

如果傳給 `down` 函式的閉包回傳 'NULL' 值，該此請求將會略過維護模式。

### 維護模式與隊列

當應用程式處於維護模式中，將不會處理任何[隊列工作](/docs/queues)。所有的隊列工作將會在應用程式離開維護模式後繼續被進行。
