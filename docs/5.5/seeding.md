# 資料庫: 資料填充

- [簡介](#introduction)
- [撰寫資料填充](#writing-seeders)
    - [使用模型工廠](#using-model-factories)
    - [呼叫其他的 Seeders](#calling-additional-seeders)
- [執行資料填充](#running-seeders)

<a name="introduction"></a>
## 簡介

Laravel 可以簡單的使用 seed 類別，填充測試用的資料至資料庫。所有的 seed 類別放在 database/seeds 目錄下。你可以任意地為 Seed 類別命名，但是應該遵守某些大小寫規範，像是 `UserTableSeeder` 之類。預設已經為你定義了一個  `DatabaseSeeder` 類別。在這個類別裡，你可以使用 `call` 方法執行其他的 seed 類別，藉此控制資料填充的順序。

<a name="writing-seeders"></a>
## 撰寫資料填充

你可以透過 `make:seeder` [Artisan 指令](/laravel_tw/docs/5.5/artisan) 來產生一個 Seeder。所有透過框架產生的 Seeder 都將被放置在 `database/seeders` 路徑：

    php artisan make:seeder UsersTableSeeder

在 seeder 類別裡只會預設一個方法：`run`。當執行 `db:seed` [Artisan 指令](/laravel_tw/docs/5.5/artisan) 時就會呼叫此方法。在 `run` 方法中，你可以新增任何想要的資料至你的資料庫中。你可使用 [查詢產生器](/laravel_tw/docs/5.5/queries) 手動新增資料或你也可以使用 [Eloquent 模型工廠](/laravel_tw/docs/5.5/database-testing#writing-factories)。

如同下面的範例，讓我們修改 Laravel 預設的 DatabaseSeeder 類別。我們加入一段新增的語句到 `run` 方法：

    <?php

    use Illuminate\Database\Seeder;
    use Illuminate\Database\Eloquent\Model;

    class DatabaseSeeder extends Seeder
    {
        /**
         * 執行資料庫填充。
         *
         * @return void
         */
        public function run()
        {
            DB::table('users')->insert([
                'name' => str_random(10),
                'email' => str_random(10).'@gmail.com',
                'password' => bcrypt('secret'),
            ]);
        }
    }

<a name="using-model-factories"></a>
### 使用模型工廠

當然，手動為每一個 seed 模型指定屬性是很繁瑣的。你可以使用[模型工廠](/laravel_tw/docs/5.5/database-testing#writing-factories)來作為替代，方便的產生大量的資料庫記錄。首先，閱讀[模型工廠的文件](/laravel_tw/docs/5.5/database-testing#writing-factories)來學習如何定義你的工廠。一旦你定義了你的工廠，你就可以使用 factory 這個輔助方法函式來新增記錄到資料庫中。

例如，讓我們建立 50 位使用者，並為每個使用者建立關聯：

    /**
     * 執行資料庫填充
     *
     * @return void
     */
    public function run()
    {
        factory(App\User::class, 50)->create()->each(function ($u) {
            $u->posts()->save(factory(App\Post::class)->make());
        });
    }

<a name="calling-additional-seeders"></a>
### 呼叫其他的 Seeder

在 `DatabaseSeeder` 類別，你可以使用 `call` 方法執行其他的 seed 類別。為避免發生單一個 seeder 類別變得壓倒性巨大的情況，使用 `call` 方法可以讓你把資料庫填充拆成多份資料填充，這樣就不會有單一的資料填充過大的情況。只需傳送你希望執行的 seeder 類別名稱：

    /**
     * 執行資料庫填充。
     *
     * @return void
     */
    public function run()
    {
        $this->call([
            UsersTableSeeder::class,
            PostsTableSeeder::class,
            CommentsTableSeeder::class,
        ]);
    }

<a name="running-seeders"></a>
## 執行資料填充

一旦你撰寫完你的 seeder 類別，可以使用 Artisan 的 `db:seed` 指令來對資料庫進行資料填充。預設的情況下，`db:seed` 指令將執行 DatabaseSeeder 類別，並透過它來呼叫其他的 seed 類別。但是，你也可以使用 `--class` 選項來單獨運行一個特別指定的 seeder 類別：

    php artisan db:seed

    php artisan db:seed --class=UsersTableSeeder

你也可以使用 `migrate:refresh` 指令來對資料庫進行資料填充，它會 rollback 並再次執行所有的遷移。在完全重建你的資料庫時這個指令是非常有用的：

    php artisan migrate:refresh --seed
