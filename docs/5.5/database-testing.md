---
layout: post
title: database-testing
tag: 5.5
---
# 資料庫測試

- [介紹](#introduction)
- [產生 Factory](#generating-factories)
- [在每次測試後重置資料庫](#resetting-the-database-after-each-test)
- [寫入 Factory](#writing-factories)
    - [Factory 狀態](#factory-states)
- [使用 Factory](#using-factories)
    - [建立模型](#creating-models)
    - [保存模型](#persisting-models)
    - [關聯](#relationships)
- [可用的斷言](#available-assertions)

<a name="introduction"></a>
## 介紹

Laravel 提供了各種有用的工具，可以輕鬆的測試資料庫驅動。首先，你可以使用 `assertDatabaseHas` 輔助函式來判資料庫是否存在與指定條件相互匹配的資料。例如，如果想要驗證在 `user` 資料表中是否存在 `sally@example.com` 的 `email` 值，你可以執行以下操作來測試：

    public function testDatabase()
    {
        // 進行呼叫應用程式...

        $this->assertDatabaseHas('users', [
            'email' => 'sally@example.com'
        ]);
    }

你還可以使用 `assertDatabaseMissing` 輔助函式來斷言資料是否不存在於資料庫。

當然，`assertDatabaseHas` 方法和其他輔助函式一樣方便。你可以自由的使用任何 PHPUnit 的內建斷言方法來完善你的測試。

<a name="generating-factories"></a>
## 產生 Factory

使用 [Artisan 指令](/laravel_tw/docs/5.5/artisan) 的 `make:factory` 來產生 Factory ：

    php artisan make:factory PostFactory

新的 Factory 會放在你的 `database/factories` 目錄。

`--model` 選項可用於指示由 Factory 建立的模型名稱。這個選項會使用給定的模型來預先填充產生的 Factory 檔案。

    php artisan make:factory PostFactory --model=Post

<a name="resetting-the-database-after-each-test"></a>
## 在每次測試後重置資料庫

在每次測試完就重置資料庫，這樣有助於之前測試用的資料不會影響下一次測試。`RefreshDatabase` trait 是根據你使用記憶體資料庫或傳統資料庫來遷移你的測試資料庫的最佳方式。只需使用 trait 在你的測試類別上，並為你處理所有事情：

    <?php

    namespace Tests\Feature;

    use Tests\TestCase;
    use Illuminate\Foundation\Testing\RefreshDatabase;
    use Illuminate\Foundation\Testing\WithoutMiddleware;

    class ExampleTest extends TestCase
    {
        use RefreshDatabase;

        /**
         * 一個基本的功能測試範例。
         *
         * @return void
         */
        public function testBasicExample()
        {
            $response = $this->get('/');

            // ...
        }
    }

<a name="writing-factories"></a>
## 寫入 Factory

在測試時，你可能需要在測試前對資料庫寫入幾筆記錄。建立這個測試資料的時候不用手動指定每列的值，因為 Laravel 可以讓你使用模型 Factory 來為每個 [Eloquent 模型](/laravel_tw/docs/5.5/eloquent)定義一組預設的屬性。開始之前請查看在你應用程式的 database/factories/UserFactory.php。這個檔案已經包含了一個現成的 Factory 定義：

    use Faker\Generator as Faker;

    $factory->define(App\User::class, function (Faker $faker) {
        static $password;

        return [
            'name' => $faker->name,
            'email' => $faker->unique()->safeEmail,
            'password' => $password ?: $password = bcrypt('secret'),
            'remember_token' => str_random(10),
        ];
    });

在作為 Factory 定義的閉包中，你可以回傳模型上所有屬性的預設測試值。閉包會接收一個 [Faker](https://github.com/fzaninotto/Faker) PHP 函式庫的實例，這讓你方便的產生各種隨機資料來測試。

你也可以為了更好組織每個模型而去建立額外的 Factory 檔案。例如，你應該建立 `UserFactory.php` 和 `CommentFactory.php` 檔案在你的 `database/factories` 目錄中。 Factory 目錄中的所有檔案會自動被 Laravel 載入。

<a name="factory-states"></a>
###  Factory 狀態

Factory 狀態讓你可以定義分散的修改，以任何組合的形式應用於你的模型 Factory。例如，你的 `User` 模型或可能有一個 `delinquent` 狀態，修改了它的一個預設屬性值。你可以使用 `state` 方法來定義你的狀態變化。你可以傳入要修改的屬性陣列來簡單的改變狀態：

    $factory->state(App\User::class, 'delinquent', [
        'account_status' => 'delinquent',
    ]);

如果你的狀態需要計算或是一個 `$faker` 實例，你可以使用閉包來計算狀態的屬性修改：

    $factory->state(App\User::class, 'address', function ($faker) {
        return [
            'address' => $faker->address,
        ];
    });

<a name="using-factories"></a>
## 使用 Factory

<a name="creating-models"></a>
### 建立模型

一旦你定義了 Factory ，你可以在你的測試或是 seed 檔案使用全域的 `factory` 函式來產生模型實例。所以，讓我們來看幾個建立模型的範例。首先，我們會使用 `make` 來建立模型，但不會儲存它們到資料庫：

    public function testDatabase()
    {
        $user = factory(App\User::class)->make();

        // 在測試中使用模型...
    }

你也可以建立多個模型集合或建立給定類型的模型：

    // 建立三個 App\User 實例...
    $users = factory(App\User::class, 3)->make();

#### 應用 Factory 狀態

你也可以應用你的任何[狀態](#factory-states)給模型。如果你想要模型的應用有多種狀態變化，你應該指定要應用的每個狀態名稱：

    $users = factory(App\User::class, 5)->states('delinquent')->make();

    $users = factory(App\User::class, 5)->states('premium', 'delinquent')->make();

#### 覆寫屬性

如果你想要覆寫模型中的某些預設值，你可以將一組陣列值傳到 `make` 方法。只有指定的值會被替換，而剩下的值將維持 Factory 指定的預設值來設定：

    $user = factory(App\User::class)->make([
        'name' => 'Abigail',
    ]);

<a name="persisting-models"></a>
### 保存模型

`create` 方法不只建立模型實例，還會使用 Eloquent 的 `save` 方法來儲存它們到資料庫：

    public function testDatabase()
    {
        // 建立一個 App\User 實例...
        $user = factory(App\User::class)->create();

        // 建立三個 App\User 實例...
        $users = factory(App\User::class, 3)->create();

        // 在測試中使用模型...
    }

你可以在模型上傳入一組陣列到 `create` 方法來覆寫屬性：

    $user = factory(App\User::class)->create([
        'name' => 'Abigail',
    ]);

<a name="relationships"></a>
### 關聯

在這個範例中，我們會嘗試關聯某些已建好的模型。當使用 `create` 方法來建立多型模型時，會回傳 Eloquent 的[集合實例](/laravel_tw/docs/5.5/eloquent-collections)，可以讓你使用集合提供的任何方便的功能，像是 `each`：

    $users = factory(App\User::class, 3)
               ->create()
               ->each(function ($u) {
                    $u->posts()->save(factory(App\Post::class)->make());
                });

#### 關聯與屬性閉包

你也可以在 Factory 定義中使用閉包屬性來關聯模型。例如，如果你想要在建立 `Post` 模型的時候建立新的 `User` 實例，你可以執行以下操作：

    $factory->define(App\Post::class, function ($faker) {
        return [
            'title' => $faker->title,
            'content' => $faker->paragraph,
            'user_id' => function () {
                return factory(App\User::class)->create()->id;
            }
        ];
    });

這些閉包也接收定義它們的 Factory 陣列所要的屬性：

    $factory->define(App\Post::class, function ($faker) {
        return [
            'title' => $faker->title,
            'content' => $faker->paragraph,
            'user_id' => function () {
                return factory(App\User::class)->create()->id;
            },
            'user_type' => function (array $post) {
                return App\User::find($post['user_id'])->type;
            }
        ];
    });

<a name="available-assertions"></a>
## 可用的斷言

Laravel 為你 [PHPUnit](https://phpunit.de/) 測試提供了幾個資料庫斷言：

方法  | 說明
------------- | -------------
`$this->assertDatabaseHas($table, array $data);`  |  斷言資料庫中的資料表是否有給定的資料。
`$this->assertDatabaseMissing($table, array $data);`  |  斷言資料庫中的資料表是否沒有給定的資料。
`$this->assertSoftDeleted($table, array $data);`  |  斷言給定的記錄是否被軟刪除。
