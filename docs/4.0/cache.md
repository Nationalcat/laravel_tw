---
layout: post
title: cache
tag: 4.0
---
# 快取

- [設定](#configuration)
- [使用快取](#cache-usage)
- [遞增及遞減](#increments-and-decrements)
- [快取區段](#cache-sections)
- [資料庫快取](#database-cache)

<a name="configuration"></a>
## 設定

Laravel 為不同的快取系統提供一個統一的 API，快取的設置在 `app/config/cache.php`。檔案中，在這個檔案，你可以指定你的應用程式，預設要使用哪一種快取驅動程式，Laravel 也支援一些常見的快取應用，像[Memcached](http://memcached.org) 及 [Redis](http://redis.io)。

快取設定檔案也包含其他的設定項目，這些項目說明是寫在快取設定檔中，所以要確定讀過這些項目的說明再行設定，Laravel 預設使用 `file` 的快取驅動程式，將快取資料序列化的存在檔案系統內，對於較大的應用程式，建議你使用記憶體式的快取系統，像是 Memcached 或 APC。

<a name="cache-usage"></a>
## 使用快取

**儲存資料至快取**

	Cache::put('key', 'value', $minutes);

**如果快取中無此資料，則儲存資料至快取**

	Cache::add('key', 'value', $minutes);

**檢查是否有此快取資料**

	if (Cache::has('key'))
	{
		//
	}

**從快取中讀取資料**

	$value = Cache::get('key');

**從快取中讀取資料，若無資料回傳預設值**

	$value = Cache::get('key', 'default');

	$value = Cache::get('key', function() { return 'default'; });

**儲存一份永久的資料至快取**

	Cache::forever('key', 'value');

有時你會想要從快取中讀取資料，但如果在快取沒有此份資料時，儲存此資料的預設值至快取，你可以使用 `Cache::remember` 方法去完成這件事:

	$value = Cache::remember('users', $minutes, function()
	{
		return DB::table('users')->get();
	});

你也可以合併 `remember` 和 `forever` 這兩種方法:

	$value = Cache::rememberForever('users', function()
	{
		return DB::table('users')->get();
	});

注意到，所有儲存到快取的資料都是序列式的，所以你可以自由的儲存所有類型的資料至快取。

**從快取中移除資料**

	Cache::forget('key');

<a name="increments-and-decrements"></a>
## 遞增及遞減

在所有驅動程式中，除了 `file` 及 `database` 這兩種驅動程式外，都支援 `increment` 及 `decrement` 這兩種操作:

**遞增資料值**

	Cache::increment('key');

	Cache::increment('key', $amount);

**遞減資料值**

	Cache::decrement('key');

	Cache::decrement('key', $amount);

<a name="cache-sections"></a>
## 快取區段#

> **注意:** 當使用 `file` 及 `database` 這兩種驅動程式時，快取不支援區段功能 (Cache sections)。

快取區段 (Cache sections) 允許你在快取中群組化相關的資料，及清除整個區段的資料，可以使用 `section` 方法去存取一個快取區段。:

**存取一個快取區段**

	Cache::section('people')->put('John', $john, $minutes);

	Cache::section('people')->put('Anne', $anne, $minutes);

你也可以使用 `increment` 及 `decrement` 的方法，去存取快取區段的資料:

**存取一個快取區段的資料**

	$anne = Cache::section('people')->get('Anne');

你可以清除整個快取區段的資料:

	Cache::section('people')->flush();

<a name="database-cache"></a>
## 資料庫快取

當你使用 `database` 的快取驅動程式，你將要需要設定一個包含快取資料結構的資料表，下列是一個快取資料表範例的 `Schema` 定義:

	Schema::create('cache', function($table)
	{
		$table->string('key')->unique();
		$table->text('value');
		$table->integer('expiration');
	});
