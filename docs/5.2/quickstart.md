---
layout: post
title: quickstart
---
# 基本任務清單

- [簡介](#introduction)
- [安裝](#installation)
- [準備資料庫](#prepping-the-database)
	- [資料庫遷移](#database-migrations)
	- [Eloquent 模型](#eloquent-models)
- [路由](#routing)
	- [建置路由](#stubbing-the-routes)
	- [顯示視圖](#displaying-a-view)
- [建構佈局與視圖](#building-layouts-and-views)
	- [定義佈局](#defining-the-layout)
	- [定義子視圖](#defining-the-child-view)
- [增加任務](#adding-tasks)
	- [驗證](#validation)
	- [建立任務](#creating-the-task)
	- [顯示已有的任務](#displaying-existing-tasks)
- [刪除任務](#deleting-tasks)
	- [增加刪除按鈕](#adding-the-delete-button)
	- [刪除該筆任務](#deleting-the-task)

<a name="introduction"></a>
## 簡介

這個快速入門導引為 Laravel 框架提供了基本的介紹，內容概括了資料庫遷移、Eloquent ORM、路由、驗證、視圖，及 Blade 樣板。如果你是第一次使用 Laravel 框架或 PHP 框架，那麼這會是個很好的起始點。如果你已經在使用 Laravel 或者其它的框架，不妨參考我們進階的快速入門。

要在 Laravel 功能中為樣本做基本的選擇，我們會建構一個簡單的任務清單，可以使用它追蹤所有想完成的任務。換句話說，就是典型的「代辦事項清單」範例。此專案完整並完成的原始碼[在 GitHub 上](http://github.com/laravel/quickstart-basic)。

<a name="installation"></a>
## 安裝

#### 安裝 Laravel

當然，首先你需要一個全新安裝的 Laravel 框架。你可以選擇使用 [Homestead 虛擬機器](/laravel_tw/docs/5.2/homestead)或是其他本機 PHP 環境來執行框架。只要你的本機環境準備好，就可以使用 Composer 安裝 Laravel 框架：

	composer create-project laravel/laravel quickstart --prefer-dist

#### 安裝此快速入門（選擇性）

你可以選擇只閱讀快速入門的剩餘部分；不過，如果你想下載這個快速入門的原始碼並在你的本機機器執行，那麼你需要 clone 它的 Git 資源庫並安裝依賴套件：

	git clone https://github.com/laravel/quickstart-basic quickstart
	cd quickstart
	composer install
	php artisan migrate

欲了解更多關於建構本機 Laravel 開發環境的完整文件，請查閱 [Homestead](/laravel_tw/docs/5.2/homestead) 及[安裝](/laravel_tw/docs/5.2/installation)的整個文件。

<a name="prepping-the-database"></a>
## 準備資料庫

<a name="database-migrations"></a>
### 資料庫遷移

首先，讓我們使用遷移定義資料表來容納我們所有的任務。Laravel 的資料庫遷移提供了一個簡單的方式，使用流暢、一目了然的 PHP 程式碼來定義資料表的結構與修改。不必再告訴你的團隊成員手動增加欄位至他們本機的資料庫副本中，你的團隊成員可以簡單的執行你提交至版本控制的遷移。

所以，讓我們建構來一張我們容納所有任務的資料表。[Artisan 指令列介面](/laravel_tw/docs/5.2/artisan)可以被用於產生各種類別，在建構 Laravel 專案時為你節省大量的手動輸入。在這個範例中，讓我們使用 `make:migration` 指令為 `tasks` 資料表產生新的資料庫遷移：

	php artisan make:migration create_tasks_table --create=tasks

此遷移會被放置於你專案的 `database/migrations` 目錄中。你可能已經注意到，`make:migration` 指令已經增加了自動遞增的 ID 及時間戳記至遷移檔。讓我們編輯這個檔案並為任務的名稱增加額外的 `string` 欄位：

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

我們將會使用 `migrate` Artisan 指令執行遷移。如果你使用 Homestead，你必須在你的虛擬機器執行這個指令，因為你的主機無法直接存取資料庫：

	php artisan migrate

這個指令會建立我們所有的資料表。如果你使用你選擇的資料庫客戶端去檢查資料表，你應該看到新的 `tasks` 資料表，並包含了我們遷移中所定義的欄位。接著，我們已經準備好為我們的任務定義 Eloquent ORM 模型！

<a name="eloquent-models"></a>
### Eloquent 模型

[Eloquent](/laravel_tw/docs/5.2/eloquent) 是 Laravel 預設的 ORM（物件關聯對映）。Eloqunet 透過明確的定義「模型」，讓你無痛的在資料庫取得及儲存資料。一般情況下，每個 Eloqunet 模型會直接對應一張資料表。

所以，讓我們定義一個對應至 `tasks` 資料表的 `Task` 模型。同樣的，我們可以使用 Artisan 指令來產生此模型。在這個範例中，我們會使用 `make:model` 指令：

	php artisan make:model Task

這個模型會放置在你應用程式的 `app` 目錄中。預設中，此模型類別會是空的。我們不必明確告知 Eloquent 模型要對應哪張資料表，因為它會假設資料表是模型名稱的複數型態。所以，在這個範例中，`Task` 模型會假設要對應至 `tasks` 資料表。我們的空模型看起來應該如下：

	<?php

	namespace App;

	use Illuminate\Database\Eloquent\Model;

	class Task extends Model
	{
		//
	}

在為我們的應用程式增加路由時，我們會學習到更多如何使用 Eloquent 模型的相關內容。當然，你可以自由參考[完整的 Eloquent 文件](/laravel_tw/docs/5.2/eloquent)來取得更多資訊。

<a name="routing"></a>
## 路由

<a name="stubbing-the-routes"></a>
### 建置路由

接著，我們已經準備好開始增加一些路由至應用程式中。路由被用來將 URLs 指向控制器或是匿名函式，當使用者進入特定頁面時就會被執行。預設中，Laravel 所有的路由被定義在 `app/Http/routes.php` 檔案，每個新的專案都會包含此檔案。

對於本應用程式，我們知道我們最後會需要三個路由：一個用於顯示我們所有任務的清單、一個用於新增任務、一個用於刪除已有的任務。我們會將這些路由包在 `web` 中介層內，這樣它們會擁有 session 狀態及 CSRF 保護。所以，讓我們在 `app/Http/routes.php` 中建置這所有的路由：

	<?php

	use App\Task;
	use Illuminate\Http\Request;

	Route::group(['middleware' => 'web'], function () {

		/**
		 * 顯示所有任務
		 */
		Route::get('/', function () {
			//
		});

		/**
		 * 增加新的任務
		 */
		Route::post('/task', function (Request $request) {
			//
		});

		/**
		 * 刪除任務
		 */
		Route::delete('/task/{task}', function (Task $task) {
			//
		});
	});


<a name="displaying-a-view"></a>
### 顯示視圖

接著，讓我們來填寫 `/` 路由。在此路由中，我們想要渲染一個包含新增任務的表單，以及目前所有任務清單的 HTML 樣板。

在 Laravel 裡，所有的 HTML 樣板都儲存在 `resources/views` 目錄，且我們可以在路由中使用 `view` 輔助方法來回傳這些樣板的其中一個：

	Route::get('/', function () {
		return view('tasks');
	});

傳遞 `tasks` 至 `view` 函式會建立一個視圖物件實例，它會對應至 `resources/views/tasks.blade.php` 模板。當然，我們必須確實的定義這些視圖，所以現在讓我們開始動手做！

<a name="building-layouts-and-views"></a>
## 建構佈局與視圖

這個應用程式只會有一張視圖，包含新增任務的表單，以及目前所有任務的清單。為了幫助你想像此視圖，下方是完成後應用程式的截圖，套用了基本的 Bootstrap CSS 樣式：

![應用程式圖片](https://laravel.tw/assets/img/quickstart/basic-overview.png)

<a name="defining-the-layout"></a>
### 定義佈局

幾乎所有的網頁應用程式都會在不同頁面共用相同的佈局。舉個例子，應用程式通常在每個頁面（如果我們有一個以上）的頂部都擁有導航欄。Laravel 使用了 Blade **佈局**讓不同頁面共享這些相同的功能。

如同我們前面討論的，Laravel 所有的視圖都被儲存在 `resources/views`。所以，讓我們定義一個新的佈局視圖至 `resources/views/layouts/app.blade.php`。`.blade.php` 副檔名會告知框架使用 [Blade 模板引擎](/laravel_tw/docs/5.2/blade)渲染此視圖。當然，你可以在 Laravel 使用純 PHP 的樣板。不過，Blade 提供了方便的簡寫來撰寫乾淨、簡潔的模板。

我們的 `app.blade.php` 視圖看起來應該如下：

    // resources/views/layouts/app.blade.php

	<!DOCTYPE html>
	<html lang="en">
		<head>
			<title>Laravel 快速入門 - 基本</title>

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

注意佈局中的 `@yield('content')` 部分。這是特別的 Blade 指令，讓子頁面可以在此處注入自己的內容來延伸佈局。接著，讓我們定義將會使用此佈局並提供主要內容的子視圖。

<a name="defining-the-child-view"></a>
### 定義子視圖

接下來，我們需要定義包含建立任務的表單及列出已有任務表格的視圖。讓我們將此視圖定義在 `resources/views/tasks.blade.php`。

我們會跳過一些 Bootstrap CSS 樣板，只專注在重要的事物上。切記，你可以在 [GitHub](https://github.com/laravel/quickstart-basic) 下載應用程式的完整原始碼：

    // resources/views/tasks.blade.php

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
					<label for="task" class="col-sm-3 control-label">Task</label>

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

`@include('common.errors')` 指令會載入位於 `resources/views/common/errors.blade.php` 的模板。我們尚未定義此模板，但是我們將會在稍後定義它！

現在我們已經為我們的應用程式定義了基本的佈局及視圖。請記得，我們在 `/` 路由中回傳了此視圖，像這樣：

	Route::get('/', function () {
		return view('tasks');
	});

接著，我們已經準備好增加程式碼至我們的 `POST /task` 路由，以處理接收到的表單輸入並增加新的任務至資料庫中。

<a name="adding-tasks"></a>
## 增加任務

<a name="validation"></a>
### 驗證

現在我們視圖中已經有一個表單，我們需要增加程式碼至我們的 `POST /task` 路由來驗證接收到的表單輸入並建立新的任務。首先，讓我們驗證輸入。

對此表單來說，我們要讓 `name` 欄位為必填，且它必須少於 `255` 字元。如果驗證失敗，我們會將使用者重導回 `/` URL，並將舊的輸入及錯誤訊息快閃至 [session](/laravel_tw/docs/5.2/session)。快閃該輸入至 session 能讓我們保留使用者的輸入，即使有驗證錯誤：

	Route::post('/task', function (Request $request) {
		$validator = Validator::make($request->all(), [
			'name' => 'required|max:255',
		]);

		if ($validator->fails()) {
			return redirect('/')
				->withInput()
				->withErrors($validator);
		}

		// 建立該任務...
	});

#### `$errors` 變數

讓我們休息一下說說範例中 `->withErrors($validator)` 的部分。`->withErrors($validator)` 的呼叫會透過給定的驗證器實例將錯誤訊息快閃至 session 中，所以我們可以在視圖中透過 `$errors` 變數存取它們。

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

現在輸入已經被驗證處理完畢。讓我們繼續填寫我們的路由來實際的建立一筆新的任務。一旦新的任務被建立後，我們會將使用者重導回 `/` URL。要建立該任務，我們可以在為新的 Eloquent 模型建立及設定屬性後使用 `save` 方法：

	Route::post('/task', function (Request $request) {
		$validator = Validator::make($request->all(), [
			'name' => 'required|max:255',
		]);

		if ($validator->fails()) {
			return redirect('/')
				->withInput()
				->withErrors($validator);
		}

		$task = new Task;
		$task->name = $request->name;
		$task->save();

		return redirect('/');
	});

好極了！我們現在可以成功的建立任務。接著，讓我們繼續建構已有的任務清單，並增加至我們的視圖中。

<a name="displaying-existing-tasks"></a>
### 顯示已有的任務

首先，我們需要編輯我們的 `/` 路由，以傳遞所有已有的任務至視圖。`view` 函式接收一個能在視圖中取用之資料的陣列作為第二個參數，陣列中的每個鍵都會在視圖中作為變數：

	Route::get('/', function () {
		$tasks = Task::orderBy('created_at', 'asc')->get();

		return view('tasks', [
			'tasks' => $tasks
		]);
	});

一旦資料被傳遞之後，我們在我們的 `tasks.blade.php` 視圖中將任務切分並將它們顯示至表格中。`@foreach` 指令結構讓我們可以撰寫簡潔的迴圈，並編譯成快速的純 PHP 程式碼：

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

我們在我們的程式碼中應該放刪除按鈕的地方留下了「待辦」的事項。所以，讓我們在 `tasks.blade.php` 視圖中列出任務的每一行增加一個刪除按鈕。我們會為列表中的每個任務建立一個只有單個按鈕的小表單。當按鈕被按下時，一個 `DELETE /task` 的請求將會被發送到應用程式：

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

<a name="deleting-the-task"></a>
### 刪除該任務

最後，讓我們增加實際刪除給定任務的邏輯。我們可以使用[隱式模型綁定](/laravel_tw/docs/5.2/routing#route-model-binding)來自動取得對應 `{task}` 路由參數的 `Task` 模型。

在我們的路由回呼中，我們將使用 `delete` 方法來刪除該筆記錄。只要該記錄被刪除，我們會將使用者重導回 `/` URL：

	Route::delete('/task/{task}', function (Task $task) {
		$task->delete();

		return redirect('/');
	});
