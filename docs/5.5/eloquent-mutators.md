---
layout: post
title: eloquent-mutators
tag: 5.5
---
# Eloquent：修改器

- [介紹](#introduction)
- [存取器與修改器](#accessors-and-mutators)
    - [定義存取器](#defining-an-accessor)
    - [定義修改器](#defining-a-mutator)
- [日期修改器](#date-mutators)
- [轉換屬性](#attribute-casting)
    - [陣列和 JSON 轉換](#array-and-json-casting)

<a name="introduction"></a>
## 介紹

存取器和修改器可以讓你在模型實例上取得或設定它們的時候格式化 Eloquent 屬性的值。例如，你可能想要使用 [Laravel 加密器](/laravel_tw/docs/5.5/encryption)來對正要被儲存在資料庫的值加密，並且你在 Eloquent 模型上存取他時自動解密該屬性。

除了自訂的存取器和修改器外，Eloquent 也會自動將日期欄位型別轉換成 [Carbon](https://github.com/briannesbitt/Carbon) 實例或甚至[將文字欄位型別轉換成 JSON](#attribute-casting)。

<a name="accessors-and-mutators"></a>
## 存取器與修改器

<a name="defining-an-accessor"></a>
### 定義存取器

要定義一個存取器，請在你的模型上建立一個 `getFooAttribute` 方法，其中 `Foo` 是你想要存取的欄位，並要使用「駝峰式」命名。在本範例中，我們會定義一個專門給 `first_name` 屬性的存取器。當 Eloquent 在嘗試取得 `first_name` 屬性的值時，該存取器就會自動被呼叫：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * 取得使用者的名字。
         *
         * @param  string  $value
         * @return string
         */
        public function getFirstNameAttribute($value)
        {
            return ucfirst($value);
        }
    }

如你所見，欄位的原始值被傳到存取器，這可以讓你操作並回傳該值。如果要存取在存取器中的值，你可以在模型實例上簡單的存取 `first_name` 屬性：

    $user = App\User::find(1);

    $firstName = $user->first_name;

當然，你也可以使用存取器從現有的屬性中回傳剛被計算過的值：

    /**
     * 取得使用者的全名。
     *
     * @return string
     */
    public function getFullNameAttribute()
    {
        return "{$this->first_name} {$this->last_name}";
    }

<a name="defining-a-mutator"></a>
### 定義修改器

要定義修改器，請在你的模型上定義 `setFooAttribute` 方法，其中 `Foo` 是你想要存取的欄位，並要使用「駝峰式」命名。所以，再一次讓我們為 `first_name` 屬性定義修改器。當我們嘗試在模型上設定 `first_name` 屬性的值時，會自動呼叫這個修改器：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * 設定使用者的名字。
         *
         * @param  string  $value
         * @return void
         */
        public function setFirstNameAttribute($value)
        {
            $this->attributes['first_name'] = strtolower($value);
        }
    }

修改器會取得在該屬性修改後的值，這可以讓你操作該值並設定到 Eloquent 模型內的 `$attributes` 屬性。例如：如果我們嘗試將 `first_name` 屬性設定為 `Sally`：

    $user = App\User::find(1);

    $user->first_name = 'Sally';

在這個範例中，`setFirstNameAttribute` 函式會在被呼叫時使用 `Sally` 這個值。修改器會接著 `strtolower` 函式應用在名稱上，並將它的結果值設定到內部 `$attributes` 陣列中。

<a name="date-mutators"></a>
## 日期修改器

預設的 Eloquent 會將 `created_at` 和 `updated_at` 欄位轉會成 [Carbon](https://github.com/briannesbitt/Carbon)的實例，套件擴充了 PHP `DateTime` 類別，並提供各種有用的方法。你可以自訂哪些日期需要自動被修改，甚至完全禁止這個修改，只要覆寫模型的 `$dates` 屬性：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * 應該被修改的日期屬性。
         *
         * @var array
         */
        protected $dates = [
            'created_at',
            'updated_at',
            'deleted_at'
        ];
    }

當然欄位被當作是一個日期時，你也可以將它的值設定成 UNIX 時間戳記、日期字串（`Y-m-d`）、日期字串，而且當然可以是一個 `DateTime`/`Carbon` 的實例，然後日期的值會自動正確的儲存在你的資料庫中：

    $user = App\User::find(1);

    $user->deleted_at = Carbon::now();

    $user->save();

如上所述，當你在 `$dates` 屬性中取得被列出的屬性時，他們會自動被轉換成 [Carbon](https://github.com/briannesbitt/Carbon) 實例，這可以讓你在屬性上使用任何 Carbon 的方法：

    $user = App\User::find(1);

    return $user->deleted_at->getTimestamp();

#### 日期格式

預設的時間戳記被格式化成 `'Y-m-d H:i:s'`。如果你需要自訂時間戳格式，請在你的模型上設定 `$dateFormat` 屬性。這個屬性決定了在資料庫中如何儲存資料屬性，以及當模型被序列化成陣列或 JSON 時決定它們的格式：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Flight extends Model
    {
        /**
         * 儲存模型的時間欄位格式。
         *
         * @var string
         */
        protected $dateFormat = 'U';
    }

<a name="attribute-casting"></a>
## 轉換屬性

在模型上的 `$casts` 屬性提供一個可方便地將屬性轉換成常用資料型別的方法。`$casts` 屬性應該是一組陣列，其中的鍵是要被轉換的屬性別稱，而值是你希望欄位要被轉換的類型。目前支援轉換的類型的有：`integer`、`real`、`float`、`double`、`string`、`boolean`、`object`、`array`、`collection`、`date`、`datetime` 和 `timestamp`。

例如，讓我們轉換 `is_admin` 屬性，該屬性被儲存在我們資料庫中，並以整數（`0` 或 `1`）來代表布林值：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * 應該轉換成原本類型的屬性。
         *
         * @var array
         */
        protected $casts = [
            'is_admin' => 'boolean',
        ];
    }

現在 `is_admin` 屬性會總是在你存取它時轉換成布林值，即使儲存在資料庫裡面的值是一個整數：

    $user = App\User::find(1);

    if ($user->is_admin) {
        //
    }

<a name="array-and-json-casting"></a>
### 陣列和 JSON 轉換

當處理以序列化 JSON 形式儲存的欄位時，`array` 型別轉換將會特別的有用。例如，如果你的資料庫有被序列化成 JSON 的 `JSON` 或 `TEXT` 欄位類型，當你在 Eloquent 模型上存取屬性時，將 `array` 新增到該屬性將自動反序列化為一個 PHP 陣列：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * 應該轉換成原本型別的屬性。
         *
         * @var array
         */
        protected $casts = [
            'options' => 'array',
        ];
    }

一旦 `$cast` 被定義，你可以存取 `options` 屬性，並且自動將它從 JSON 反序列化成一組 PHP 陣列。當你在設定 `options` 屬性值的時候，給定的陣列會自動被序列化成 JSON，然後被儲存：

    $user = App\User::find(1);

    $options = $user->options;

    $options['key'] = 'value';

    $user->options = $options;

    $user->save();
