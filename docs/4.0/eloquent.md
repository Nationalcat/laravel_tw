---
layout: post
title: eloquent
tag: 4.0
---
# Eloquent ORM

- [介紹](#introduction)
- [基本使用](#basic-usage)
- [大量指定](#mass-assignment)
- [新增、更新及刪除](#insert-update-delete)
- [軟刪除](#soft-deleting)
- [時間戳記](#timestamps)
- [查詢範圍](#query-scopes)
- [關聯](#relationships)
- [查詢關聯](#querying-relations)
- [預先加載](#eager-loading)
- [新增相關模組](#inserting-related-models)
- [更新母節點時間戳記](#touching-parent-timestamps)
- [使用數據透視表](#working-with-pivot-tables)
- [聚集](#collections)
- [存取及修改器](#accessors-and-mutators)
- [日期設置器](#date-mutators)
- [模組事件](#model-events)
- [模組監控](#model-observers)
- [轉換成陣列 / JSON](#converting-to-arrays-or-json)

<a name="introduction"></a>
## 介紹

Eloquent ORM 是 Laravel 提供的一個優雅、簡單的 ActiveRecord 實作資料庫的操作的方法，每個資料表有一個相對應的 "模型 (Model)" ，讓你可以對相對應的資料表進行互動操作。

在開始之前，要先確定你有在 `app/config/database.php` 檔案中設定好資料庫的連線資訊。

<a name="basic-usage"></a>
## 基本使用

在開始要建立 Eloquent 模型時，通常這些模型的檔案都放在 `app/models` 目錄下，但你也可以放在任何你想放的地方，只要能夠透過 `composer.json` 檔案中的設定去載入模型即可。

**定義 Eloquent 模型**

	class User extends Eloquent {}

你會注意到，我們並沒有告訴 Eloquent 我們的 `User` 模型是要用哪一個資料表去做存取控制，除非你有另外指定要使用的資料表名稱，否則 Eloquent 將會使用類別名稱的"小寫"及"複數"的單字去當作預設的資料表名稱，所以在這個例子中， Eloquent 會假設 `User` 模型的資料是存放在 `users` 資料表中，你也可以在你的模型中使用 `table` 變數去指定你想要的使用的資料表名稱:

	class User extends Eloquent {

		protected $table = 'my_users';

	}

> **注意:** Eloquent 會假設每一個資料表的主鍵 (primary key) 名稱為 `id` ，你也可以使用 `primaryKey` 變數去複寫原來的規則，同樣的，你也可以定義 `connection` 變數去複寫你想要在這個模型中使用的資料庫連線。

只要模型一定義完成，你就可以開始取得或建立資料到你的資料表了，但這裡必須注意到，預設的情況下，你必須在資料表中建立 `updated_at` 及 `created_at` 這兩個欄位，用來記錄資料的建立時間及更新時間，如果你不希望模型去幫你自動維護資料建立時間及更新時間，在你的模型中你只要將 `$timestamps` 變數設定為 `false` 即可。

**取得所有模型的資料**

	$users = User::all();

**透過主鍵 (Primary Key) 取得單筆資料**

	$user = User::find(1);

	var_dump($user->name);

> **注意:** 所有在 [Query 產生器](/docs/queries) 的方法，在使用 Eloquent 模型時也可以使用。

**透過主鍵 (Primary Key) 取得單筆資料，或丟出例外狀況**

假如模型沒有找到指定的資料，有時你可能想要丟出例外狀況，讓你的例外狀況可以讓 `App::error` 捕捉到，並且呈現 404 頁面給使用者。

	$model = User::findOrFail(1);

	$model = User::where('votes', '>', 100)->firstOrFail();

傾聽 `ModelNotFoundException` 可以註冊一個模型錯誤處理器

	use Illuminate\Database\Eloquent\ModelNotFoundException;

	App::error(function(ModelNotFoundException $e)
	{
		return Response::make('Not Found', 404);
	});

**使用 Eloquent 模型進行查詢**

	$users = User::where('votes', '>', 100)->take(10)->get();

	foreach ($users as $user)
	{
		var_dump($user->name);
	}

當然你也可以使用查詢產生器去整合這些函式。

**Eloquent 整合**

	$count = User::where('votes', '>', 100)->count();

If you are unable to generate the query you need via the fluent interface, feel free to use `whereRaw`:

	$users = User::whereRaw('age > ? and votes = 100', array(25))->get();

**Specifying The Query Connection**

You may also specify which database connection should be used when running an Eloquent query. Simply use the `on` method:

	$user = User::on('connection-name')->find(1);

<a name="mass-assignment"></a>
## 大量指定

當建立模型時，你傳遞一個陣列屬性到模型建構子，這些屬性會經由大量指定 (mass-assignment) 的方式去指定到模型中，這樣是相當方便的，但是，當綁定使用者傳入的資料到模型中，也可能是一個 **嚴重** 的安全性問題，假如使用者傳入的資料綁定到模型，使用者就可以任意的修改 **任何 (any)** 和 **所有 (all)** 模型中的屬性，出於這個原因，所有的 Eloquent 模型預設會避免及保護資料不會被大量指定的方式所覆寫。

為了使用大量指定的功能，你必須在你的模型設定 `fillable` 或 `guarded` 變數資料。

`fillable` 變數指定那些欄位可以使用被大量指定功能指定資料，可以設定類別 (class) 或 實例 (instance) 層級的變數。

**定義可大量指定的屬性欄位到模型**

	class User extends Eloquent {

		protected $fillable = array('first_name', 'last_name', 'email');

	}

在這個範例，只有清單中的三個變數屬性資料可以被使用大量指定方式修改。

在 `fillable` 的反義屬性就是 `guarded`，這屬性可以設定 "黑名單" 而不只是設定 "白名單" :

**定義受保護的屬性欄位到模型**

	class User extends Eloquent {

		protected $guarded = array('id', 'password');

	}

在上述範例 `id` 及 `password` 屬性將 **不會** 被大量指定方式修改原模型的變數資料，所有除了這兩個變數外的變數，都可以被使用大量指定方式指定去修改資料，你也可以保護 **所有** 的屬性都不會被大量指定的方式修改資料值:

**封鎖所有變數不被大量指定方式修改資料內容**

	protected $guarded = array('*');

<a name="insert-update-delete"></a>
## 新增、更新及刪除

為了透過模型建立一筆新的資料到資料庫，只需要建立新的模型實例後，並且呼叫 `save` 方法即可。

**儲存新的模型資料**

	$user = new User;

	$user->name = 'John';

	$user->save();

> ** 注意:** 通常你的 Eloquent 模型都會有一個自動增加的鍵值 (key)，但你如果想要指定你自己的鍵值在模型中有自動增加的屬性，只要將 `incrementing` 設定為 `自動增加 (incrementing)` 即可。

你可以使用 `create` 方法在單一行去儲存資料到模型中，插入 (INSERT) 的模型實例將會被回傳，但是在使用這樣的方式去儲存資料前，你需要在模型中指定 `fillable` 或 `guarded` 屬性，這樣 Eloquent 模型會保護你的模型不被大量指定方式所攻擊。

在儲存或建立一個使用自動遞增 ID 的物件後，你可以存取物件的 `id` 欄位來取得 ID 值：

	$insertedId = $user->id;

**設定保護 (Guarded) 屬性到模型中**

	class User extends Eloquent {

		protected $guarded = array('id', 'account_id');

	}

**使用模型的新增 (Create) 方法**

	$user = User::create(array('name' => 'John'));

為了更新資料，你需要取得資料後，並改變一個你要更新的屬性欄，並使用 `save` 方法即可更新資料:

**更新一個已取得資料的模型**

	$user = User::find(1);

	$user->email = 'john@foo.com';

	$user->save();

有時你會須希望不僅儲存模型中的資料，也希望能夠儲存所有關聯的資料，你可以使用 `push` 的方法，去達到這樣的目的:

**儲存模型資料及關連的資料**

	$user->push();

你也可以對一個集合的模型資料進行更新:

	$affectedRows = User::where('votes', '>', 100)->update(array('status' => 2));

只需要在模型實例使用 `delete` 方法，即可刪除模型的資料:

**刪除一存在的模型資料**

	$user = User::find(1);

	$user->delete();

**透過鍵值刪除一存在的模型資料**

	User::destroy(1);

	User::destroy(array(1, 2, 3));

	User::destroy(1, 2, 3);

單然你也可以對一模型資料集合執行刪除的動作:

	$affectedRows = User::where('votes', '>', 100)->delete();

如果你只想要更新模型的時間戳記，你可以使用 `touch` 方法去進行更新:

**僅更新模型的時間戳記**

	$user->touch();

<a name="soft-deleting"></a>
## 軟刪除

當軟刪除模型資料時，資料不會真的從資料庫中移除，而是會去更新 `deleted_at` 時間戳記欄位，在模型中設定 `softDelete` 變數，就可以讓模型開啟軟刪除的功能::

	class User extends Eloquent {

		protected $softDelete = true;

	}

你可以在 Migration 中使用 `softDeletes` 方法，在資料表中加入 `deleted_at` 欄位:

	$table->softDeletes();

現在當你在模型中呼叫 `delete` 方法時，資料表中的 `deleted_at` 欄位值會被設定為現在的時間戳記，當使用微刪除的模型在對資料庫進行查詢撈取資料時，被 "微刪除 (deleted)" 的資料將不會被撈取出來，如果要強制模型去撈取被微刪除的資料，在查詢過程中使用 `withTrashed` 方法即可:

**強制微刪除的資料也出現在查詢結果中**

	$users = User::withTrashed()->where('account_id', 1)->get();

如果你 **只 (only)** 希望撈取微刪除的資料出來，你可以使用 `onlyTrashed` 去達成這件事:

	$users = User::onlyTrashed()->where('account_id', 1)->get();

使用 `restore` 方法，可以回復原本被微刪除的資料:

	$user->restore();

你可以在查詢過程中使用 `restore` 方法:

	User::withTrashed()->where('account_id', 1)->restore();

`restore` 方法也可以用在關聯的資料上:

	$user->posts()->restore();

使用 `forceDelete` 方法，可以真的將資料從資料庫中移除:

	$user->forceDelete();

`forceDelete` 方法也可以用在關聯資料上:

	$user->posts()->forceDelete();

使用 `trashed` 方法，可以知道模型中是否有被微刪除的資料:

	if ($user->trashed())
	{
		//
	}

<a name="timestamps"></a>
## 時間戳記

預設的情況下， Eloquent 會自動在你的資料表加入並維護 `created_at` 及 `updated_at` 這兩個欄位資料，欄位會被設定為 `datetime` 的資料型態，這兩個欄位的資料異動都交給 Eloquent 去處理，若你不希望 Eloquent 去幫你維護這兩個欄位的資料，在你的模型中將參數 `$timestamps` 設定為 `false` 即可，如下範例所示:

**關閉自動維護時間戳記功能**

	class User extends Eloquent {

		protected $table = 'users';

		public $timestamps = false;

	}

If you wish to customize the format of your timestamps, you may override the `getDateFormat` method in your model:
如果你希望自訂時間戳記格式，你可以在模型中使用 `getDateFormat` 方法去複寫原始設定的格式:

**提供自訂時間戳記格式**

	class User extends Eloquent {

		protected function getDateFormat()
		{
			return 'U';
		}

	}

<a name="query-scopes"></a>
## 查詢範圍

Scopes 允許你容易地在模型中去重複使用查詢邏輯，只要使用 `scope` 當作模型中方法的前綴字，即可定義 Scope:

**定義查詢 Scope**

	class User extends Eloquent {

		public function scopePopular($query)
		{
			return $query->where('votes', '>', 100);
		}

		public function scopeWomen($query)
		{
			return $query->whereGender('W');
		}

	}

**使用查詢 Scope**

	$users = User::popular()->women()->orderBy('created_at')->get();

**Dynamic Scopes**

Sometimes You may wish to define a scope that accepts parameters. Just add your parameters to your scope function:

	class User extends Eloquent {

		public function scopeOfType($query, $type)
		{
			return $query->whereType($type);
		}

	}

Then pass the parameter into the scope call:

	$users = User::ofType('member')->get();

<a name="relationships"></a>
## 關聯

當然，你的資料表總會關連到其他資料表的資料，舉例來說，一篇部落格文章會有數個文章評論，或者會有不同的使用者資料放到不同的排序資料中， Eloquent 可以讓你容易的管理這些關聯關係， Laravel 支援四種不同型態的關聯:

- [一對一 (One To One)](#one-to-one)
- [一對多 (One To Many)](#one-to-many)
- [多對多 (Many To Many)](#many-to-many)
- [多型態的關聯 (Polymorphic Relations)](#polymorphic-relations)

<a name="one-to-one"></a>
### 一對一 (One To One)

一對一的關聯資料是非常基本的關聯，舉例來說 `User` 模型中的使用，有一筆 `Phone` 模型中的電話資料，我們可以在 Eloquent 定義這個關聯關係:

**定義一對一關聯**

	class User extends Eloquent {

		public function phone()
		{
			return $this->hasOne('Phone');
		}

	}

傳送給 `hasOne` 方法的第一個參數是要關聯的模組名稱，只要關聯定義完成，你可以使用 Eloquent 的 [動態屬性](#dynamic-properties) 去取得被關聯的資料:

	$phone = User::find(1)->phone;

SQL 將會執行下列的語法去做查詢:

	select * from users where id = 1

	select * from phones where user_id = 1

這裡要注意到， Eloquent 認定要做關聯的外來鍵 (foreign key) 是基於模組名稱去做設定的，在這個例子中， `Phone` 模型會被認定要用 `user_id` 做關聯時的外來鍵，假如你要變更這個預設的外來鍵名稱設定，你可以在 `hasOne` 方法中的第二個參數傳送你要自訂的外來鍵名稱:

	return $this->hasOne('Phone', 'custom_key');

我們可以使用 `belongsTo` 方法，在 `Phone` 模型中定義反向的關聯:

**定義反向關聯**

	class Phone extends Eloquent {

		public function user()
		{
			return $this->belongsTo('User');
		}

	}

In the example above, Eloquent will look for a `user_id` column on the `phones` table. If you would like to define a different foreign key column, you may pass it as the second argument to the `belongsTo` method:

	class Phone extends Eloquent {

		public function user()
		{
			return $this->belongsTo('User', 'custom_key');
		}

	}

<a name="one-to-many"></a>
### 一對多 (One To Many)

一對多關聯的範例，就像一個部落格文章有"多"個評論，所以我們可以在模型中定義這個關聯關係:

	class Post extends Eloquent {

		public function comments()
		{
			return $this->hasMany('Comment');
		}

	}

現在我們可以透過 [動態屬性](#dynamic-properties)去存取部落格文章的評論資料了:

	$comments = Post::find(1)->comments;

如果需要在取得的評論資料中家更多的限制，可以呼叫 `comments` 方法，並持續使用方法鏈 (chain) 的方式，做更多的條件的設定:

	$comments = Post::find(1)->comments()->where('title', '=', 'foo')->first();

只要在 `hasMany` 第二個參數傳送外來鍵的名稱，即可複寫預設的外來鍵名稱:

	return $this->hasMany('Comment', 'custom_key');

我們可以使用 `belongsTo` 方法，在 `Comment` 模型中定義反向的關聯::

**定義反向關聯**

	class Comment extends Eloquent {

		public function post()
		{
			return $this->belongsTo('Post');
		}

	}

<a name="many-to-many"></a>
### 多對多 (Many To Many)

多對多關聯是一個較複雜的關連類型，舉例來說，像使用者有多種腳色，而相同的腳色可能有數個不同的使用者扮演，就像許多使用者有 "管理者 (Admin)" 的腳色，資料庫中需要三個資料表去表示之間的關係: `users` 、 `roles` 及 `role_user` 這三個資料表，其中 `role_user` 資料表名稱，是根據關聯的兩個資料表 (users 及 roles) 的字母順序去做命名，在 `role_user` 資料表中需要有 `user_id` 及 `role_id` 這兩個欄位。

我們可以使用 `belongsToMany` 方法，去定義多對多的關係:

	class User extends Eloquent {

		public function roles()
		{
			return $this->belongsToMany('Role');
		}

	}

現在我們可以透過 `User` 模型，去取得使用者的角色資料:

	$roles = User::find(1)->roles;

在 `belongsToMany` 方法的第二個參數你可以傳入資料表名稱，使用非預設的名稱當作關聯資料表的名稱:

	return $this->belongsToMany('Role', 'user_roles');

你也可以複寫預設使用的關聯鍵值 (associated keys):

	return $this->belongsToMany('Role', 'user_roles', 'user_id', 'foo_id');

Of course, you may also define the inverse of the relationship on the `Role` model:

	class Role extends Eloquent {

		public function users()
		{
			return $this->belongsToMany('User');
		}

	}

<a name="polymorphic-relations"></a>
### Polymorphic Relations

Polymorphic relations allow a model to belong to more than one other model, on a single association. For example, you might have a photo model that belongs to either a staff model or an order model. We would define this relation like so:

	class Photo extends Eloquent {

		public function imageable()
		{
			return $this->morphTo();
		}

	}

	class Staff extends Eloquent {

		public function photos()
		{
			return $this->morphMany('Photo', 'imageable');
		}

	}

	class Order extends Eloquent {

		public function photos()
		{
			return $this->morphMany('Photo', 'imageable');
		}

	}

Now, we can retrieve the photos for either a staff member or an order:

**Retrieving A Polymorphic Relation**

	$staff = Staff::find(1);

	foreach ($staff->photos as $photo)
	{
		//
	}

However, the true "polymorphic" magic is when you access the staff or order from the `Photo` model:

**Retrieving The Owner Of A Polymorphic Relation**

	$photo = Photo::find(1);

	$imageable = $photo->imageable;

The `imageable` relation on the `Photo` model will return either a `Staff` or `Order` instance, depending on which type of model owns the photo.

To help understand how this works, let's explore the database structure for a polymorphic relation:

**Polymorphic Relation Table Structure**

	staff
		id - integer
		name - string

	orders
		id - integer
		price - integer

	photos
		id - integer
		path - string
		imageable_id - integer
		imageable_type - string

The key fields to notice here are the `imageable_id` and `imageable_type` on the `photos` table. The ID will contain the ID value of, in this example, the owning staff or order, while the type will contain the class name of the owning model. This is what allows the ORM to determine which type of owning model to return when accessing the `imageable` relation.

<a name="querying-relations"></a>
## Querying Relations

When accessing the records for a model, you may wish to limit your results based on the existence of a relationship. For example, you wish to pull all blog posts that have at least one comment. To do so, you may use the `has` method:

**Checking Relations When Selecting**

	$posts = Post::has('comments')->get();

You may also specify an operator and a count:

	$posts = Post::has('comments', '>=', 3)->get();

<a name="dynamic-properties"></a>
### Dynamic Properties

Eloquent allows you to access your relations via dynamic properties. Eloquent will automatically load the relationship for you, and is even smart enough to know whether to call the `get` (for one-to-many relationships) or `first` (for one-to-one relationships) method.  It will then be accessible via a dynamic property by the same name as the relation. For example, with the following model `$phone`:

	class Phone extends Eloquent {

		public function user()
		{
			return $this->belongsTo('User');
		}

	}

	$phone = Phone::find(1);

Instead of echoing the user's email like this:

	echo $phone->user()->first()->email;

It may be shortened to simply:

	echo $phone->user->email;

> **Note:** Relationships that return many results will return an instance of the `Illuminate\Database\Eloquent\Collection` class.

<a name="eager-loading"></a>
## Eager Loading

Eager loading exists to alleviate the N + 1 query problem. For example, consider a `Book` model that is related to `Author`. The relationship is defined like so:

	class Book extends Eloquent {

		public function author()
		{
			return $this->belongsTo('Author');
		}

	}

Now, consider the following code:

	foreach (Book::all() as $book)
	{
		echo $book->author->name;
	}

This loop will execute 1 query to retrieve all of the books on the table, then another query for each book to retrieve the author. So, if we have 25 books, this loop would run 26 queries.

Thankfully, we can use eager loading to drastically reduce the number of queries. The relationships that should be eager loaded may be specified via the `with` method:

	foreach (Book::with('author')->get() as $book)
	{
		echo $book->author->name;
	}

In the loop above, only two queries will be executed:

	select * from books

	select * from authors where id in (1, 2, 3, 4, 5, ...)

Wise use of eager loading can drastically increase the performance of your application.

Of course, you may eager load multiple relationships at one time:

	$books = Book::with('author', 'publisher')->get();

You may even eager load nested relationships:

	$books = Book::with('author.contacts')->get();

In the example above, the `author` relationship will be eager loaded, and the author's `contacts` relation will also be loaded.

### Eager Load Constraints

Sometimes you may wish to eager load a relationship, but also specify a condition for the eager load. Here's an example:

	$users = User::with(array('posts' => function($query)
	{
		$query->where('title', 'like', '%first%');
	}))->get();

In this example, we're eager loading the user's posts, but only if the post's title column contains the word "first".

### Lazy Eager Loading

It is also possible to eagerly load related models directly from an already existing model collection. This may be useful when dynamically deciding whether to load related models or not, or in combination with caching.

	$books = Book::all();

	$books->load('author', 'publisher');

<a name="inserting-related-models"></a>
## Inserting Related Models

You will often need to insert new related models. For example, you may wish to insert a new comment for a post. Instead of manually setting the `post_id` foreign key on the model, you may insert the new comment from its parent `Post` model directly:

**Attaching A Related Model**

	$comment = new Comment(array('message' => 'A new comment.'));

	$post = Post::find(1);

	$comment = $post->comments()->save($comment);

In this example, the `post_id` field will automatically be set on the inserted comment.

### Associating Models (Belongs To)

When updating a `belongsTo` relationship, you may use the `associate` method. This method will set the foreign key on the child model:

	$account = Account::find(10);

	$user->account()->associate($account);

	$user->save();

### Inserting Related Models (Many To Many)

You may also insert related models when working with many-to-many relations. Let's continue using our `User` and `Role` models as examples. We can easily attach new roles to a user using the `attach` method:

**Attaching Many To Many Models**

	$user = User::find(1);

	$user->roles()->attach(1);

You may also pass an array of attributes that should be stored on the pivot table for the relation:

	$user->roles()->attach(1, array('expires' => $expires));

Of course, the opposite of `attach` is `detach`:

	$user->roles()->detach(1);

You may also use the `sync` method to attach related models. The `sync` method accepts an array of IDs to place on the pivot table. After this operation is complete, only the IDs in the array will be on the intermediate table for the model:

**Using Sync To Attach Many To Many Models**

	$user->roles()->sync(array(1, 2, 3));

You may also associate other pivot table values with the given IDs:

**Adding Pivot Data When Syncing**

	$user->roles()->sync(array(1 => array('expires' => true)));

Sometimes you may wish to create a new related model and attach it in a single command. For this operation, you may use the `save` method:

	$role = new Role(array('name' => 'Editor'));

	User::find(1)->roles()->save($role);

In this example, the new `Role` model will be saved and attached to the user model. You may also pass an array of attributes to place on the joining table for this operation:

	User::find(1)->roles()->save($role, array('expires' => $expires));

<a name="touching-parent-timestamps"></a>
## Touching Parent Timestamps

When a model `belongsTo` another model, such as a `Comment` which belongs to a `Post`, it is often helpful to update the parent's timestamp when the child model is updated. For example, when a `Comment` model is updated, you may want to automatically touch the `updated_at` timestamp of the owning `Post`. Eloquent makes it easy. Just add a `touches` property containing the names of the relationships to the child model:

	class Comment extends Eloquent {

		protected $touches = array('post');

		public function post()
		{
			return $this->belongsTo('Post');
		}

	}

Now, when you update a `Comment`, the owning `Post` will have its `updated_at` column updated:

	$comment = Comment::find(1);

	$comment->text = 'Edit to this comment!';

	$comment->save();

<a name="working-with-pivot-tables"></a>
## Working With Pivot Tables

As you have already learned, working with many-to-many relations requires the presence of an intermediate table. Eloquent provides some very helpful ways of interacting with this table. For example, let's assume our `User` object has many `Role` objects that it is related to. After accessing this relationship, we may access the `pivot` table on the models:

	$user = User::find(1);

	foreach ($user->roles as $role)
	{
		echo $role->pivot->created_at;
	}

Notice that each `Role` model we retrieve is automatically assigned a `pivot` attribute. This attribute contains a model representing the intermediate table, and may be used as any other Eloquent model.

By default, only the keys will be present on the `pivot` object. If your pivot table contains extra attributes, you must specify them when defining the relationship:

	return $this->belongsToMany('Role')->withPivot('foo', 'bar');

Now the `foo` and `bar` attributes will be accessible on our `pivot` object for the `Role` model.

If you want your pivot table to have automatically maintained `created_at` and `updated_at` timestamps, use the `withTimestamps` method on the relationship definition:

	return $this->belongsToMany('Role')->withTimestamps();

To delete all records on the pivot table for a model, you may use the `detach` method:

**Deleting Records On A Pivot Table**

	User::find(1)->roles()->detach();

Note that this operation does not delete records from the `roles` table, but only from the pivot table.

<a name="collections"></a>
## Collections

All multi-result sets returned by Eloquent, either via the `get` method or a `relationship`, will return a collection object. This object implements the `IteratorAggregate` PHP interface so it can be iterated over like an array. However, this object also has a variety of other helpful methods for working with result sets.

For example, we may determine if a result set contains a given primary key using the `contains` method:

**Checking If A Collection Contains A Key**

	$roles = User::find(1)->roles;

	if ($roles->contains(2))
	{
		//
	}

Collections may also be converted to an array or JSON:

	$roles = User::find(1)->roles->toArray();

	$roles = User::find(1)->roles->toJson();

If a collection is cast to a string, it will be returned as JSON:

	$roles = (string) User::find(1)->roles;

Eloquent collections also contain a few helpful methods for looping and filtering the items they contain:

**Iterating Collections**

	$roles = $user->roles->each(function($role)
	{
		//
	});

**Filtering Collections**

When filtering collections, the callback provided will be used as callback for [array_filter](http://php.net/manual/en/function.array-filter.php).

	$users = $user->filter(function($user)
	{
		if($user->isAdmin())
		{
			return $user;
		}
	});

> **Note:** When filtering a collection and converting it to JSON, try calling the `values` function first to reset the array's keys.

**Applying A Callback To Each Collection Object**

	$roles = User::find(1)->roles;

	$roles->each(function($role)
	{
		//
	});

**Sorting A Collection By A Value**

	$roles = $roles->sortBy(function($role)
	{
		return $role->created_at;
	});

Sometimes, you may wish to return a custom Collection object with your own added methods. You may specify this on your Eloquent model by overriding the `newCollection` method:

**Returning A Custom Collection Type**

	class User extends Eloquent {

		public function newCollection(array $models = array())
		{
			return new CustomCollection($models);
		}

	}

<a name="accessors-and-mutators"></a>
## Accessors & Mutators

Eloquent provides a convenient way to transform your model attributes when getting or setting them. Simply define a `getFooAttribute` method on your model to declare an accessor. Keep in mind that the methods should follow camel-casing, even though your database columns are snake-case:

**Defining An Accessor**

	class User extends Eloquent {

		public function getFirstNameAttribute($value)
		{
			return ucfirst($value);
		}

	}

In the example above, the `first_name` column has an accessor. Note that the value of the attribute is passed to the accessor.

Mutators are declared in a similar fashion:

**Defining A Mutator**

	class User extends Eloquent {

		public function setFirstNameAttribute($value)
		{
			$this->attributes['first_name'] = strtolower($value);
		}

	}

<a name="date-mutators"></a>
## Date Mutators

By default, Eloquent will convert the `created_at`, `updated_at`, and `deleted_at` columns to instances of [Carbon](https://github.com/briannesbitt/Carbon), which provides an assortment of helpful methods, and extends the native PHP `DateTime` class.

You may customize which fields are automatically mutated, and even completely disable this mutation, by overriding the `getDates` method of the model:

	public function getDates()
	{
		return array('created_at');
	}

When a column is considered a date, you may set its value to a UNIX timetamp, date string (`Y-m-d`), date-time string, and of course a `DateTime` / `Carbon` instance.

To totally disable date mutations, simply return an empty array from the `getDates` method:

	public function getDates()
	{
		return array();
	}

<a name="model-events"></a>
## Model Events

Eloquent models fire several events, allowing you to hook into various points in the model's lifecycle using the following methods: `creating`, `created`, `updating`, `updated`, `saving`, `saved`, `deleting`, `deleted`, `restoring`, `restored`.

Whenever a new item is saved for the first time, the `creating` and `created` events will fire. If an item is not new and the `save` method is called, the `updating` / `updated` events will fire. In both cases, the `saving` / `saved` events will fire.

If `false` is returned from the `creating`, `updating`, `saving`, or `deleting` events, the action will be cancelled:

**Cancelling Save Operations Via Events**

	User::creating(function($user)
	{
		if ( ! $user->isValid()) return false;
	});

Eloquent models also contain a static `boot` method, which may provide a convenient place to register your event bindings.

**Setting A Model Boot Method**

	class User extends Eloquent {

		public static function boot()
		{
			parent::boot();

			// Setup event bindings...
		}

	}

<a name="model-observers"></a>
## Model Observers

To consolidate the handling of model events, you may register a model observer. An observer class may have methods that correspond to the various model events. For example, `creating`, `updating`, `saving` methods may be on an observer, in addition to any other model event name.

So, for example, a model observer might look like this:

	class UserObserver {

		public function saving($model)
		{
			//
		}

		public function saved($model)
		{
			//
		}

	}

You may register an observer instance using the `observe` method:

	User::observe(new UserObserver);

<a name="converting-to-arrays-or-json"></a>
## Converting To Arrays / JSON

When building JSON APIs, you may often need to convert your models and relationships to arrays or JSON. So, Eloquent includes methods for doing so. To convert a model and its loaded relationship to an array, you may use the `toArray` method:

**Converting A Model To An Array**

	$user = User::with('roles')->first();

	return $user->toArray();

Note that entire collections of models may also be converted to arrays:

	return User::all()->toArray();

To convert a model to JSON, you may use the `toJson` method:

**Converting A Model To JSON**

	return User::find(1)->toJson();

Note that when a model or collection is cast to a string, it will be converted to JSON, meaning you can return Eloquent objects directly from your application's routes!

**Returning A Model From A Route**

	Route::get('users', function()
	{
		return User::all();
	});

Sometimes you may wish to limit the attributes that are included in your model's array or JSON form, such as passwords. To do so, add a `hidden` property definition to your model:

**Hiding Attributes From Array Or JSON Conversion**

	class User extends Eloquent {

		protected $hidden = array('password');

	}

> **Note:** When hiding relationships, use the relationship's **method** name, not the dynamic accessor name.

Alternatively, you may use the `visible` property to define a white-list:

	protected $visible = array('first_name', 'last_name');

<a name="array-appends"></a>
Occasionally, you may need to add array attributes that do not have a corresponding column in your database. To do so, simply define an accessor for the value:

	public function getIsAdminAttribute()
	{
		return $this->attributes['admin'] == 'yes';
	}

Once you have created the accessor, just add the value to the `appends` property on the model:

	protected $appends = array('is_admin');

Once the attribute has been added to the `appends` list, it will be included in both the model's array and JSON forms.
