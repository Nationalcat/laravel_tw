---
layout: post
title: quickstart-intermediate
---
# 中級任務清單

- [簡介](#introduction)
- [安裝](#installation)
- [準備資料庫](#prepping-the-database)
	- [資料庫遷移](#database-migrations)
	- [Eloquent 模型](#eloquent-models)
	- [Eloquent 關聯](#eloquent-relationships)
- [路由](#routing)
	- [顯示視圖](#displaying-a-view)
	- [認證](#authentication-routing)
	- [任務控制器](#the-task-controller)
- [建構佈局與視圖](#building-layouts-and-views)
	- [定義佈局](#defining-the-layout)
	- [定義子視圖](#defining-the-child-view)
- [增加任務](#adding-tasks)
	- [驗證](#validation)
	- [建立任務](#creating-the-task)
- [顯示已有的任務](#displaying-existing-tasks)
	- [依賴注入](#dependency-injection)
	- [顯示任務](#displaying-the-tasks)
- [刪除任務](#deleting-tasks)
	- [增加刪除按鈕](#adding-the-delete-button)
	- [路由模型綁定](#route-model-binding)
	- [授權](#authorization)
	- [刪除該筆任務](#deleting-the-task)

<a name="introduction"></a>
## 簡介

這個快速入門導引為 Laravel 框架提供了進階的介紹，內容概括了資料庫遷移、Eloquent ORM、路由、認證、授權、依賴注入、驗證、視圖，及 Blade 樣板。如果你已經熟悉 Laravel 框架的基礎或 PHP 框架，那麼這會是個很好的起始點。

要在 Laravel 功能中為樣本做基本的選擇，我們會建構一個簡單的任務清單，可以使用它追蹤所有想完成的任務（典型的「代辦事項清單」範例）。與「基本」快速入門相比，此教學會允許使用者在應用程式建立帳號並認證。此專案完整並完成的原始碼[在 GitHub 上](http://github.com/laravel/quickstart-intermediate)。

<a name="installation"></a>
## 安裝

當然，首先你需要一個全新安裝的 Laravel 框架。你可以選擇使用 [Homestead 虛擬機器](/laravel_tw/docs/5.1/homestead)或是其他本機 PHP 環境來執行框架。只要你的本機環境準備好，就可以使用 Composer 安裝 Laravel 框架：

	composer create-project laravel/laravel quickstart --prefer-dist

你可以自由的只閱讀快速入門的剩餘部分；不過，如果你想下載這個快速入門的原始碼並在你的本機機器執行，那麼你需要克隆它的 Git 資源庫並安裝依賴：

	git clone https://github.com/laravel/quickstart-intermediate quickstart
	cd quickstart
	composer install
	php artisan migrate

欲了解更多關於建構本機 Laravel 開發環境已完成的文件，請查閱完整的 [Homestead](/laravel_tw/docs/5.1/homestead) 及[安裝](/laravel_tw/docs/5.1/installation)文件。

<a name="prepping-the-database"></a>
## 準備資料庫

<a name="database-migrations"></a>
### 資料庫遷移

首先，讓我們使用遷移定義資料表來容納我們所有的任務。Laravel 的資料庫遷移提供了一個簡單的方式，使用流暢，一目了然的 PHP 程式碼來定義資料表的結構與修改。不必再告訴你的團隊成員手動增加欄位至他們本機的資料庫副本中，你的隊友可以簡單的執行你推送至版本控制的遷移。

#### `users` 資料表

因為我們要讓使用者可以在應用程式中建立他們的帳號，所以我們需要一張資料表來儲存我們的使用者。值得慶幸的是，Laravel 已經附帶了建立 `users` 資料表的遷移，所以我們不必手動產生一個。預設的 `users` 資料表遷移位於 `database/migrations` 目錄中。

#### `tasks` 資料表

接著，讓我們建構一張我們將容納所有任務的資料表。[Artisan 指令列介面](/laravel_tw/docs/5.1/artisan)可以被用於產生各種類別，為你建構 Laravel 專案時節省大量的手動輸入。在此例中，讓我們使用 `make:migration` 指令為 `tasks` 資料表產生新的資料庫遷移：

	php artisan make:migration create_tasks_table --create=tasks

此遷移會被放置於你專案的 `database/migrations` 目錄中。你可能已經注意到，`make:migration` 指令已經增加了自動遞增的 ID 及時間戳記至遷移檔。讓我們編輯這個檔案並為任務的名稱增加額外的 `string` 欄位，也增加連結 `tasks` 與 `users` 資料表的 `user_id` 欄位：

	<?php

	use Illuminate\Database\Schema\Blueprint;
	use Illuminate\Database\Migrations\Migration;

	class CreateTasksTable extends Migration
	{
	    /**
	     * 執行遷移。
	     *
	     * @return void
	     */
	    public function up()
	    {
	        Schema::create('tasks', function (Blueprint $table) {
	            $table->increments('id');
	            $table->integer('user_id')->index();
	            $table->string('name');
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
	        Schema::drop('tasks');
	    }
	}

我們可以使用 `migrate` Artisan 指令執行遷移。如果你使用 Homestead，你必須在你的虛擬機器執行這個指令，因為你的主機無法直接存取資料庫：

	php artisan migrate

這個指令會建立我們所有的資料表。如果你使用你選擇的資料庫客戶端檢查資料表，你應該看到新的 `tasks` 與 `users` 資料表，並包含了我們遷移中所定義的欄位。接著，我們已經準備好定義我們的 Eloquent ORM 模型！

<a name="eloquent-models"></a>
### Eloquent 模型

[Eloquent](/laravel_tw/docs/5.1/eloquent) 是 Laravel 預設的 ORM（物件關聯對映）。Eloqunet 透過明確的定義「模型」，讓你無痛的在資料庫取得及儲存資料。一般情況下，每個 Eloqunet 模型會直接對應於一張資料表。

#### `User` 模型

首先，我們需要對應至 `users` 資料表的模型。不過，如果你看過你專案的 `app` 目錄，你會發現 Laravel 已經附帶了一個 `User` 模型，所以我們不必手動產生。

#### `Task` 模型

所以，讓我們定義一個對應至 `tasks` 資料表的 `Task` 模型。同樣的，我們可以使用 Artisan 指令來產生此模型。在此例中，我們會使用 `make:model` 指令：

	php artisan make:model Task

這個模型會放置在你應用程式的 `app` 目錄中。預設中，此模型類別會是空的。我們不必明確告知 Eloquent 模型要對應哪張資料表，因為它會假設資料表是模型名稱的複數型態。所以，在此例中，`Task` 模型會假設對應至 `tasks` 資料表。

讓我們增加一些東西至模型。首先，我們需要宣告模型的 `name` 屬性應該能被「批量賦值」：

	<?php

	namespace App;

	use Illuminate\Database\Eloquent\Model;

	class Task extends Model
	{
	    /**
	     * 這些屬性能被批量賦值。
	     *
	     * @var array
	     */
	    protected $fillable = ['name'];
	}

在為我們的應用程式增加路由時，我們會學習更多關於如何使用 Eloquent 模型。當然，你可以很自由的參考[完整的 Eloquent 文件](/laravel_tw/docs/5.1/eloquent)取得更多資訊。

<a name="eloquent-relationships"></a>
### Eloquent 關聯

現在我們的模型已經定義好，我們需要將他們連結在一起。例如，我們的 `User` 可以擁有多筆 `Task` 實例，而一筆 `Task` 則被賦予給一名 `User`。定義關聯可以讓我們流暢的走過關聯，像這樣：

	$user = App\User::find(1);

	foreach ($user->tasks as $task) {
		echo $task->name;
	}

#### `tasks` 關聯

首先，讓我們在 `User` 模型定義 `tasks` 的關聯。Eloquent 關聯被定義為模型中的方法。Eloquent 支援多種不同類型的關聯，所以請務必查閱[完整的 Eloquent 文件](/laravel_tw/docs/5.1/eloquent-relationships)取得更多資訊。在本例中，我們會在 `User` 模型中定義一個 `tasks` 函式，並呼叫 Eloquent 提供的 `hasMany` 方法：

	<?php

	namespace App;

	// 導入的命名空間...

	class User extends Model implements AuthenticatableContract,
	                                    AuthorizableContract,
	                                    CanResetPasswordContract
	{
	    use Authenticatable, Authorizable, CanResetPassword;

	    // 其它的 Eloquent 屬性...

	    /**
	     * 取得該使用者的所有任務。
	     */
	    public function tasks()
	    {
	        return $this->hasMany(Task::class);
	    }
	}

#### `user` 關聯

接著，讓我們在 `Task` 模型定義 `user` 關聯。同樣的，我們會將此關聯定義為模型中的方法。在本例中，我們會使用 Eloquent 提供的 `belongsTo` 方法來定義關聯：

	<?php

	namespace App;

	use App\User;
	use Illuminate\Database\Eloquent\Model;

	class Task extends Model
	{
	    /**
	     * 這些屬性能被批量賦值。
	     *
	     * @var array
	     */
	    protected $fillable = ['name'];

	    /**
	     * 取得擁有此任務的使用者。
	     */
	    public function user()
	    {
	        return $this->belongsTo(User::class);
	    }
	}

太棒了！現在我們的關聯已經定義好了，我們可以開始建構我們的控制器！

<a name="routing"></a>
## 路由

在我們任務清單應用程式的[基本版本](/laravel_tw/docs/5.1/quickstart)中，我們在我們的 `routes.php` 中將所有邏輯都定義為閉包。對於大多數的應用程式來說，我們會使用[控制器](/laravel_tw/docs/5.1/controllers)來組織我們的路由。控制器讓我們將 HTTP 請求處理邏輯分散至多個檔案以進行更好的組織。

<a name="displaying-a-view"></a>
### 顯示視圖

我們只會有一個路由使用閉包：`/` 路由，它會是個給應用程式訪客的簡單起始頁面。所以，讓我們填寫我們的 `/` 路由。對於此路由，我們想渲染一個包含「歡迎」頁面的 HTML 模板：

在 Laravel 裡，所有的 HTML 樣板都儲存在 `resources/views` 目錄，且我們可以在路由中使用 `view` 輔助方法來回傳這些樣板的其中一個：

	Route::get('/', function () {
		return view('welcome');
	});

當然，我們必須確實的定義這些視圖。我們將在稍後完成！

<a name="authentication-routing"></a>
### 認證

還記得我們需要讓使用者建立帳號並登入至我們的應用程式。一般來說，為網頁應用程式建構完整的認證是相當乏味的工作。不過，因為它是一個通用的需求，所以 Laravel 試著讓這個過程變得無痛。

首先，你會注意到在應用程式中已經包含一個 `app/Http/Controllers/Auth/AuthController`。這個控制器使用了特別的 `AuthenticatesAndRegistersUsers` trait，它包含了所有建立及認證使用者必要的邏輯。

#### 認證路由

所以，還有哪些部分是留著讓我們做的？好的，我們依然需要建立註冊及登入模板，與定義指向認證控制器的路由。首先，讓我們在我們的 `app/Http/routes.php` 檔案中增加需要的路由：

	// 認證路由...
	Route::get('auth/login', 'Auth\AuthController@getLogin');
	Route::post('auth/login', 'Auth\AuthController@postLogin');
	Route::get('auth/logout', 'Auth\AuthController@getLogout');

	// 註冊路由...
	Route::get('auth/register', 'Auth\AuthController@getRegister');
	Route::post('auth/register', 'Auth\AuthController@postRegister');

#### 認證視圖

認證需要在 `resources/views/auth` 目錄中建立 `login.blade.php` 與 `register.blade.php`。當然，這些視圖的設計及樣式是不重要的；不過，它們至少必須包含一些基本的欄位。

`register.blade.php` 檔案必需包含一個表單，包含 `name`、`email`、`password` 與 `password_confirmation` 欄位，並製造一個 `POST` 請求至 `/auth/register` 路由。

`login.blade.php` 檔案必需包含一個表單，包含 `email` 與 `password` 欄位，並製造一個 `POST` 請求至 `/auth/login`。

> **注意：**如果你想查看這些視圖的完整範例，請記得應用程式的完整原始碼可以在 [GitHub 上取得](https://github.com/laravel/quickstart-intermediate)。

<a name="the-task-controller"></a>
### 任務控制器

因為我們已經知道我們需要取得及儲存任務，所以讓我們使用 Artisan 指令列介面建立一個 `TaskController`，這個新的控制器會放置在 `app/Http/Controllers` 目錄中：

	php artisan make:controller TaskController --plain

現在這個控制器已經被產生，讓我們繼續在 `app/Http/routes.php` 檔案中建置一些對應至此控制器的路由：

	Route::get('/tasks', 'TaskController@index');
	Route::post('/task', 'TaskController@store');
	Route::delete('/task/{task}', 'TaskController@destroy');

#### 認證所有的任務路由

對於此應用程式，我們希望我們所有的任務路由需要一個認證的使用者。換句話說，使用者為了建立任務必須「登入至」應用程式中。所以，我們需要限制我們的任務路由僅限已認證的使用者存取。Laravel 使用[中介層](/laravel_tw/docs/5.1/middleware)讓這件事變得相當容易。

要讓所有控制器中的行為要求已認證的使用者，我們可以在控制器的建構子中增加 `middleware` 方法的呼叫。所以可用的路由中介層都被定義在 `app/Http/Kernel.php` 檔案中。在本例中，我們希望為所有控制器的動作指派 `auth` 中介層：

	<?php

	namespace App\Http\Controllers;

	use App\Http\Requests;
	use Illuminate\Http\Request;
	use App\Http\Controllers\Controller;

	class TaskController extends Controller
	{
	    /**
	     * 建立一個新的控制器實例。
	     *
	     * @return void
	     */
	    public function __construct()
	    {
	        $this->middleware('auth');
	    }
	}

<a name="building-layouts-and-views"></a>
## 建構佈局與視圖

這個應用程式只會有一張視圖，包含新增任務的表單，及目前所有任務的清單。為了幫助你想像此視圖，下方是完成後應用程式的截圖，套用了基本的 Bootstrap CSS 樣式：

![應用程式圖片](https://laravel.tw/assets/img/quickstart/basic-overview.png)

<a name="defining-the-layout"></a>
### 定義佈局

幾乎所有的網頁應用程式都會在不同頁面共用相同的佈局。舉個例子，應用程式通常在每個頁面（如果我們有一個以上）的頂部都擁有導航欄。Laravel 使用了 Blade **佈局**讓不同頁面共用這些相同的功能。

如同我們前面討論，Laravel 所有的視圖都被儲存在 `resources/views`。所以，讓我們定義一個新的佈局視圖至 `resources/views/layouts/app.blade.php`。`.blade.php` 副檔名會告知框架使用 [Blade 模板引擎](/laravel_tw/docs/5.1/blade)渲染此視圖。當然，你可以在 Laravel 使用純 PHP 的樣板。不過，Blade 提供了方便的簡寫來撰寫乾淨、簡潔的模板。

我們的 `app.blade.php` 視圖看起來應該如下：

    // resources/views/layouts/app.blade.php

	<!DOCTYPE html>
	<html lang="en">
		<head>
			<title>Laravel 快速入門 - 進階</title>

			<!-- CSS 及 JavaScript -->
		</head>

		<body>
			<div class="container">
				<nav class="navbar navbar-default">
					<!-- Navbar 內容 -->
				</nav>
			</div>

			@yield('content')
		</body>
	</html>

注意佈局中的 `@yield('content')` 部分。這是特別的 Blade 指令，讓子頁面可以在此處注入自己的內容以延伸佈局。接著，讓我們定義將會使用此佈局並提供主要內容的子視圖。

<a name="defining-the-child-view"></a>
### 定義子視圖

很好，我們的應用程式佈局已經完成。接下來，我們需要定義包含建立任務的表單及列出已有任務表格的視圖。讓我們將此視圖定義在 `resources/views/tasks/index.blade.php`，它會對應至我們 `TaskController` 的 `index` 方法。

我們會跳過一些 Bootstrap CSS 樣板，只專注在重要的事物上。切記，你可以在 [GitHub](https://github.com/laravel/quickstart-intermediate) 下載應用程式的完整原始碼：

    // resources/views/tasks/index.blade.php

	@extends('layouts.app')

	@section('content')

		<!-- Bootstrap 樣板... -->

		<div class="panel-body">
			<!-- 顯示驗證錯誤 -->
			@include('common.errors')

			<!-- 新任務的表單 -->
			<form action="/task" method="POST" class="form-horizontal">
				{% raw %} {{ csrf_field() }} {% endraw %}

				<!-- 任務名稱 -->
				<div class="form-group">
					<label for="task-name" class="col-sm-3 control-label">任務</label>

					<div class="col-sm-6">
						<input type="text" name="name" id="task-name" class="form-control">
					</div>
				</div>

				<!-- 增加任務按鈕-->
				<div class="form-group">
					<div class="col-sm-offset-3 col-sm-6">
						<button type="submit" class="btn btn-default">
							<i class="fa fa-plus"></i> 增加任務
						</button>
					</div>
				</div>
			</form>
		</div>

		<!-- 代辦：目前任務 -->
	@endsection

#### 一些注意事項的說明

在繼續之前，讓我們談談有關模板的一些事項。首先 `@extends` 指令會告知 Blade，我們使用定義於 `resources/views/layouts/app.blade.php` 的佈局。所有在 `@section('content')` 及 `@endsection` 之間的內容會被注入到 `app.blade.php` 佈局中的 `@yield('content')` 位置裡。

現在我們已經為我們的應用程式定義了基本的佈局及視圖。接著讓我們在 `TaskController` 的 `index` 方法回傳此視圖：

    /**
     * 顯示使用者所有任務的清單。
     *
     * @param  Request  $request
     * @return Response
     */
	public function index(Request $request)
	{
		return view('tasks.index');
	}

接著，我們已經準備好增加程式碼至我們的 `POST /task` 路由的控制器方法，以處理接收到的表單輸入並增加新的任務至資料庫中。

> **注意：**`@include('common.errors')` 指令會載入位於 `resources/views/common/errors.blade.php` 的模板。我們尚未定義此模板，但是我們將會在稍後定義它！

<a name="adding-tasks"></a>
## 增加任務

<a name="validation"></a>
### 驗證

現在我們視圖中已經有一個表單，我們需要增加程式碼至我們的 `TaskController@store` 方法來驗證接收到的表單輸入並建立新的任務。首先，讓我們驗證輸入。

對此表單來說，我們要讓 `name` 欄位為必填，且它必須少於 `255` 字元。如果驗證失敗，我們會將使用者重導回 `/` URL，並將舊的輸入及錯誤訊息快閃至 [session](/laravel_tw/docs/5.1/session)：

    /**
     * 建立新的任務。
     *
     * @param  Request  $request
     * @return Response
     */
	public function store(Request $request)
	{
		$this->validate($request, [
			'name' => 'required|max:255',
		]);

		// Create The Task...
	}

如果你跟隨過[基本快速入門](/laravel_tw/docs/5.1/quickstart)，你會注意到驗證的程式碼相比起來有些不同！因為我們在控制器內，所以我們可以利用 Laravel 基底控制器所包含方便的 `ValidatesRequests` trait。這個 trait 提供了一個簡單的 `validate` 方法，它接收一個請求與包含驗證規則的陣列。

我們不必再手動判斷當驗證失敗時需手動重導。如果給定的規則驗證失敗，使用者會自動被重導回原本的位置，並自動將錯誤訊息快閃至 session 中。很好！

#### `$errors` 變數

記得我們在視圖中使用了 `@include('common.errors')` 指令來渲染表單的驗證錯誤訊息。`common.errors` 讓我們可以簡單的在我們所有的頁面顯示相同格式的驗證錯誤訊息。現在讓我們定義此視圖的內容：

    // resources/views/common/errors.blade.php

    @if (count($errors) > 0)
        <!-- 表單錯誤清單 -->
        <div class="alert alert-danger">
            <strong>哎呀！出了些問題！</strong>

            <br><br>

            <ul>
                @foreach ($errors->all() as $error)
                    <li>{% raw %} {{ $error }} {% endraw %}</li>
                @endforeach
            </ul>
        </div>
    @endif


> **注意：**`errors` 變數可用於**每個** Laravel 的視圖中。如果沒有驗證錯誤訊息存在，那麼它就會是一個空的 `ViewErrorBag` 實例。

<a name="creating-the-task"></a>
### 建立任務

現在輸入已經被驗證處理完畢。讓我們繼續填寫我們的路由來實際的建立一筆新的任務。一旦新的任務被建立後，我們會將使用者重導回 `/tasks` URL。要建立該任務，我們會充分的利用 Eloquent 的關聯功能。

Laravel 大部分的關聯提供了一個 `create` 方法，它接收一個包含屬性的陣列，並會在儲存至資料庫前自動設置關聯模型的外鍵值。在此例中，`create` 方法會自動將給定任務的 `user_id` 屬性設置為目前已驗證使用者的 ID，因為我們透過 `$request->user()` 存取。

    /**
     * 建立新的任務。
     *
     * @param  Request  $request
     * @return Response
     */
    public function store(Request $request)
    {
        $this->validate($request, [
            'name' => 'required|max:255',
        ]);

        $request->user()->tasks()->create([
            'name' => $request->name,
        ]);

        return redirect('/tasks');
    }

好極了！我們現在可以成功的建立任務。接著，讓我們繼續建構已有的任務清單，並增加至我們的視圖中。

<a name="displaying-existing-tasks"></a>
### 顯示已有的任務

首先，我們需要編輯我們的 `TaskController@index` 方法，以傳遞所有已有的任務至視圖。`view` 函式接收一個能在視圖中取用之資料的陣列作為第二個參數，陣列中的每個鍵都會在視圖中作為變數。For example, we could do this:

    /**
     * Display a list of all of the user's task.
     *
     * @param  Request  $request
     * @return Response
     */
    public function index(Request $request)
    {
    	$tasks = Task::where('user_id', $request->user()->id)->get();

        return view('tasks.index', [
            'tasks' => $tasks,
        ]);
    }

不過，讓我們來探討一些 Laravel 的依賴注入功能，注入 `TaskRepository` 至我們的 `TaskController`，我們會透過它存取所有的資料。

<a name="dependency-injection"></a>
### 依賴注入

Laravel 的[服務容器](/laravel_tw/docs/5.1/container)是整個框架中最強大的功能之一。在讀完本快速上手之後，請務必閱讀容器文件的全部。

#### 建立資源庫

如前面所提，我們希望定義一個 `TaskRepository` 存放所有 `Task` 模型的資料存取邏輯。當應用程式擴增，且你需要在整個應用程式中共用同樣的 Eloquent 查詢，那麼這會是相當有用的。

所以，讓我們建立一個 `app/Repositories` 目錄，並增加 `TaskRepository` 類別。切記，Laravel 的 `app` 中所有的資料夾會自動載入並使用 PSR-4 自動載入標準，所以你可以自由的建立許多需要的額外目錄。

	<?php

	namespace App\Repositories;

	use App\User;
	use App\Task;

	class TaskRepository
	{
	    /**
	     * 取得給定使用者的所有任務。
	     *
	     * @param  User  $user
	     * @return Collection
	     */
	    public function forUser(User $user)
	    {
	        return Task::where('user_id', $user->id)
	                    ->orderBy('created_at', 'asc')
	                    ->get();
	    }
	}

#### 注入資源庫

一旦我們的資源庫定義好，我們可以簡單的在 `TaskController` 控制器的建構子中對它使用「型別提示」，並在我們的 `index` 路由中使用它。因為 Laravel 使用容器來解析所有的控制器，所以我們的依賴會自動被注入至控制器的實例中：

	<?php

	namespace App\Http\Controllers;

	use App\Task;
	use App\Http\Requests;
	use Illuminate\Http\Request;
	use App\Http\Controllers\Controller;
	use App\Repositories\TaskRepository;

	class TaskController extends Controller
	{
	    /**
	     * 任務資源庫的實例。
	     *
	     * @var TaskRepository
	     */
	    protected $tasks;

	    /**
	     * 建立新的控制器實例。
	     *
	     * @param  TaskRepository  $tasks
	     * @return void
	     */
	    public function __construct(TaskRepository $tasks)
	    {
	        $this->middleware('auth');

	        $this->tasks = $tasks;
	    }

	    /**
	     * 取得給定使用者的所有任務。
	     *
	     * @param  Request  $request
	     * @return Response
	     */
	    public function index(Request $request)
	    {
	        return view('tasks.index', [
	            'tasks' => $this->tasks->forUser($request->user()),
	        ]);
	    }
	}

<a name="displaying-the-tasks"></a>
### 顯示任務

一旦資料被傳遞之後，我們在我們的 `tasks/index.blade.php` 視圖中將任務切分並將它們顯示至表格中。`@foreach` 指令結構讓我們可以撰寫簡潔的迴圈，並編譯成快速的純 PHP 程式碼：

	@extends('layouts.app')

	@section('content')
        <!-- 建立任務表單... -->

        <!-- 目前任務 -->
        @if (count($tasks) > 0)
            <div class="panel panel-default">
                <div class="panel-heading">
                   目前任務
                </div>

                <div class="panel-body">
                    <table class="table table-striped task-table">

                        <!-- 表頭 -->
                        <thead>
                            <th>Task</th>
                            <th>&nbsp;</th>
                        </thead>

                        <!-- 表身 -->
                        <tbody>
                            @foreach ($tasks as $task)
                                <tr>
                                    <!-- 任務名稱 -->
                                    <td class="table-text">
                                        <div>{% raw %} {{ $task->name }} {% endraw %}</div>
                                    </td>

                                    <td>
                                       <!-- 代辦：刪除按鈕 -->
                                    </td>
                                </tr>
                            @endforeach
                        </tbody>
                    </table>
                </div>
            </div>
        @endif
	@endsection

我們任務應用程式大部分都完成了。但是，當我們完成已有的任務後，還沒有任何方式可以刪除它們。接著讓我們增加此功能！

<a name="deleting-tasks"></a>
## 刪除任務

<a name="adding-the-delete-button"></a>
### 增加刪除按鈕

我們在我們的程式碼中應該放刪除按鈕的地方留下了「待辦」的事項。所以，讓我們在 `tasks/index.blade.php` 視圖中列出任務的每一行增加一個刪除按鈕。我們會為列表中的每個任務建立一個只有單個按鈕的小表單。當按鈕被按下時，一個 `DELETE /task` 的請求將會被發送到應用程式，它會觸發我們的 `TaskController@destroy` 方法：

    <tr>
        <!-- 任務名稱 -->
        <td class="table-text">
            <div>{% raw %} {{ $task->name }} {% endraw %}</div>
        </td>

        <!-- 刪除按鈕 -->
        <td>
            <form action="/task/{% raw %} {{ $task->id }} {% endraw %}" method="POST">
                {% raw %} {{ csrf_field() }} {% endraw %}
                {% raw %} {{ method_field('DELETE') }} {% endraw %}

                <button>刪除任務</button>
            </form>
        </td>
    </tr>

<a name="a-note-on-method-spoofing"></a>
#### 方法欺騙的註記

注意，刪除按鈕的表單 `method` 被列為 `POST`，即使我們回應的請求使用了 `Route::delete` 路由。HTML 表單只允許 `GET` 及 `POST` HTTP 動詞，所以我們需要有個方式在表單假冒一個 `DELETE` 請求。

我們可以在表單中透過 `method_field('DELETE')` 函式輸出的結果假冒一個 `DELETE` 請求。此函式會產生一個隱藏的表單輸入，Laravel 會辨識並覆蓋掉實際使用的 HTTP 請求方法。產生的欄位看起來如下：

	<input type="hidden" name="_method" value="DELETE">

<a name="route-model-binding"></a>
### 路由模型綁定

現在，我們大致上已經準備好定義我們 `TaskController` 的 `destroy` 方法。但是首先，讓我們重新審視我們為它宣告的路由：

	Route::delete('/task/{task}', 'TaskController@destroy');

不增加任何額外的程式碼，Laravel 會注入給定的任務 ID 至 `TaskController@destroy` 方法中，如下：

    /**
     * Destroy the given task.
     *
     * @param  Request  $request
     * @param  string  $taskId
     * @return Response
     */
	public function destroy(Request $request, $taskId)
	{
		//
	}

但是，我們要在這個方法中做的第一件事，就是透過給定的 ID 從資料庫中取得 `Task` 實例。所以，如果 Laravel 可以先注入與 ID 符合的 `Task` 實例，那豈不是很棒？讓我們做到這一點！

在你的 `app/Providers/RouteServiceProvider.php` 檔案的 `boot` 方法中，讓我們增加下方這行程式碼：

	$router->model('task', 'App\Task');

這一小行的程式碼會告知 Laravel，若在路由宣告中看見 `{task}`，就會取得與給定 ID 對應的 `Task` 模型。現在我們可以定義我們的 destroy 方法，如下：

    /**
     * 卸除給定的任務。
     *
     * @param  Request  $request
     * @param  Task  $task
     * @return Response
     */
    public function destroy(Request $request, Task $task)
    {
        //
    }

<a name="authorization"></a>
### 認證

現在，我們有一個注入至 `destroy` 方法的 `Task` 實例；然而，我們不能保證通過認證的使用者實際上「擁有」給定的任務。舉個例子，一個惡意的請求可能透過傳遞一個隨機任務 ID 至 `/tasks/{task}` URL，企圖嘗試刪除其他使用者的任務。所以，我們需要使用 Laravel 的授權功能，以確保已認證的使用者實際上擁有注入至路由的 `Task` 實例。

#### 建立一個原則

Laravel 使用了「原則」將授權邏輯組織至簡單，小型的類別。一般來說，每個原則會對應至一個模型。所以，讓我們使用 Artisan 指令列介面建立一個 `TaskPolicy`，產生的檔案會被放置於 `app/Policies/TaskPolicy.php`：

	php artisan make:policy TaskPolicy

接著，讓我們增加一個 `destroy` 方法治原則中。此方法會取得一個 `User` 實例及一個 `Task` 實例。此方法會簡單的檢查當使用者的 ID 符合任務的 `user_id`。實際上，所有的授權方法必須回傳 `true` 或 `false`：

	<?php

	namespace App\Policies;

	use App\User;
	use App\Task;
	use Illuminate\Auth\Access\HandlesAuthorization;

	class TaskPolicy
	{
	    use HandlesAuthorization;

	    /**
	     * 判斷當給定的使用者可以刪除給定的任務。
	     *
	     * @param  User  $user
	     * @param  Task  $task
	     * @return bool
	     */
	    public function destroy(User $user, Task $task)
	    {
	        return $user->id === $task->user_id;
	    }
	}

最後，我們需要連接我們的 `Task` 模型與 `TaskPolicy`。我們可以透過增加一行至 `app/Providers/AuthServiceProvider.php` 檔案的 `$policies` 屬性做到這件事。這會告知 Laravel，每當我們嘗試授權 `Task` 實例的行為時該用哪個原則：

    /**
     * 應用程式的原則對應。
     *
     * @var array
     */
    protected $policies = [
        Task::class => TaskPolicy::class,
    ];


#### 授權行為

現在我們的原則已經撰寫完，讓我們在我們的 `destroy` 方法中使用它。Laravel 所有的控制器可以呼叫一個 `authorize` 方法，它由 `AuthorizesRequest` trait 所提供：

    /**
     * 卸除給定的任務。
     *
     * @param  Request  $request
     * @param  Task  $task
     * @return Response
     */
    public function destroy(Request $request, Task $task)
    {
        $this->authorize('destroy', $task);

        // 刪除該任務...
    }

讓我們花點時間看看此方法的呼叫。傳遞至 `authorize` 的第一個參數是我們希望呼叫的原則方法名稱。第二個參數是我們目前有關的模型實例。切記，我們已經告訴 Laravel 我們的 `Task` 模型會對應至我們的 `TaskPolicy`，所以框架會知道該觸發哪個原則的 `destroy` 方法。目前的使用者會自動發送至授權的方法中，所以我們不必在此手動傳遞它。

如果該行為被授權了，我們的程式碼會繼續正常執行。但是，如果該行為不被授權（意指原則的 `destroy` 方法回傳 `false`），會自動被拋出一個 403 例外並顯示錯誤頁面給使用者。

> **注意：**Laravel 提供的授權服務還有多種方式可供互動。請務必瀏覽完整的[授權文件](/laravel_tw/docs/5.1/authorization)。

<a name="deleting-the-task"></a>
### 刪除該任務

最後，讓我們完成增加邏輯至我們的 `destroy` 方法來實際刪除給定的任務。我們可以使用 Eloquent 的 `delete` 方法從資料庫中刪除給定的模型實例。一旦記錄被刪除，我們會將使用者重導回 `tasks` URL：

    /**
     * 卸除給定的任務。
     *
     * @param  Request  $request
     * @param  Task  $task
     * @return Response
     */
    public function destroy(Request $request, Task $task)
    {
        $this->authorize('destroy', $task);

        $task->delete();

        return redirect('/tasks');
    }
