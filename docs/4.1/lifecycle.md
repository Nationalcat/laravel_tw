---
layout: post
title: lifecycle
tag: 4.1
---
# Request 生命週期

- [概覽](#overview)
- [Request 生命週期](#request-lifecycle)
- [啟動文件](#start-files)
- [應用程式事件](#application-events)

<a name="overview"></a>
## 概覽

在“真實世界”裡使用任何工具，只要你了解該工具的運作原理，你將會很順手。應用程式開發上也是相同的。當你了解開發工具的功能，你使用上會更得心應手。而這份文件的目的就是要給予你一個優良、高層次的Laravel 框架運作概覽。隨著了解整個框架越多，你就不會覺得很"奇妙"且更容易去建構你的應用程式。除了關於 request 生命週期的高層次概覽外，我們也會介紹”啟動“檔案和應用程式事件。

如果你對許多東西不明白，不用擔心。只要了解個基本的原理，隨著探索其他文件，你將會得到更多的知識。 

<a name="request-lifecycle"></a>
## Request 生命週期

在你的應用程式中，所有的 Request 都會被指向到 `public/index.php` 上。如果是使用 Apache，`.htsccess` 檔案會將所有的 requests 都指向 `index.php` 來去處理。從這開始，Laravel 開始處理 requests 且返回回應給客戶端。了解 Laravel 的引導過程的總體思路是有益的。

到目前為止，在學習 laravel 的引導過程的最重要概念是 **服務提供商（Service Providers)）**。你可在 `app/config/app.php` 設定檔裡找到 `providers` 陣列，裡面條列了服務提供商的列表。這些服務提供商是 Laravel 的主要引導機制。但在我們深入這些服務提供商前，讓我們先回到 `index.php`。在 request 進入 `index.php` 後，`bootstrap/start.php` 將接著被引入。這個檔案將會建立一個新的 Laravel `Application` 物件，也作為一個 [IoC 容器](/docs/ioc)。

在 `Application` 物件建立之後，一些物件路徑將被設定，而且進行 [環境偵測](/docs/configuration#environment-configuration)。然後，一個內部的 Laravel 引導腳本將會被呼叫。這個檔案深藏在 Laravel 的原始碼中，而且基於你的設定檔，設定了一些選項，例如：時區（timezone）、錯誤回報（error reporting）等等。但是，除了設定這些瑣碎的選項外，它還做了一件非常重要的事情：註冊所有設定在你應用程式中的服務提供商。

Simple service providers only have one method: `register`. This `register` method is called when the service provider is registered with the application object via the application's own `register` method. Within this method, service providers register things with the [IoC container](/docs/ioc). Essentially, each service provider binds one or more [closures](http://us3.php.net/manual/en/functions.anonymous.php) into the container, which allows you to access those bound services within your application. So, for example, the `QueueServiceProvider` registers closures that resolve the various [Queue](/docs/queues) related classes. Of course, service providers may be used for any bootstrapping task, not just registering things with the IoC container. A service provider may register event listeners, view composers, Artisan commands, and more.

After all of the service providers have been registered, your `app/start` files will be loaded. Lastly, your `app/routes.php` file will be loaded. Once your `routes.php` file has been loaded, the Request object is sent to the application so that it may be dispatched to a route.

So, let's summarize:

1. Request enters `public/index.php` file.
2. `bootstrap/start.php` file creates Application and detects environment.
3. Internal `framework/start.php` file configures settings and loads service providers.
4. Application `app/start` files are loaded.
5. Application `app/routes.php` file is loaded.
6. Request object sent to Application, which returns Response object.
7. Response object sent back to client.

Now that you have a good idea of how a request to a Laravel application is handled, let's take a closer look at "start" files!

<a name="start-files"></a>
## Start Files

Your application's start files are stored at `app/start`. By default, three are included with your application: `global.php`, `local.php`, and `artisan.php`. For more information about `artisan.php`, refer to the documentation on the [Artisan command line](/docs/commands#registering-commands).

The `global.php` start file contains a few basic items by default, such as the registration of the [Logger](/docs/errors) and the inclusion of your `app/filters.php` file. However, you are free to add anything to this file that you wish. It will be automatically included on _every_ request to your application, regardless of environment. The `local.php` file, on the other hand, is only called when the application is executing in the `local` environment. For more information on environments, check out the [configuration](/docs/configuration) documentation.

Of course, if you have other environments in addition to `local`, you may create start files for those environments as well. They will be automatically included when your application is running in that environment. So, for example, if you have a `development` environment configured in your `bootstrap/start.php` file, you may create a `app/start/development.php` file, which will be included when any requests enter the application in that environment.

### What To Place In Start Files

Start files serve as a simple place to place any "bootstrapping" code. For example, you could register a View composer, configure your logging preferences, set some PHP settings, etc. It's totally up to you. Of course, throwing all of your bootstrapping code into your start files can get messy. For large applications, or if you feel your start files are getting messy, consider moving some bootstrapping code into [service providers](/docs/ioc#service-providers).

<a name="application-events"></a>
## Application Events

#### Registering Application Events

You may also do pre and post request processing by registering `before`, `after`, `finish`, and `shutdown` application events:

	App::before(function($request)
	{
		//
	});

	App::after(function($request, $response)
	{
		//
	});

Listeners to these events will be run `before` and `after` each request to your application. These events can be helpful for global filtering or global modification of responses. You may register them in one of your `start` files or in a [service provider](/docs/ioc#service-providers).

You may also register a listener on the `matched` event, which is fired when an incoming request has been matched to a route but that route has not yet been executed:

	Route::matched(function($route, $request)
	{
		//
	});

The `finish` event is called after the response from your application has been sent back to the client. This is a good place to do any last minute processing your application requires. The `shutdown` event is called immediately after all of the `finish` event handlers finish processing, and is the last opportunity to do any work before the script terminates. Most likely, you will not have a need to use either of these events.