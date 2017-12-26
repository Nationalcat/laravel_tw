# Eloquent: 關聯

- [介紹](#introduction)
- [定義關聯](#defining-relationships)
    - [一對一](#one-to-one)
    - [一對多](#one-to-many)
    - [一對多（反向）](#one-to-many-inverse)
    - [多對多](#many-to-many)
    - [遠層一對多](#has-many-through)
    - [多型關聯](#polymorphic-relations)
    - [多對多多型關聯](#many-to-many-polymorphic-relations)
- [查詢關聯](#querying-relations)
    - [關聯方法與動態屬性比較](#relationship-methods-vs-dynamic-properties)
    - [查詢存在的關聯](#querying-relationship-existence)
    - [查詢不存在的關聯](#querying-relationship-absence)
    - [計算關聯模型](#counting-related-models)
- [預載入](#eager-loading)
    - [限制預載入](#constraining-eager-loads)
    - [延遲預載入](#lazy-eager-loading)
- [寫入與更新關聯模型](#inserting-and-updating-related-models)
    - [`save` 方法](#the-save-method)
    - [`create` 方法](#the-create-method)
    - [更新關聯歸屬](#updating-belongs-to-relationships)
    - [更新多對多關聯](#updating-many-to-many-relationships)
- [更新主要關聯的時間戳記](#touching-parent-timestamps)

<a name="introduction"></a>
## 介紹

資料庫的資料表通常會互相關聯。例如，部落格文章可能有許多評論，或是訂單會與下單的使用者有關聯。Eloquent 使這些關聯變得更容易於管理與運用，並支援幾種不同類型的關聯：

- [一對一](#one-to-one)
- [一對多](#one-to-many)
- [多對多](#many-to-many)
- [遠層一對多](#has-many-through)
- [多型關聯](#polymorphic-relations)
- [多對多的多型關聯](#many-to-many-polymorphic-relations)

<a name="defining-relationships"></a>
## 定義關聯

Eloquent 關聯在你的 Eloquent 模型類別上定義方法。因此，像是 Eloquent 模型本身，關聯也有幾個強大的[查詢建構器](/laravel_tw/docs/5.5/queries)，定義關聯作為方法提供強大的方法鏈結和查詢功能。例如，我們可以在 `post` 關聯上鏈結額外的條件：

    $user->posts()->where('active', 1)->get();

但是，在深入探討如何使用關聯之前，讓我們學習如何定義每種類型吧。

<a name="one-to-one"></a>
### 一對一

一對一關聯是相當基本的關聯。例如，`User` 模型可與 `Phone` 關聯。要定義這個關聯，我們先在 `User` 模型上放置 `phone` 方法。`phone` 方法會呼叫 `hasOne` 方法並回傳它的結果：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * 取得與使用者有關的電話記錄。
         */
        public function phone()
        {
            return $this->hasOne('App\Phone');
        }
    }

模型關聯名稱作為第一個參數傳入 `hasOne` 方法。關聯一旦被定義，我們可以使用 Eloquent 的動態屬性來取得關聯的記錄。動態屬性可以讓你存取關聯方法，這就像是在模型上定義它們的屬性一樣：

    $phone = User::find(1)->phone;

Eloquent 決定了基於模型名稱的關聯外鍵。在這個案例中，`Phone` 模型會自動假設有一個 `user_id` 外鍵。如果你希望覆寫這個命名，你可以在 `hasOne` 方法傳入想要的外鍵名稱作為第二個參數：

    return $this->hasOne('App\Phone', 'foreign_key');

另外，Eloquent 假設外鍵會有一個與上層欄位的 `id` (或自訂的 `$primaryKey`) 相符合的值。換句話說，Eloquent 會在 `Phone` 記錄上的 `user_id` 欄位中尋找使用者的 `id` 欄位。如果你想要關聯使用 `id` 以外的值，你可以在 `hasOne` 方法傳入選擇你自訂的鍵作為第三個參數：

    return $this->hasOne('App\Phone', 'foreign_key', 'local_key');

#### 定義反向的關聯

所以我們能從我們的 `User` 中存取 `Phone`。現在，讓我們在 `Phone` 模型上定義關聯，這可以讓我們存取擁有手機的 `User`。我們能使用 `belongsTo` 來定義 `hasOne` 的反向關聯：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Phone extends Model
    {
        /**
         * 取得擁有手機的使用者。
         */
        public function user()
        {
            return $this->belongsTo('App\User');
        }
    }

在以上範例中，Eloquent 會嘗試匹配 `Phone` 模型的 `user_id` 至 `User` 模型的 `id`。Eloquent 透過檢查關聯方法的名稱和使用 `_id` 後綴方法的名稱來決定預設外鍵名稱。然而，如果在 `Phone` 上的外鍵不是 `user_id`，你可以在 `belongsTo` 方法上傳入自訂鍵作為第二個參數：

    /**
     * 取得擁有手機的使用者。
     */
    public function user()
    {
        return $this->belongsTo('App\User', 'foreign_key');
    }

如果你的上層模型不是使用 `id` 作為主鍵，或是你希望以不同的欄位 join 下層模型，你可以傳遞第三參數至 `belongsTo` 方法指定層資料表的自訂鍵：

    /**
     * 取得擁有手機的使用者。
     */
    public function user()
    {
        return $this->belongsTo('App\User', 'foreign_key', 'other_key');
    }

<a name="default-models"></a>
#### 預設模型

如果給定的關聯為 `null`，`belongsTo` 關聯可以讓你定義預設模型。這個模式通常被稱作[空物件模式](https://en.wikipedia.org/wiki/Null_Object_pattern)，可以協助移除程式碼中條件檢查。在以下範例中，`user` 關聯會因為沒有 `user` 被附加到 post 上而回傳空的 `App\User` 模型：

    /**
     * 取得該文章的作者。
     */
    public function user()
    {
        return $this->belongsTo('App\User')->withDefault();
    }

要預先填充模型的屬性，你可以傳入陣列或閉包到 `withDefault` 方法：

    /**
     * 取得該文章的作者。
     */
    public function user()
    {
        return $this->belongsTo('App\User')->withDefault([
            'name' => 'Guest Author',
        ]);
    }

    /**
     * 取得該文章的作者。
     */
    public function user()
    {
        return $this->belongsTo('App\User')->withDefault(function ($user) {
            $user->name = 'Guest Author';
        });
    }

<a name="one-to-many"></a>
### 一對多

「一對多」關聯是被用於定義單一模型可以擁有好幾個模型關聯。例如，一篇部落格文章可能有很多評論。像是所有其他的 Eloquent 關聯，一對多關聯是在你的 Eloquent 模型上放置一個函式來定義關聯：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Post extends Model
    {
        /**
         * 取得部落格文章的評論
         */
        public function comments()
        {
            return $this->hasMany('App\Comment');
        }
    }

請記得，Eloquent 會在 `Comment` 模型上自動決定該屬性外鍵欄位。按照慣例，Eloquent 會使用被關聯的模型「snake case」 名稱與後綴 `_id` 來命名。因此，在這個範例，Eloquent 會假設在 `Comment` 模型上的外鍵是 `post_id`。

一旦關聯被定義，我們能透過訪問 `comments` 屬性來存取 comments 的集合。請記得，由於 Eloquent 提供「動態屬性」的關係，我們才能存取模型方法，就像是他們被定義為模型上的屬性：

    $comments = App\Post::find(1)->comments;

    foreach ($comments as $comment) {
        //
    }

當然，因為所有的關聯也提供查詢產生器的功能，你可以對取得的評論進一步增加條件，透過呼叫 `comments` 方法，接著在該查詢的後方鏈結上條件：

    $comments = App\Post::find(1)->comments()->where('title', 'foo')->first();

就像 `hasOne` 方法，你也可以透過傳入額外的參數至 `hasMany` 方法複寫外鍵與本地鍵：

    return $this->hasMany('App\Comment', 'foreign_key');

    return $this->hasMany('App\Comment', 'foreign_key', 'local_key');

<a name="one-to-many-inverse"></a>
### 一對多 (反向)

現在我們能存取所有文章的評論，讓我們定義一個透過評論存取上層文章的關聯。若要定義相對於 `hasMany` 的關聯，在下層模型定義一個叫做 `belongsTo` 方法的關聯函式：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Comment extends Model
    {
        /**
         * 取得擁有該評論的文章。
         */
        public function post()
        {
            return $this->belongsTo('App\Post');
        }
    }

一旦關聯被定義之後，我們可以透過 `post`「動態屬性」取得 `Comment` 的 `Post` 模型：

    $comment = App\Comment::find(1);

    echo $comment->post->title;

在上述例子中，Eloquent 會嘗試匹配 `Comment` 模型的 `post_id` 至 `Post` 模型的 `id`。Eloquent 判斷的預設外鍵名稱參考於關聯模型的方法，並在方法名稱後面加上 `_id`。當然，如果 `Comment` 模型的外鍵不是 `post_id`，你可以傳遞自訂的鍵名至 `belongsTo` 方法的第二個參數：

    /**
     * 取得擁有該評論的文章。
     */
    public function post()
    {
        return $this->belongsTo('App\Post', 'foreign_key');
    }

如果你的上層模型不是使用 `id` 作為主鍵，或是你希望以不同的欄位 join 下層模型，你可以傳遞第三參數至 `belongsTo` 方法指定上層資料表的自訂鍵：

    /**
     * 取得擁有該評論的文章。
     */
    public function post()
    {
        return $this->belongsTo('App\Post', 'foreign_key', 'other_key');
    }

<a name="many-to-many"></a>
### 多對多

多對多關聯稍微比 `hasOne` 及 `hasMany` 關聯還複雜。這種關聯的例子如，一位使用者可能用有很多身份，而一種身份可能很多使用者都有。舉例來說，很多使用者都擁有「管理者」的身份。要定義這種關聯，需要使用三個資料表：`users`、`roles` 和 `role_user`。`role_user` 表命名是以相關聯的兩個模型資料表，依照字母順序命名，並包含了 `user_id` 和 `role_id` 欄位。

多對多關聯透過撰寫一個被用來回傳 `belongsToMany` 方法結果來定義。例如，讓我們在 `User` 模型上定義 `roles` 方法：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * 屬於該使用者的身份。
         */
        public function roles()
        {
            return $this->belongsToMany('App\Role');
        }
    }

一旦關聯被定義，你可以使用 `roles` 動態屬性存取使用者的身份：

    $user = App\User::find(1);

    foreach ($user->roles as $role) {
        //
    }

當然，就像所有的其他關聯類型，你可以呼叫 `roles` 方法，接著在該關聯之後鏈結上查詢的條件：

    $roles = App\User::find(1)->roles()->orderBy('name')->get();

如前文所提，若要判斷關聯合併的資料表名稱，Eloquent 會合併兩個關聯模型的名稱並依照字母順序命名。當然你可以自由的複寫這個慣例。你可以透過傳遞第二個參數至 `belongsToMany` 方法來達成：

    return $this->belongsToMany('App\Role', 'role_user');

除了自訂合併資料表的名稱，你也可以透過傳遞額外參數至 `belongsToMany` 方法來自訂資料表裡鍵的欄位名稱。第三個參數是你定義在關聯中的模型的外鍵名稱，而第四個參數則是你要合併的模型中的外鍵名稱：

    return $this->belongsToMany('App\Role', 'role_user', 'user_id', 'role_id');

#### 定義反向的關聯

要定義反向多對多的關聯，你只需要簡單的放置另一個名為 `belongsToMany` 至你關聯的模型。繼續我們的使用者身份範例，讓我們在 `Role` 模型定義 `users` 方法：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Role extends Model
    {
        /**
         * 屬於該身份的使用者們。
         */
        public function users()
        {
            return $this->belongsToMany('App\User');
        }
    }

如你所見，此定義除了簡單的參考 `App\User` 模型外，與 `User` 的對應完全相同。因為我們重複使用了 `belongsToMany` 方法，當定義相對於多對多的關聯時，所有常用的自訂資料表與鍵的選項都是可用的。

#### 取得中介表欄位

如你所知，要操作多對多關聯需要一個中介的資料表。Eloquent 提供了一些有用的方法和這張表互動。例如，假設 `User` 物件關聯到很多 `Role` 物件。存取這些關聯物件時，我們可以在模型使用 `pivot` 屬性存取中介資料表的資料：

    $user = App\User::find(1);

    foreach ($user->roles as $role) {
        echo $role->pivot->created_at;
    }

注意我們取出的每個 `Role` 模型物件，會自動被賦予 `pivot` 屬性。此屬性是代表中介表的模型，且可以像其它的 Eloquent 模型一樣被使用。

預設來說，`pivot` 物件只提供模型的鍵。如果你的 pivot 資料表包含了其他的屬性，可以在定義關聯方法時指定那些欄位：

    return $this->belongsToMany('App\Role')->withPivot('column1', 'column2');

如果你想要樞紐表自動維護 `created_at` 和 `updated_at` 時間戳記，在定義關聯方法時加上 `withTimestamps` 方法：

    return $this->belongsToMany('App\Role')->withTimestamps();

#### 自訂 `pivot` 屬性名稱

如之前所說的，從中介表中的數行可以在模型上使用 `pivot` 方法來存取。然而，你可以自由的自訂該屬性的名稱來更好的反映在應用程式中的用途。

例如，如果你的應用程式包含可以訂閱 Podcasts 的使用，使用者與 Podcasts 之間可能存在著多對多的關係。如果遇到這種情況，你可能希望你的中介表來存取器重新命名為 `subscription`，而不是 `pivot`。在定義關聯時，可以使用 `as` 方法來做到：

    return $this->belongsToMany('App\Podcast')
                    ->as('subscription')
                    ->withTimestamps();

一旦完成了，你就可以存取使用自訂的名稱來存取中介表的資料：

    $users = User::with('podcasts')->get();

    foreach ($users->flatMap->podcasts as $podcast) {
        echo $podcast->subscription->created_at;
    }

#### 透過中介表來篩選關聯

在你定義關聯時，還能使用 `wherePivot` 和 `wherePivotIn` 方法過濾回傳的結果：

    return $this->belongsToMany('App\Role')->wherePivot('approved', 1);

    return $this->belongsToMany('App\Role')->wherePivotIn('priority', [1, 2]);

#### 自訂義中介表模型

如果你想要自訂義模型來表示關聯的中介表，你可以在定義關聯時呼叫 `using` 方法。所有用於表示關聯的中介表的自訂模型必須繼承 `Illuminate\Database\Eloquent\Relations\Pivot` 類別。 例如，我們可以選擇使用自訂的 `UserRole` 中介模型來定義 `Role`：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Role extends Model
    {
        /**
         * 屬於該身份的使用者們。
         */
        public function users()
        {
            return $this->belongsToMany('App\User')->using('App\UserRole');
        }
    }

在定義 `UserRole` 模型時，我們可以繼承 `Pivot` 類別：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Relations\Pivot;

    class UserRole extends Pivot
    {
        //
    }

<a name="has-many-through"></a>
### 遠層一對多

「遠層一對多」透過中介的關聯提供一個方便的方式來取得遠層的關聯。例如，`Country` 模型可以有許多 `Post` 模型通過中介表 `User` 模型。在這個範例中，你能輕易的收集給定國家的所有部落格文章。讓我們看這個關聯所需的資料表吧：

    countries
        id - integer
        name - string

    users
        id - integer
        country_id - integer
        name - string

    posts
        id - integer
        user_id - integer
        title - string

雖然 `posts` 本身不包含一個 `country_id` 欄位，但 `hasManyThrough` 關聯透過 `$country->posts` 來提供我們存取一篇國家的文章。要執行此查詢，Eloquent 會檢查中介表 `users` 的 `country_id`。在找到匹配的使用者 ID 後，就會在 `posts` 資料表使用它們來查詢。

現在我們已經檢查了關聯的資料表結構，讓我們將它定義在 `Country` 模型：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Country extends Model
    {
        /**
         * 取得該國家的所有文章。
         */
        public function posts()
        {
            return $this->hasManyThrough('App\Post', 'App\User');
        }
    }

傳送到 `hasManyThrough` 方法的第一個參數是我們希望最終存取的模型名稱，而第二個參數為中介模型的名稱。

當執行關聯的查詢時，通常將會使用 Eloquent 的外鍵慣例。如果你想要自訂關聯的鍵，你可以傳遞它們至 `hasManyThrough` 方法的第三與第四個參數。第三個參數為中介模型的外鍵名稱。第四個參數為最終模型的外鍵名稱。第五個參數是本地鍵，而第六個參數是中介模型的本地鍵：

    class Country extends Model
    {
        public function posts()
        {
            return $this->hasManyThrough(
                'App\Post',
                'App\User',
                'country_id', // 在 users 資料表上的外鍵...
                'user_id', // 在 posts 資料表上的外鍵...
                'id', // 在 countries 資料表上的本地鍵...
                'id' // 在 users 資料表上的本地鍵...
            );
        }
    }

<a name="polymorphic-relations"></a>
### 多型關聯

#### 資料表結構

多型關聯允許一個模型在單一的關聯從屬一個以上的其他模型。例如，想像你有個應用程式的使用者能「評論」該文章與影片。使用多行關聯，你能使用單一個 `comments` 資料表來來應付這兩種情況。首先，讓我們看看建構這個關聯所需的資料表結構：

    posts
        id - integer
        title - string
        body - text

    videos
        id - integer
        title - string
        url - string

    comments
        id - integer
        body - text
        commentable_id - integer
        commentable_type - string

在 `comments` 資料表上，需要注意兩個重要的欄位：`commentable_id` 和 `commentable_type` 欄位。`commentable_id` 欄位會存放 post 或 video 的 ID 值，而 `commentable_type` 欄位會存放擁有的模型的類別名稱。當存取 `commentable` 關聯時，`commentable_type` 欄位讓 ORM 確定回傳擁有的模型是哪種「類型」。

#### 模型結構

接著，讓我們查看建立這種關聯所需的模型定義：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Comment extends Model
    {
        /**
         * 取得所有擁有可回覆的模型。
         */
        public function commentable()
        {
            return $this->morphTo();
        }
    }

    class Post extends Model
    {
        /**
         * 取得該文章的所有評論。
         */
        public function comments()
        {
            return $this->morphMany('App\Comment', 'commentable');
        }
    }

    class Video extends Model
    {
        /**
         * 取得該影片的所有評論。
         */
        public function comments()
        {
            return $this->morphMany('App\Comment', 'commentable');
        }
    }

#### 取得多型關聯

你的資料表與模型一旦定義好了，你可以透過模型來存取關聯。例如，要為文章存取所有的評論，我們能簡單的使用 `comments` 動態屬性：

    $post = App\Post::find(1);

    foreach ($post->comments as $comment) {
        //
    }

你也可以透過存取呼叫 `morphTo` 的方法名稱來從多型模型中取得多型關聯的擁有者。在本案例中，就是指 `Comment` 模型上的 `commentable` 方法。所以，我們會存取該方法作為動態屬性：

    $comment = App\Comment::find(1);

    $commentable = $comment->commentable;

在 `Comment` 模型上的 `commentable` 關聯會回傳整個 `Post` 或 `Video` 實例，並根據模型類型來決定誰能擁有該 comment。

#### 自訂多型類型

預設的 Laravel 會使用完全合格的類別名稱來儲存關聯模型的類型。例如，以上面的例子中的 `Comment` 可能屬於 `Post` 或 `Video`，預設的 `commentable_type` 將會是 `App\Post` 和 `App\Video` 其中一個。然而，你可能希望從應用程式的內部結構解耦你的資料庫。在那種情況下，你可以定義一個關聯的 `morph map`來指示 Eloquent 為每個模型使用一個自訂義名稱，而不是類別名稱：

    use Illuminate\Database\Eloquent\Relations\Relation;

    Relation::morphMap([
        'posts' => 'App\Post',
        'videos' => 'App\Video',
    ]);

你可以在 `AppServiceProvider` 的 `boot` 函式中註冊 `morphMap`，或者如果你願意的話可以建立獨立的服務提供者。

<a name="many-to-many-polymorphic-relations"></a>
### 多對多的多型關聯

#### 資料表結構

除了一般的多型關聯，你也可以定義「多對多」的多型關聯。例如，部落格的 `Post` 和 `Video` 模型可以共用多型關聯至 `Tag` 模型。使用多對多的多型關聯能夠讓你的部落格文章及影片共用獨立標籤的單一列表。首先，讓我們查看資料表結構：

    posts
        id - integer
        name - string

    videos
        id - integer
        name - string

    tags
        id - integer
        name - string

    taggables
        tag_id - integer
        taggable_id - integer
        taggable_type - string

#### 模型結構

接著，我們已經準備好定義模型的關聯。`Post` 及 `Video` 模型會都擁有 `tags` 方法，並在該方法內呼叫自身 Eloquent 類別的 `morphToMany` 方法：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Post extends Model
    {
        /**
         * 取得該文章所有的標籤。
         */
        public function tags()
        {
            return $this->morphToMany('App\Tag', 'taggable');
        }
    }

#### 定義反向的關聯

然後，在 `Tag` 模型上，你必須對每個要關聯的模型定義一個方法。所以，在這個例子裡，我們需要定義一個 `posts` 方法及一個 `videos` 方法：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Tag extends Model
    {
        /**
         * 取得被分配到這個標籤的所有文章。
         */
        public function posts()
        {
            return $this->morphedByMany('App\Post', 'taggable');
        }

        /**
         * 取得被分配到這個標籤的所有影片。
         */
        public function videos()
        {
            return $this->morphedByMany('App\Video', 'taggable');
        }
    }

#### 取得關聯

一旦你的資料表及模型被定義後，你可以透過你的模型存取關聯。例如，若要存取文章的所有標籤，你可以簡單的使用 `tags` 動態屬性：

    $post = App\Post::find(1);

    foreach ($post->tags as $tag) {
        //
    }

你也可以從多型模型的多型關聯裡，透過存取執行呼叫 `morphedByMany` 的方法名稱取得擁有者。在我們例子中，就是 `Tag` 模型中的 `posts` 或 `videos` 方法。所以，你可以存取使用動態屬性存取這個方法：

    $tag = App\Tag::find(1);

    foreach ($tag->videos as $video) {
        //
    }

<a name="querying-relations"></a>
## 查詢關聯

因為所有類型的 Eloquent 關聯都是透過方法來定義，所以可以呼叫這些方法來取得一個關聯實例，而不需要實際執行關聯查詢。此外，所有類型的 Eloquent 關聯也能當作[查詢構建器](/laravel_tw/docs/5.5/queries)，這可以讓你在資料庫執行最後的 SQL 之前繼續將查詢條件鏈結到關聯查詢上。

例如，假設有一個部落格系統，其中 `User` 模型擁有許多關聯的 `Post` 模型：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * 取得使用者的所有文章。
         */
        public function posts()
        {
            return $this->hasMany('App\Post');
        }
    }

你可以查詢 `posts` 關聯並增加額外的條件至關聯，像是：

    $user = App\User::find(1);

    $user->posts()->where('active', 1)->get();

你可以在關聯上使用任何關於[查詢構建器](/laravel_tw/docs/5.5/queries)的方法，所以請務必查閱查詢構建器的文件，來了解所有可用的方法。

<a name="relationship-methods-vs-dynamic-properties"></a>
### 關聯方法與動態屬性比較

如果你不需要增加額外的條件至 Eloquent 的關聯查詢，你可以簡單的存取關聯就如同屬性一樣。例如，延續我們剛剛的 `User` 及 `Post` 範例模型，我們可以存取所有使用者的文章，像是：

    $user = App\User::find(1);

    foreach ($user->posts as $post) {
        //
    }

動態屬性是「延遲載入」，意指它們只會在你需要存取它們的時候載入關聯資料。正因為如此，開發者通常使用[預載入](#eager-loading)來預先載入在載入模型後將會被存取的關聯資料。預載入提供了一個明顯減少會被執行於載入模型關聯的 SQL 查詢。

<a name="querying-relationship-existence"></a>
### 查詢存在的關聯

為模型分配該記錄時，你可能希望根據關聯的存在來限制結果。例如，想像你想要取得至少有一筆評論的所有部落格文章。想要這麼做，你可以傳入關聯名稱到 `has` 和 `orHas` 方法：

    // 取得至少有一筆評論的所有文章...
    $posts = App\Post::has('comments')->get();

你也可以指定運算子和計算來進一步的自訂查詢：

    // 取得至少有三筆以上評論的所有文章...
    $posts = Post::has('comments', '>=', 3)->get();

也可以使用「點」符號建構巢狀的 `has` 語句。例如，你可能想取得所有至少有一篇評論被評分的文章：

    // 取得所有至少有一筆評論的被評分的文章...
    $posts = Post::has('comments.votes')->get();

如果你想要更進階的用法，可以使用 `whereHas` 和 `orWhereHas` 方法，在 `has` 查詢裡設定「where」條件。此方法可以讓你增加自訂的條件至關聯條件中，像是檢查評論的內容：

    // 取得所有至少有一篇文章相似於 foo% 的文章
    $posts = Post::whereHas('comments', function ($query) {
        $query->where('content', 'like', 'foo%');
    })->get();

<a name="querying-relationship-absence"></a>
### 查詢尚未存在的關聯

為模型存取一筆記錄時，你可能希望根據尚未存在的關聯來限制結果。例如，想像你想要存取**沒有**任何評論的所有部落格文章。若要這麼做，你可以傳入關聯名稱到 `doesntHave` 和 `orDoesntHave` 方法：

    $posts = App\Post::doesntHave('comments')->get();

如果你需要更多用法，你可以在 `doesntHave` 查詢中使用 `whereDoesntHave` 和 `orWhereDoesntHave` 方法時加入「where」條件。這些方法可以讓你新增自訂的條件限制加到關聯中，像是檢查評論內容：

    $posts = Post::whereDoesntHave('comments', function ($query) {
        $query->where('content', 'like', 'foo%');
    })->get();

<a name="counting-related-models"></a>
### 計算關聯模型

如果你想要從關聯中計算結果的數字，而不載入它們。你可以使用 `withCount` 方法，該方法會在你的結果模型上放置 `{relation}_count` 欄位。例如：

    $posts = App\Post::withCount('comments')->get();

    foreach ($posts as $post) {
        echo $post->comments_count;
    }

你可以為多個關聯新增「counts」，還有為查詢新增條件限制：

    $posts = Post::withCount(['votes', 'comments' => function ($query) {
        $query->where('content', 'like', 'foo%');
    }])->get();

    echo $posts[0]->votes_count;
    echo $posts[0]->comments_count;

你也可以別名該關聯的計算結果，可以讓你對同一個關聯進行多次計算：

    $posts = Post::withCount([
        'comments',
        'comments as pending_comments_count' => function ($query) {
            $query->where('approved', false);
        }
    ])->get();

    echo $posts[0]->comments_count;

    echo $posts[0]->pending_comments_count;

<a name="eager-loading"></a>
## 預載入

當透過屬性存取 Eloquent 關聯時，該關聯資料會被「延遲載入」。意指該關聯資料直到你第一次以屬性存取前，實際上並沒有被載入。不過，Eloquent 可以在你查詢上層模型時「預載入」關聯資料。預載入避免了 N + 1 查詢的問題。要說明 N + 1 查詢的問題，試想一個 `Book` 模型會關聯至 `Author`：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Book extends Model
    {
        /**
         * 取得撰寫該書的作者。
         */
        public function author()
        {
            return $this->belongsTo('App\Author');
        }
    }

現在，讓我們取得所有的書籍及其作者：

    $books = App\Book::all();

    foreach ($books as $book) {
        echo $book->author->name;
    }

上方的迴圈會執行一次查詢並取回所有資料表上的書籍，然後每本書都會執行一次查詢取得作者。所以，若有 25 本書，迴圈就會進行 26 次查詢：1 次是原本的書籍，及對每本書查詢並取得作者的額外 25 次。

很幸運地，我們可以使用預載入將查詢的操作減少至 2 次。當查詢時，使用 `with` 方法指定想要預載入的關聯資料：

    $books = App\Book::with('author')->get();

    foreach ($books as $book) {
        echo $book->author->name;
    }

對於該操作，只會執行兩次查詢：

    select * from books

    select * from authors where id in (1, 2, 3, 4, 5, ...)

#### 預載入多個關聯

有時你可能想要在單次操作中預載入多種不同的關聯。要這麼做，只需要傳遞額外的參數至 `with` 方法：

    $books = App\Book::with(['author', 'publisher'])->get();

#### 巢狀預載入

若要預載入巢狀關聯，你可以使用「點」語法。例如，讓我們在一個 Eloquent 語法中，預載入所有書籍的作者，及所有作者的個人聯絡方式：

    $books = App\Book::with('author.contacts')->get();

#### 預載入特定欄位

你可能並不總是需要取得關聯中的每一個欄位。出於這樣的原因，Eloquent 可以讓你指定想要取得的關聯欄位：

    $users = App\Book::with('author:id,name')->get();

> {note} 使用這個功能時，你應該總是在你想要取得的欄位清單中引入 `id` 欄位。

<a name="constraining-eager-loads"></a>
### 預載入條件限制

有時你可能想要預載入關聯，並且指定預載入額外的查詢條件。下面有一個例子：

    $users = App\User::with(['posts' => function ($query) {
        $query->where('title', 'like', '%first%');
    }])->get();

在這個例子裡，Eloquent 只會預載入文章標題欄位包含 `first` 的文章。當然，你也可以呼叫其他的[查詢產生器](/laravel_tw/docs/5.5/queries)來進一步自訂預載入的操作：

    $users = App\User::with(['posts' => function ($query) {
        $query->orderBy('created_at', 'desc');
    }])->get();

<a name="lazy-eager-loading"></a>
### 延遲預載入

有時你可能需要在上層模型已經被取得後才預載入關聯。例如，當你需要動態決定是否載入關聯模型時相當有幫助：

    $books = App\Book::all();

    if ($someCondition) {
        $books->load('author', 'publisher');
    }

如果你需要在預載入查詢中設定額外的查詢限制，你可以傳入一組你想要載入的關聯的陣列鍵。該陣列值會是接收查詢實例的`閉包`實例：

    $books->load(['author' => function ($query) {
        $query->orderBy('published_date', 'asc');
    }]);

想只有在尚未載入的情況下載入關聯，請使用 `loadMissing` 方法：

    public function format(Book $book)
    {
        $book->loadMissing('author');

        return [
            'name' => $book->name,
            'author' => $book->author->name
        ];
    }

<a name="inserting-and-updating-related-models"></a>
## 寫入與更新關聯模型

<a name="the-save-method"></a>
### Save 方法

Eloquent 提供了方便的方法來增加新的模型至關聯中。例如，也許你需要寫入新的 `Commnet` 至 `Post` 模型中。除了手動設定 `Commnet` 的 `post_id` 屬性外，你也可以直接使用關聯的 `save` 方法寫入 `Comment`：

    $comment = new App\Comment(['message' => 'A new comment.']);

    $post = App\Post::find(1);

    $post->comments()->save($comment);

注意我們並沒有使用動態屬性來存取 `comments` 關聯，反而是呼叫 `comments` 方法來取得關聯的實例。`save` 方法會自動在新的 `Comment` 模型中確實的新增 `post_id` 值。

如果你需要儲存多筆關聯模型，你可以使用 `saveMany` 方法：

    $post = App\Post::find(1);

    $post->comments()->saveMany([
        new App\Comment(['message' => 'A new comment.']),
        new App\Comment(['message' => 'Another comment.']),
    ]);

<a name="the-create-method"></a>
### Create 方法

除了 `save` 與 `saveMany` 方法，你也可以使用 `create` 方法，該方法可以讓你傳入屬性的陣列來建立模型，並寫入資料庫。還有啊，`save` 與 `create` 不同的地方在於 `save` 可以傳入一個完整的 Eloquent 模型實例，但 `create` 只能傳入原生的 PHP `陣列`：

    $post = App\Post::find(1);

    $comment = $post->comments()->create([
        'message' => 'A new comment.',
    ]);

> {tip} 在使用 `create` 方法之前，請確定瀏覽了[批量賦值](/laravel_tw/docs/5.5/eloquent#mass-assignment)的文件。

你可以使用 `createMany` 方法來建立多筆關聯模型：

    $post = App\Post::find(1);

    $post->comments()->createMany([
        [
            'message' => 'A new comment.',
        ],
        [
            'message' => 'Another new comment.',
        ],
    ]);

<a name="updating-belongs-to-relationships"></a>
### 更新「從屬」關聯

當更新一筆 `belongsTo` 關聯時，你可以使用 `associate` 方法。此方法會設定外鍵至下層模型：

    $account = App\Account::find(10);

    $user->account()->associate($account);

    $user->save();

要移除 `belongsTo` 關聯時，你可以使用 `dissociate` 方法。這個方法會設定關聯的外鍵為 `null`：

    $user->account()->dissociate();

    $user->save();

<a name="updating-many-to-many-relationships"></a>
### 更新多對多關聯

#### 附加與卸除

Eloquent 也提供一些額外的輔助方法讓操作關聯模型時更加方便。例如，讓我們假設一位使用者可以擁有多個身份，且每個身份可以被多位使用者擁有。要附加一個規則至一位使用者，並 join 模型及寫入記錄至中介表，可以使用 `attach` 方法：

    $user = App\User::find(1);

    $user->roles()->attach($roleId);

當附加一個關聯至模型時，你也可以傳遞一個需被寫入至中介表的額外資料陣列：

    $user->roles()->attach($roleId, ['expires' => $expires]);

當然，有些時候也需要移除使用者的一個身份。要移除一筆多對多的記錄，使用 `detach` 方法。`detach` 方法會從中介表中移除正確的記錄；當然，這兩筆資料依然會存在於資料庫中：

    // 從使用者上移除單一身份...
    $user->roles()->detach($roleId);

    // 從使用者上移除所有身份...
    $user->roles()->detach();

為了方便，`attach` 與 `detach` 都允許傳入 ID 的陣列：

    $user = App\User::find(1);

    $user->roles()->detach([1, 2, 3]);

    $user->roles()->attach([
        1 => ['expires' => $expires],
        2 => ['expires' => $expires]
    ]);

#### 同步關聯

你也可以使用 `sync` 方法建構多對多關聯。`sync` 允許傳入放置於中介表的 ID 陣列。任何不在給定陣列中的 ID 將會從中介表中被刪除。所以，在此操作結束後，只會有陣列中的 ID 存在於中介表中：

    $user->roles()->sync([1, 2, 3]);

你也可以傳遞中介表上該 ID 額外的值：

    $user->roles()->sync([1 => ['expires' => true], 2, 3]);

如果你不想移除已存在的 ID，你可以使用 `syncWithoutDetaching` 方法：

    $user->roles()->syncWithoutDetaching([1, 2, 3]);

#### 切換關聯

多對多關聯也提供 `toggle` 方法來「切換」給定 ID 的附加狀態。如果給定 ID 目前已被附加，它將會被卸除。同樣的，如果它目前被卸除，那麼它將會被附加：

    $user->roles()->toggle([1, 2, 3]);

#### 在中介表上儲存額外的資料

當你在使用多對多關聯時，`save` 方法接受一組額外的中介表屬性作為它的第二個參數：

    App\User::find(1)->roles()->save($role, ['expires' => $expires]);

#### 修改中介表中的特定記錄

如果你需要修改已存在中介表中的記錄，你可以使用 `updateExistingPivot` 方法。這個方法接受中介表記錄的外鍵和一組要更新的屬性陣列：:

    $user = App\User::find(1);

    $user->roles()->updateExistingPivot($roleId, $attributes);

<a name="touching-parent-timestamps"></a>
## 連動上層時間戳記

當一個模型 `belongsTo` 或 `belongsToMany` 另一個模型時，像是一個 `Comment` 屬於一個 `Post`，對於下層模型被更新時，欲更新上層的時間戳記相當有幫助。舉例來說，當一個 `Commnet` 模型被更新，你可能想要自動的「連動」所屬 `Post` 的 `updated_at` 時間戳記。Eloquent 使得此事相當容易。只要在關聯的下層模型增加一個包含名稱的 `touches` 屬性即可：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Comment extends Model
    {
        /**
         * 所有會被連動的關聯。
         *
         * @var array
         */
        protected $touches = ['post'];

        /**
         * 取得該評論所屬的文章。
         */
        public function post()
        {
            return $this->belongsTo('App\Post');
        }
    }

現在，當你在更新一筆 `Comment` 時，它所屬的 `Post` 擁有的 `updated_at` 欄位也會同時更新，這麼做能更方便的知道何時會使 `Post` 模型的快取失效：

    $comment = App\Comment::find(1);

    $comment->text = 'Edit to this comment!';

    $comment->save();
