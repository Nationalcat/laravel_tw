---
layout: post
title: database
tag: 4.0
---
# 基本資料庫用法

- [設置](#configuration)
- [執行查詢](#running-queries)
- [資料庫交易](#database-transactions)
- [存取連線](#accessing-connections)
- [查詢記錄](#query-logging)

<a name="configuration"></a>
## 設置

Laravel讓連線資料庫及執行查詢時變得相當的簡單，資料庫的相關設定存放在 `app/config/db.php` 檔案中，在這個檔案你可以定義你所有的資料庫連連線，並指定哪一個連線是預設的資料庫連線，所有支援的資料庫系統都寫在這個檔案中。

Laravel支援四個資料庫系統: MySQL 、 Postgres 、 SQLite 及 SQL Server。

<a name="running-queries"></a>
## 執行查詢

完成資料庫連線的設定後，你就可以使用 `DB` 類別進行資料庫的查詢了。

**執行 Select 語法**

	$results = DB::select('select * from users where id = ?', array(1));

`select` 方法都會回傳一個 `陣列 (array)` 的結果。

**執行 Insert 語法**

	DB::insert('insert into users (id, name) values (?, ?)', array(1, 'Dayle'));

**執行 Update 語法**

	DB::update('update users set votes = 100 where name = ?', array('John'));

**執行 Delete 語法**

	DB::delete('delete from users');

> **注意:** `update` 和 `delete` 語法將會回傳在這個操作中，共影響了幾筆資料的結果。

**執行一般語法**

	DB::statement('drop table users');

你可以使用 `DB::listen` 方法，去監聽查詢事件:

**監聽查詢事件**

	DB::listen(function($sql, $bindings, $time)
	{
		//
	});

<a name="database-transactions"></a>
## 資料庫交易

你可以使用 `transaction` 方法，去執行一組資料庫交易集合的操作語法:

	DB::transaction(function()
	{
		DB::table('users')->update(array('votes' => 1));

		DB::table('posts')->delete();
	});

<a name="accessing-connections"></a>
## 存取連線

你可以使用 `DB::connection` 方法，去使用數個不同的資料庫連線:

	$users = DB::connection('foo')->select(...);

你也可以使用 PDO 實例去存取資料:

	$pdo = DB::connection()->getPdo();

有時你可能需要重新連結資料庫:

	DB::reconnect('foo');

<a name="query-logging"></a>
## 查詢紀錄

預設的情況下， Laravel 會將目前 HTTP 請求中，所執行過的查詢紀錄存放在記憶體中，在一些情況下，像是增加 (Insert) 大量的資料時，這樣可能會造成應用程式使用過多多餘的記憶體資源，所以你可以使用 `disableQueryLog ` 方法去關閉紀錄查詢到記憶體的動作:

	DB::connection()->disableQueryLog();

如果要拿到執行的查詢陣列，可以使用 'getQueryLog` 方法:

       $queries = DB::getQueryLog();
