---
layout: post
title: commands
tag: 4.0
---
# Artisan 開發

- [介紹](#introduction)
- [建立指令](#building-a-command)
- [註冊指令](#registering-commands)
- [呼叫其他指令](#calling-other-commands)

<a name="introduction"></a>
## 介紹

除了 Artisan 提供的指令外，你也可以在你的應用程式中建立自訂的 Artisan 指令，你可以將你的自訂指令存放在 `app/commands` 目錄中，然而，只要 `composer.json` 檔案中的 "自動載入 (autoload)" 的部分有設定好，你也可以依照你的喜好放置自訂指令到不同的目錄下。

<a name="building-a-command"></a>
## 建立指令

### 產生類別

你可以在 Artisan 指令中使用 `command:make` 指令，讓你可以產生製作新指令的初始程式框架:

**產生新的 Artisan 指令類別**

	php artisan command:make FooCommand

預設產生的 Artisan 指令類別檔案會存放在 `app/commands` 目錄下，你也可以指定自訂的 "路徑 (path)" 或 "命名空間 (namespace)":

	php artisan command:make FooCommand --path=app/classes --namespace=Classes

### 撰寫 Artisan 指令

在 Artisan 命令類別寫好後，你應該要替類別加上 `name` 及 `description` 屬性，這些屬性資訊會在 `命令清單 (list)` 畫面中顯示。

`fire` 方法會在執行命令後被呼叫執行，你可以在 fire 方法中撰寫任何的指令處理邏輯。

### 參數 & 選項

可以透過 `getArguments` 及 `getOptions` 方法去取得任何你自訂的參數及選項，這兩個方法皆會回傳在 `命令清單 (list)` 畫面中顯示指令的陣列資料。

在定義 `參數 (arguments)` 時，定義參數值的陣列資料會呈現如下所示:

	array($name, $mode, $description, $defaultValue)

參數 `mode` 可以是 `InputArgument::REQUIRED` 或 `InputArgument::OPTIONAL` 中的其中任何一個。

在定義 `選項 (options)` 時，定義選項值的陣列資料會呈現如下所示:

	array($name, $shortcut, $mode, $description, $defaultValue)

對於 "選項 (options)" 來說，參數 `mode` 可以是 `InputOption::VALUE_REQUIRED` 、 `InputOption::VALUE_OPTIONAL` 、 `InputOption::VALUE_IS_ARRAY` 或 `InputOption::VALUE_NONE` 中的其中任何一個。

`VALUE_IS_ARRAY` 模式指的是，在呼叫指令時，該選項數值可以傳入多次:

	php artisan foo --option=bar --option=baz

`VALUE_NONE` 模式指的是該選項僅用來做 "切換 (switch)" 使用，不帶任何資料:

	php artisan foo --option

### 取得輸入的資料

當指令執行時，你的應用程式想必一定需要去接收參數及選項，你可以使用 `argument` 及 `option` 方法去接收參數及選項的資料:

**取得命令中指定的參數值**

	$value = $this->argument('name');

**取得所有參數值**

	$arguments = $this->argument();

**取得命令中指定的選項值**

	$value = $this->option('name');

**取得所有選項值**

	$options = $this->option();

### 撰寫輸出

你可以使用 `info` 、 `comment` 、 `question` 及 `error` 方法去輸出資料到命令列 (console)，這些方法會使用符合其用途的 ANSI 顏色的字做輸出。

**傳送 "資訊 (info)" 到命令列**

	$this->info('Display this on the screen');

**傳送 "錯誤 (error)" 訊息到命令列**

	$this->error('Something went wrong!');

### 詢問問題

你可以使用 `ask` 及 `confirm` 方法去提示使用者輸入資料:

**詢問使用者請求輸入資料**

	$name = $this->ask('What is your name?');

**詢問使用者請求輸入隱藏資料**

	$password = $this->secret('What is the password?');

**詢問使用者做資訊確認**

	if ($this->confirm('Do you wish to continue? [yes|no]'))
	{
		//
	}

你也可以指定預設值到 `confirm` 方法中，預設值必須為 `true` 或 `false`:

	$this->confirm($question, true);

<a name="registering-commands"></a>
## 註冊指令

在你完成自訂 Artisan 指令後，你需要使用 Artisan 指令進行指令的註冊，這樣才能被使用，這個通常是在 `app/start/artisan.php` 檔案中完成，在這個檔案中，你可以使用 `Artisan::add` 方法去註冊指令:

**註冊 Artisan 指令**

	Artisan::add(new CustomCommand);

如果你的命令是在應用程式中的 [IoC 容器](/docs/ioc)中註冊，你可以使用 `Artisan::resolve` 方法，讓 Artisan 可以使用該方法:

**註冊在 IoC 容器的指令**

	Artisan::resolve('binding.name');

<a name="calling-other-commands"></a>
## 呼叫其他指令

有時你可能會需要呼叫其他的指令，你可以使用 `call` 方法去呼叫執行其他的指令:

**呼叫其他指令**

	$this->call('command.name', array('argument' => 'foo', '--option' => 'bar'));
