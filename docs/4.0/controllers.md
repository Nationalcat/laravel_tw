---
layout: post
title: controllers
tag: 4.0
---
# 控制器(Controllers)

- [基本控制器](#basic-controllers)
- [控制器過濾](#controller-filters)
- [RESTful 控制器](#restful-controllers)
- [資源控制器](#resource-controllers)
- [處理不存在的方法](#handling-missing-methods)

<a name="basic-controllers"></a>
## 基本控制器

除了在 `routes.php` 檔案中定義你所有路由層級的邏輯外，你可能希望使用控制器類別去整理這些處理邏輯，控制器可以整合相關的路由邏輯成一個類別，也可以有進階框架功能的優點，像是自動化的[依賴注入](/docs/ioc)。

Controllers are typically stored in the `app/controllers` directory, and this directory is registered in the `classmap` option of your `composer.json` file by default.

控制器基本上是存放在 `app/controllers` 目錄裡，這個目錄已經在預設在 `composer.json` 檔案中的 `classmap` 選項預設註冊載入。

這裡有一個基本控制器的範例:

	class UserController extends BaseController {

		/**
		 * Show the profile for the given user.
		 */
		public function showProfile($id)
		{
			$user = User::find($id);

			return View::make('user.profile', array('user' => $user));
		}

	}

所有的控制器必須繼承 `BaseController` 的類別，`BaseController` 也存放在 `app/controllers` 的目錄裡，也可以用來放一些共用的控制器邏輯，`BaseController` 繼承了 Laravel 框架的 `Controller` 類別，現在我們可以路由到這個控制的的方法，像這樣:

	Route::get('user/{id}', 'UserController@showProfile');

如果你選擇用巢狀或 PHP 命名空間 (namespaces) 的方式去組織你的控制器，在定義路由時，只要使用完整符合類別名稱的規則即可:

	Route::get('foo', 'Namespace\FooController@method');

你也可以命名控制器的路由:

	Route::get('foo', array('uses' => 'FooController@method',
											'as' => 'name'));

為了產生 URL 指到到控制器，你可以使用 `URL::action` 的方法:

	$url = URL::action('FooController@method');

你可以執行 `currentRouteAction` 方法去取得目前路由到控制器的方法:

	$action = Route::currentRouteAction();

<a name="controller-filters"></a>
## 控制器過濾

[過濾器](/docs/routing#route-filters) 可以指定用在控制器，就像常見的"路由"一樣。

	Route::get('profile', array('before' => 'auth',
				'uses' => 'UserController@showProfile'));

然而，你也可以在控制器裡面指定你的過濾器:

	class UserController extends BaseController {

		/**
		 * Instantiate a new UserController instance.
		 */
		public function __construct()
		{
			$this->beforeFilter('auth', array('except' => 'getLogin'));

			$this->beforeFilter('csrf', array('on' => 'post'));

			$this->afterFilter('log', array('only' =>
								array('fooAction', 'barAction')));
		}

	}

你也可以在控制器的過濾器裡使用閉合函式:

	class UserController extends BaseController {

		/**
		 * Instantiate a new UserController instance.
		 */
		public function __construct()
		{
			$this->beforeFilter(function()
			{
				//
			});
		}

	}

<a name="restful-controllers"></a>
## RESTful 控制器

Laravel 允許你使用簡單的 REST 命名規範，去定義一個簡單的路由去處理每一個在控制器裡的方法，首先使用 `Route::controller` 去定義路由:

**定義一個 RESTful 控制器**

	Route::controller('users', 'UserController');

`controller` 方法允許兩個參數，第一個是 URI 為基準的控制器處理方法，第二個是控制器類別名稱，接下來只要增加方法到你的控制器，前綴使用 HTTP 請求方法的動詞去對應即可:

	class UserController extends BaseController {

		public function getIndex()
		{
			//
		}

		public function postProfile()
		{
			//
		}

	}

`index` 方法將會對應到控制器處理 URI 的根目錄，在這個例子是 `users`。

如果你的控制器方法包含數個字詞，你可以在 URI 使用 "破折號 (dash)" 的語法去存取它，舉例來說，在下面 `UserController` 的動作將會對應到 `users/admin-profile` 的 URI:

	public function getAdminProfile() {}

<a name="resource-controllers"></a>
## 資源控制器

資源 (Resource) 的控制器可以很容易地去建置 RESTful 控制器，舉例來說，你也許希望在你的應用裡，建立一個控制器去管理你的"相片"，在 Artisan介面指令 使用 `controller:make` 的指令，和 `Route::resource `的方法，我們就可以快速地建立這些控制器。

為了透過命令列建立控制器，你可以執行下列指令:

	php artisan controller:make PhotoController

現在我們可以註冊一個資源化 (resourceful) 的路由到這個控制器:

	Route::resource('photo', 'PhotoController');

這個單一路由的定義，在 photo 建立了多個路由資源，去做不同的 RESTful 動作存取，同樣的，被產生的控制器將會有基本的方法，這些方法動作將會告知你哪個 URI 和 HTTP 請求動作是他們所處理的。

**資源控制器處理的動作**

Verb      | Path                        | Action       | Route Name
----------|-----------------------------|--------------|---------------------
GET       | /resource                   | index        | resource.index
GET       | /resource/create            | create       | resource.create
POST      | /resource                   | store        | resource.store
GET       | /resource/{resource}        | show         | resource.show
GET       | /resource/{resource}/edit   | edit         | resource.edit
PUT/PATCH | /resource/{resource}        | update       | resource.update
DELETE    | /resource/{resource}        | destroy      | resource.destroy

有時候，你也許只需要處理資源的某些方法:

	php artisan controller:make PhotoController --only=index,show

	php artisan controller:make PhotoController --except=index

接下來，也可以指名路由去處理部分的方法動作:

	Route::resource('photo', 'PhotoController',
					array('only' => array('index', 'show')));

	Route::resource('photo', 'PhotoController',
					array('except' => array('create', 'store', 'update', 'delete')));

<a name="handling-missing-methods"></a>
## 處理不存在的方法

可以定義一個擷取所有動作的方法，在沒有比對到控制器中的方法時，會呼叫此方法，這個方法是命名為 `missingMethod`，且接受了的參數陣列為此請求方法的唯一參數:

**定義一個接收所有 (Catch-All) 請求的方法**

	public function missingMethod($parameters)
	{
		//
	}
