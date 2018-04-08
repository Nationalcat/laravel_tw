---
layout: post
title: eloquent-serialization
tag: 5.5
---
# Eloquent：序列化

- [介紹](#introduction)
- [序列化模型與集合](#serializing-models-and-collections)
    - [序列化成陣列](#serializing-to-arrays)
    - [序列化成 JSON](#serializing-to-json)
- [從 JSON 隱藏屬性](#hiding-attributes-from-json)
- [將值附加到 JSON](#appending-values-to-json)
- [日期序列化](#date-serialization)

<a name="introduction"></a>
## 介紹

建構 JSON API 時，你經常會需要將模型和關聯轉換成陣列或 JSON。Eloquent 已經包含了這些用來轉換的方便方法，以及控制序列化中應該包含哪些屬性。

<a name="serializing-models-and-collections"></a>
## 序列化模型與集合

<a name="serializing-to-arrays"></a>
### 序列化成陣列

要將模型以及被一同載入的[關聯](/laravel_tw/docs/5.5/eloquent-relationships)轉換成陣列，你應該使用 `toArray` 方法。這個方法會採用遞迴的方式，因此，所有屬性和關聯（包含關聯中的關聯）都會被轉換成陣列：

    $user = App\User::with('roles')->first();

    return $user->toArray();

你也可以將整個模型[集合](/laravel_tw/docs/5.5/eloquent-collections)轉換成陣列：

    $users = App\User::all();

    return $users->toArray();

<a name="serializing-to-json"></a>
### 序列化成 JSON

你應該使用 `toJson` 方法來將模型轉換成 JSON。像是 `toArray`， `toJson` 方法會採用遞迴的方式，因此，所有屬性和關聯都會被轉換成 JSON：

    $user = App\User::find(1);

    return $user->toJson();

或者，你可以將模型或關聯轉換成字串，這會自動在模型或集合上呼叫 `toJson` 方法。

    $user = App\User::find(1);

    return (string) $user;

轉換成字串時，由於模型與合集被轉換成 JSON，你能直接從應用程式的路由或控制器中回傳 Eloquent 物件：

    Route::get('users', function () {
        return App\User::all();
    });

<a name="hiding-attributes-from-json"></a>
## 從 JSON 隱藏屬性

有時你可能希望限制包含在模型的陣列或 JSON 的屬性，像是密碼。可以新增 `$hidden` 屬性到你的模型來達到這個目的：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * 應該隱藏陣列中的屬性。
         *
         * @var array
         */
        protected $hidden = ['password'];
    }

> {note} 當隱藏關聯時，使用關聯方法名稱。

或者，你可以使用 `visible` 屬性來定義應該被包含在模型的陣列或 JSON 的屬性的白名單。當模型被轉換成陣列或 JSON 時，所有未包含的屬性將會被隱藏：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * 應該顯示陣列中的屬性。
         *
         * @var array
         */
        protected $visible = ['first_name', 'last_name'];
    }

#### 暫時修改屬性的可見性

如果你想要在給定模型實例上顯示原先被隱藏的屬性，你可以使用 `makeVisible` 方法。`makeVisible` 方法會回傳模型實例，讓你可以方便鏈結方法：

    return $user->makeVisible('attribute')->toArray();

同樣的，如果你想要在給定模型實例上隱藏原先被顯示的屬性，你可以使用 `makeHidden` 方法。

    return $user->makeHidden('attribute')->toArray();

<a name="appending-values-to-json"></a>
## 將值附加到 JSON

有時候，將模型轉換為陣列或JSON 時，你可能希望在資料庫中新增沒有相應欄位的屬性。要達到此目的，首先為這個值定義[存取器](/laravel_tw/docs/5.5/eloquent-mutators)：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * 為使用者取得管理者的標誌。
         *
         * @return bool
         */
        public function getIsAdminAttribute()
        {
            return $this->attributes['admin'] == 'yes';
        }
    }

建立存取器後，在該模型上新增屬性名稱到 `appends` 屬性。請注意，屬性名稱通常採用「snake case」，甚至使用「駝峰式命名」來定義存取器：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * 該存取器被附加到模型的陣列表單。
         *
         * @var array
         */
        protected $appends = ['is_admin'];
    }

該屬性一旦被新增到 `appends` 清單，它會被包含在模型的陣列與 JSON 中。`appends` 陣列中的屬性也會依循模型上的 `visible` 和 `hidden` 設定。

<a name="date-serialization"></a>
## 日期序列化

Laravel 擴充了 [Carbon](https://github.com/briannesbitt/Carbon) 日期函式庫，這會提供方便定製 Carbon 的 JSON 序列化格式。要如何為你所有的應用程式序列化定製的 Carbon 日期，請使用 `Carbon::serializeUsing` 方法。`serializeUsing` 方法接受一個閉包，閉包會是為 JSON 序列化所回傳的字串型別日期：

    <?php

    namespace App\Providers;

    use Illuminate\Support\Carbon;
    use Illuminate\Support\ServiceProvider;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * 執行註冊後，啟動服務。
         *
         * @return void
         */
        public function boot()
        {
            Carbon::serializeUsing(function ($carbon) {
                return $carbon->format('U');
            });
        }

        /**
         * 在容器中註冊綁定。
         *
         * @return void
         */
        public function register()
        {
            //
        }
    }
