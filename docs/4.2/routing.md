---
layout: post
title: routing
tag: 4.2
---
# 路由

- [基本路由](#basic-routing)
- [路由參數](#route-parameters)
- [路由篩選](#route-filters)
- [指名路由](#named-routes)
- [路由群組](#route-groups)
- [子網域路由](#sub-domain-routing)
- [前綴路由](#route-prefixing)
- [路由模型綁定](#route-model-binding)
- [404 錯誤](#throwing-404-errors)
- [控制器路由](#routing-to-controllers)

<a name="basic-routing"></a>
## 基本路由

應用程式大多數的路由都會被定義在 `app/routes.php` 中。最簡單的一個路由是由一個 URI 和閉包回調(Closure callback)。

#### 基本 GET 路由

	Route::get('/', function()
	{
		return 'Hello World';
	});

#### 基本 POST 路由

	Route::post('foo/bar', function()
	{
		return 'Hello World';
	});

#### 在一個路由中註冊多個動作

	Route::match(array('GET', 'POST'), '/', function()
	{
		return 'Hello World';
	});

#### 在一個路由中回應所有 HTTP 動作

	Route::any('foo', function()
	{
		return 'Hello World';
	});

#### 強制路由走 HTTPS

	Route::get('foo', array('https', function()
	{
		return 'Must be over HTTPS';
	}));

通常情況下，你需要產生 URLs 到你的路由上，你可以使用 `URL::to` 方法來達成：

	$url = URL::to('foo');

<a name="route-parameters"></a>
## 路由參數

	Route::get('user/{id}', function($id)
	{
		return 'User '.$id;
	});

#### 選用路由參數

	Route::get('user/{name?}', function($name = null)
	{
		return $name;
	});

#### 帶預設值的選用路由參數

	Route::get('user/{name?}', function($name = 'John')
	{
		return $name;
	});

#### 正規表示式路由

	Route::get('user/{name}', function($name)
	{
		//
	})
	->where('name', '[A-Za-z]+');

	Route::get('user/{id}', function($id)
	{
		//
	})
	->where('id', '[0-9]+');

#### 傳遞陣列使用 Where 篩選

當然，如果需要你可以傳遞限制條件的陣列：

	Route::get('user/{id}/{name}', function($id, $name)
	{
		//
	})
	->where(array('id' => '[0-9]+', 'name' => '[a-z]+'))

#### 定義全域樣式

如果你有常用的限制正規標示式樣式，你可以使用 `pattern` 方法：

	Route::pattern('id', '[0-9]+');

	Route::get('user/{id}', function($id)
	{
		// Only called if {id} is numeric.
	});

#### 存取路由參數值

如果你要在路由之外存取路由參數值，你可以使用 `Route::input` 方法：

	Route::filter('foo', function()
	{
		if (Route::input('id') == 1)
		{
			//
		}
	});

<a name="route-filters"></a>
## 路由篩選器

路由篩選器提供一個便捷的方式對於一個給定的路由做出限制訪問，這對於你的站台需要認證的情況下非常有用。在 Laravel 框架中包含了數個篩選器，像是 `auth`、`auth.basic`、`guest` 和 `csrf` 篩選器。他們都放在 `app/filters.php` 中。

#### 定義一個路由篩選器

	Route::filter('old', function()
	{
		if (Input::get('age') < 200)
		{
			return Redirect::to('home');
		}
	});

如果篩選器傳回了回應，這個會應將會直接被視為該請求的回應，且路由將不會繼續被執行，任何路由的 `after` 篩選器將直接被取消。

#### 對路由加上篩選器

	Route::get('user', array('before' => 'old', function()
	{
		return 'You are over 200 years old!';
	}));

#### 對控制器動作加上篩選器

	Route::get('user', array('before' => 'old', 'uses' => 'UserController@showProfile'));

#### 對單一路由加上多個篩選器

	Route::get('user', array('before' => 'auth|old', function()
	{
		return 'You are authenticated and over 200 years old!';
	}));

#### 透過陣列加上多個篩選器

	Route::get('user', array('before' => array('auth', 'old'), function()
	{
		return 'You are authenticated and over 200 years old!';
	}));

#### 指定篩選器參數

	Route::filter('age', function($route, $request, $value)
	{
		//
	});

	Route::get('user', array('before' => 'age:200', function()
	{
		return 'Hello World';
	}));

在篩選器接收到一個 `$response` 會被當成第三個參數傳遞進篩選器：

	Route::filter('log', function($route, $request, $response)
	{
		//
	});

#### 篩選器樣式

你可以依據路由符合的 URI 來指定其篩選器：

	Route::filter('admin', function()
	{
		//
	});

	Route::when('admin/*', 'admin');

在上面的範例中，`admin` 篩選器將會套用在所有以 `admin/` 開頭的路由中。星號通常用作通配符，他會匹配任何的字元組合。

你一樣可以篩選指定的 HTTP 動作：

	Route::when('admin/*', 'admin', array('post'));

#### 篩選器類別

進階的篩選，你可以使用類別來取代閉包。因為篩選器類別被解析出應用程式 [IoC 容器](/docs/ioc)，你將會可以在這些篩選器中利用依賴注入來得到更好的可測試性。

#### 註冊基於類別的篩選器

	Route::filter('foo', 'FooFilter');

預設下，`FooFilter` 類別的 `filter` 方法將會被呼叫：

	class FooFilter {

		public function filter()
		{
			// Filter logic...
		}

	}

如果你不希望使用 `filter` 方法，只要指定其他方法即可：

	Route::filter('foo', 'FooFilter@foo');

<a name="named-routes"></a>
## 指名路由

指名路由在產生重導與 URLs 至路由時更為方便。你可以指定一個名稱給指定的路由：

	Route::get('user/profile', array('as' => 'profile', function()
	{
		//
	}));

你一樣可以為控制器動作指定一個路由名稱：

	Route::get('user/profile', array('as' => 'profile', 'uses' => 'UserController@showProfile'));

現在你可以在產生 URLs 或重導時使用該路由名稱：

	$url = URL::route('profile');

	$redirect = Redirect::route('profile');

你一樣可以透過 `currentRouteName` 方法來取得正在執行中的路由名稱：

	$name = Route::currentRouteName();

<a name="route-groups"></a>
## 路由群組

有時候你需要套用篩選器到一個群組的路由上。不需要為每個路由去套用篩選器，你只需使用路由群組：

	Route::group(array('before' => 'auth'), function()
	{
		Route::get('/', function()
		{
			// Has Auth Filter
		});

		Route::get('user/profile', function()
		{
			// Has Auth Filter
		});
	});

你一樣可以在 `group` 陣列中使用 `namespace` 參數，指定在這 group 中的控制器都有一個共同的命名空間：

	Route::group(array('namespace' => 'Admin'), function()
	{
		//
	});

<a name="sub-domain-routing"></a>
## 子網域路由

Laravel 路由一樣可以處理通配的子網域，並且從網域中傳遞你的通配符參數：

#### 註冊子網域路由

	Route::group(array('domain' => '{account}.myapp.com'), function()
	{

		Route::get('user/{id}', function($account, $id)
		{
			//
		});

	});

<a name="route-prefixing"></a>
## 前綴路由

群組路由可以透過群組的描述陣列中使用 `prefix` 選項，將群組內的路由加上前綴：

	Route::group(array('prefix' => 'admin'), function()
	{

		Route::get('user', function()
		{
			//
		});

	});

<a name="route-model-binding"></a>
## 路由模型綁定

模型綁定提供一個方便的方式將模型實體注入到你的路由中。例如，要注入一個使用者 ID 你可以注入符合給定 ID 的整個使用者模型實體。首先，使用 `Route::model` 方法可以指定作為參數的模型：

#### 綁定參數至模型

	Route::model('user', 'User');

接著，定義一個路由並包括一個 `{user}` 參數：

	Route::get('profile/{user}', function(User $user)
	{
		//
	});

既然我們綁定了 `{user}` 參數到 `User` 模型，則 `User` 實體就會被注入到路由內。因此，假定有一個請求送至 `profile/1` 則會注入一個 ID 為 1 的 `User` 實體。

> **注意：** 假如在資料庫內沒有任何一個模型實體符合，則會拋出 404 錯誤。

假如您希望指定您自定的「找不到」錯誤行為，你可以在 `model` 方法裡的第三個參數指定一個 Closure：

	Route::model('user', 'User', function()
	{
		throw new NotFoundHttpException;
	});

在某些情況下，您可能會希望可以使用您自定的路由綁定方式。這時您可以使用 `Route::bind` 方法來達成：

	Route::bind('user', function($value, $route)
	{
		return User::where('name', $value)->first();
	});

<a name="throwing-404-errors"></a>
## 404 錯誤

有兩種方式可以在路由內手動觸發 404 錯誤。第一種是呼叫 `App::abort` 方法：

	App::abort(404);

第二種，你可以拋出一個 `Symfony\Component\HttpKernel\Exception\NotFoundHttpException` 實體。

有關如何處理 404 例外狀況和自定回應的詳細資訊，可以參考 [錯誤](/docs/errors#handling-404-errors) 章節內的說明。

<a name="routing-to-controllers"></a>
## 控制器路由

Laravel 允許您不止可以路由至 Closures，也可以路由至控制器類別，甚至可以路由至 [資源控制器](/docs/controllers#resource-controllers)。

更多詳細資訊可參考 [控制器](/docs/controllers) 一節內的說明。
