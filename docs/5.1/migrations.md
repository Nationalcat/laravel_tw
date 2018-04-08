---
layout: post
title: migrations
tag: 5.1
---
# 資料庫: 遷移

- [簡介](#introduction)
- [產生遷移](#generating-migrations)
- [遷移結構](#migration-structure)
- [執行遷移](#running-migrations)
    - [還原遷移](#rolling-back-migrations)
- [編寫遷移](#writing-migrations)
    - [建立資料表](#creating-tables)
    - [重新命名與刪除資料表](#renaming-and-dropping-tables)
    - [建立欄位](#creating-columns)
    - [修改欄位](#modifying-columns)
    - [移除欄位](#dropping-columns)
    - [建立索引](#creating-indexes)
    - [移除索引](#dropping-indexes)
    - [外鍵約束](#foreign-key-constraints)

<a name="introduction"></a>
## 簡介

遷移是一種資料庫的版本控制，讓團隊能夠輕鬆的修改跟共享應用程式的資料庫結構。遷移通常會搭配 Laravel 的結構建構器，讓你可以輕鬆的建構應用程式的資料庫結構。

Laravel 的 `Schema` [facade](/laravel_tw/docs/5.1/facades) 提供了在資料庫建立和操作資料表的相關支援。它對所有 Laravel 所支援的資料庫系統共用了同樣一目了然、流暢的 API。

<a name="generating-migrations"></a>
## 產生遷移

可以使用 `make:migration` [Artisan 指令](/laravel_tw/docs/5.1/artisan) 建立遷移：

    php artisan make:migration create_users_table

新的遷移檔將會放置在 `database/migrations` 目錄中。每個遷移檔名稱都包含了一個時間戳記，讓 Laravel 能夠確認遷移的順序。

`--table` 和 `--create` 選項可用來指定資料表的名稱，或是該遷移會建立新的資料表。這些選項只需預先在產生遷移建置檔案時填入指定的資料表：

    php artisan make:migration add_votes_to_users_table --table=users

    php artisan make:migration create_users_table --create=users

如果你想為產生的遷移指定一個自定的輸出路徑，你可以在執行 `make:migration` 指令時使用 `--path` 選項。提供的路徑必須相對於你應用程式的基本路徑。

<a name="migration-structure"></a>
## 遷移結構

一個遷移類別會包含兩個方法：`up` 和 `down`。`up` 方法用於在資料庫內增加新的資料表表、欄位、或索引，而 `down` 方法則必須簡單的反向執行 `up` 方法的操作。

這兩個方法中你可以使用 Laravel 結構建構器明確的建立及修改資料表。若要瞭解`結構`建構器中所有可用的方法，[請查閱它的文件](#creating-tables)。例如：讓我們瞧瞧建立一張 `flights` 資料表的例子：

    <?php

    use Illuminate\Database\Schema\Blueprint;
    use Illuminate\Database\Migrations\Migration;

    class CreateFlightsTable extends Migration
    {
        /**
         * 執行遷移。
         *
         * @return void
         */
        public function up()
        {
            Schema::create('flights', function (Blueprint $table) {
                $table->increments('id');
                $table->string('name');
                $table->string('airline');
                $table->timestamps();
            });
        }

        /**
         * 還原遷移。
         *
         * @return void
         */
        public function down()
        {
            Schema::drop('flights');
        }
    }


<a name="running-migrations"></a>
## 執行遷移

要執行你應用程式中所有未完成的遷移，可以使用 `migrate` Artisan 指令。如果你使用 [Homestead 虛擬主機](/laravel_tw/docs/5.1/homestead)，你應該在你的虛擬機器中執行下方的指令：

    php artisan migrate

如果在你執行時出現「class not found」的錯誤，請試著在執行 `composer dump-autoload` 指令後再次執行一次遷移指令。

#### 在上線環境強制執行遷移

一些遷移的操作是有破壞性的，意思是它們可能會導致你失去資料。為了保護你上線環境的資料庫執行這些指令，你會在這些指令被執行之前，系統將會提示你進行確認。若要忽略提示強制執行指令，你可以使用 `--force` 標記：

    php artisan migrate --force

<a name="rolling-back-migrations"></a>
### 還原遷移

若要還原遷移至上一個「操作」，你可以使用 `rollback` 指令。請注意，此還原是回復到上一次執行的「批量」遷移，其中可能包括多筆的遷移檔案：

    php artisan migrate:rollback

`migrate:reset` 指令會還原應用程式的所有遷移：

    php artisan migrate:reset

#### 單個指令還原或執行遷移

`migrate:refresh` 指令首先會還原你資料庫的所有遷移，接著再執行 `migrate` 指令。此指令能有效的重新建立整個資料庫：

    php artisan migrate:refresh

    php artisan migrate:refresh --seed

<a name="writing-migrations"></a>
## 撰寫遷移

<a name="creating-tables"></a>
### 建立資料表

要建立一張新的資料表，可以使用 `Schema` facade 的 `create`方法。`create` 方法接收兩個參數。第一個參數為資料表的名稱，第二個參數為一個`閉包`，它接收一個用於定義新資料表的 `Blueprint` 物件：

    Schema::create('users', function (Blueprint $table) {
        $table->increments('id');
    });

當然，當建立資料表時，你可以使用任何的結構建構器的[欄位方法](#creating-columns)來定義資料表的欄位。

#### 檢查資料表或欄位是否存在

您可以使用 `hasTable` 和 `hasColumn` 方法簡單的檢查資料表或欄位是否存在：

    if (Schema::hasTable('users')) {
        //
    }

    if (Schema::hasColumn('users', 'email')) {
        //
    }

#### 連接與儲存引擎

如果你想要在一個非預設的資料庫連接進行結構操作，可以使用 `connection` 方法：

    Schema::connection('foo')->create('users', function ($table) {
        $table->increments('id');
    });

若要設置資料表的儲存引擎，只要在結構建構器上設置 `engine` 屬性：

    Schema::create('users', function ($table) {
        $table->engine = 'InnoDB';

        $table->increments('id');
    });

<a name="renaming-and-dropping-tables"></a>
### 重新命名與刪除資料表

若要重新命名一張已存在的資料表，可以使用 `rename` 方法：

    Schema::rename($from, $to);

要刪除已存在的資料表，你可使用 `drop` 或 `dropIfExists` 方法：

    Schema::drop('users');

    Schema::dropIfExists('users');

<a name="creating-columns"></a>
### 建立欄位

若要更新一張已存在的資料表，我們會使用 `Schema` facade 的 `table` 方法。如同 `create` 方法，`table` 方法接收兩個參數：資料表的名稱，及一個接收 `Blueprint` 實例的`閉包`，我們可以使用它為資料表增加欄位：

    Schema::table('users', function ($table) {
        $table->string('email');
    });

#### 可用的欄位類型

當然，結構建構器包含許多欄位類型，供你建構資料表時使用：

指令  | 描述
------------- | -------------
`$table->bigIncrements('id');`  |  遞增的 ID（主鍵），使用相當於「UNSIGNED BIG INTEGER」的型態。
`$table->bigInteger('votes');`  |  相當於 BIGINT 型態。
`$table->binary('data');`  |  相當於 BLOB 型態。
`$table->boolean('confirmed');`  | 相當於 BOOLEAN 型態。
`$table->char('name', 4);`  | 相當於 CHAR 型態，並帶有長度。
`$table->date('created_at');`  |  相當於 DATE 型態。
`$table->dateTime('created_at');`  |  相當於 DATETIME 型態。
`$table->decimal('amount', 5, 2);`  |  相當於 DECIMAL 型態，並帶有精度與基數。
`$table->double('column', 15, 8);`  |  相當於 DOUBLE 型態，總共有 15 位數，在小數點後面有 8 位數。
`$table->enum('choices', ['foo', 'bar']);` | 相當於 ENUM 型態。
`$table->float('amount');`  |  相當於 FLOAT 型態。
`$table->increments('id');`  |  遞增的 ID (主鍵)，使用相當於「UNSIGNED INTEGER」的型態。
`$table->integer('votes');`  |  相當於 INTEGER 型態。
`$table->json('options');`  |  相當於 JSON 型態。
`$table->jsonb('options');`  |  相當於 JSONB 型態。
`$table->longText('description');`  |  相當於 LONGTEXT 型態。
`$table->mediumInteger('numbers');`  |  相當於 MEDIUMINT 型態。
`$table->mediumText('description');`  |  相當於 MEDIUMTEXT 型態。
`$table->morphs('taggable');`  |  加入整數 `taggable_id` 與字串 `taggable_type`。
`$table->nullableTimestamps();`  |  與 `timestamps()` 相同，但允許 NULL。
`$table->rememberToken();`  |  加入 `remember_token` 使用 VARCHAR(100) NULL。
`$table->smallInteger('votes');`  |  相當於 SMALLINT 型態。
`$table->softDeletes();`  |  加入 `deleted_at` 欄位於軟刪除使用。
`$table->string('email');`  |  相當於 VARCHAR 型態。
`$table->string('name', 100);`  |  相當於 VARCHAR 型態，並帶有長度。
`$table->text('description');`  |  相當於 TEXT 型態。
`$table->time('sunrise');`  |  相當於 TIME 型態。
`$table->tinyInteger('numbers');`  |  相當於 TINYINT 型態。
`$table->timestamp('added_on');`  |  相當於 TIMESTAMP 型態。
`$table->timestamps();`  |  加入 `created_at` 和 `pdated_at` 欄位。
`$table->uuid('id');`  |  相當於 UUID 型態。

#### 欄位修飾

除了上述的欄位類型列表，還有一些其它的欄位「修飾」，你可以將它增加至欄位。例如，若要讓欄位「nullable」，那麼你可以使用 `nullable` 方法：

    Schema::table('users', function ($table) {
        $table->string('email')->nullable();
    });

以下列表為欄位可用的修飾。此列表不包括[索引修飾](#creating-indexes)：

修飾  | 描述
------------- | -------------
`->first()`  |  將此欄位放置在資料表的「第一個」（僅限 MySQL）
`->after('column')`  |  將此欄位放置在其他欄位「之後」（僅限 MySQL）
`->nullable()`  |  此欄位允許寫入 NULL 值
`->default($value)`  |  為此欄位指定「預設」值
`->unsigned()`  |  設置 `integer` 欄位為 `UNSIGNED`

<a name="changing-columns"></a>
<a name="modifying-columns"></a>
### 修改欄位

#### 先決條件

在修改欄位之前，務必在你的 `composer.json` 的增加 `doctrine/dbal` 依賴。Doctrine DBAL 函式庫被用於判斷目前的欄位狀態及建立調整指定欄位的 SQL 查詢。

#### 更新欄位屬性

`change` 方法讓你可以修改一個已存在欄位的類型，或修改欄位的屬性。例如，你可以想增加字串欄位的長度。要看看 `change` 方法的作用，讓我們將 `name` 欄位的長度從 25 增加到 50：

    Schema::table('users', function ($table) {
        $table->string('name', 50)->change();
    });

我們也能將欄位修改為 nullable：

    Schema::table('users', function ($table) {
        $table->string('name', 50)->nullable()->change();
    });

<a name="renaming-columns"></a>
#### 重新命名欄位

要重新命名欄位，你可使用結構建構器的 `renameColumn` 方法。在重新命名欄位前，請確定你的 `composer.json` 檔案內已經加入 `doctrine/dbal` 依賴：

    Schema::table('users', function ($table) {
        $table->renameColumn('from', 'to');
    });

> **注意：**資料表的 `enum` 欄位目前尚未支援修改欄位名稱。

<a name="dropping-columns"></a>
### 移除欄位

要移除欄位，可使用結構建構器的 `dropColumn` 方法：

    Schema::table('users', function ($table) {
        $table->dropColumn('votes');
    });

你可以傳遞欄位名稱的陣列至 `dropCloumn` 方法，移除多筆欄位：

    Schema::table('users', function ($table) {
        $table->dropColumn(['votes', 'avatar', 'location']);
    });

> **注意：**在 SQLite 資料庫中移除欄位前，你需要增加 `doctrine/dbal` 依賴至你的 `composer.json` 檔案，並在你的終端機執行 `composer update` 指令安裝該函式庫。

> **注意：**當使用 SQLite 資料庫時並不支援在單行遷移中移除或修改多筆欄位。

<a name="creating-indexes"></a>
### 建立索引

結構建構器支援多種類型的索引。首先，讓我們看看一個指定欄位的值必須是唯一的例子。要建立索引，我們可以簡單的在欄位定義之後鏈結上 `unique` 方法：

    $table->string('email')->unique();

此外，你可以在定義完欄位之後建立索引。例如：

    $table->unique('email');

你也可以傳遞一個欄位的陣列至索引方法來建立複合索引：

    $table->index(['account_id', 'created_at']);

#### 可用的索引類型

指令  | 描述
------------- | -------------
`$table->primary('id');`  |  加入主鍵。
`$table->primary(['first', 'last']);`  |  加入複合鍵。
`$table->unique('email');`  |  加入唯一索引。
`$table->index('state');`  |  加入基本索引。

<a name="dropping-indexes"></a>
### 移除索引

若要移除索引，你必須指定索引的名稱。預設中，Laravel 會自動分配合理的名稱至索引。簡單地連結這些資料表名稱，索引的欄位名稱，及索引類型。舉例如下：

指令  | 描述
------------- | -------------
`$table->dropPrimary('users_id_primary');`  |  從「users」資料表移除主鍵。
`$table->dropUnique('users_email_unique');`  |  從「users」資料表移除唯一索引。
`$table->dropIndex('geo_state_index');`  |  從「geo」資料表移除基本索引。

<a name="foreign-key-constraints"></a>
### 外鍵約束

Laravel 也為建立外鍵約束提供支援，這是用於在資料庫層面強制參考完整性。例如，讓我們定義 `posts` 資料表內有個 `user_id` 欄位要參考至 `users` 資料表的 `id` 欄位：

    Schema::table('posts', function ($table) {
        $table->integer('user_id')->unsigned();

        $table->foreign('user_id')->references('id')->on('users');
    });

你也可以指定約束的「on delete」及「on update」作為所需的操作：

    $table->foreign('user_id')
          ->references('id')->on('users')
          ->onDelete('cascade');

要移除外鍵，你可以使用 `dropForeign` 方法。外鍵約束與索引採用相同的命名方式。所以，我們可以連結資料表明與約束的欄位，接著在該名稱加上「_foreign」後綴：

    $table->dropForeign('posts_user_id_foreign');
