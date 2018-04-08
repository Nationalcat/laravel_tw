---
layout: post
title: upgrade
tag: 4.1
---
# 升級導引

- [Upgrading To 4.1.29 From <= 4.1.x](#upgrade-4.1.29)
- [Upgrading To 4.1.26 From <= 4.1.25](#upgrade-4.1.26)
- [從 4.0 升級至 4.1](#upgrade-4.1)

<a name="upgrade-4.1.29"></a>
## Upgrading To 4.1.29 From <= 4.1.x

Laravel 4.1.29 improves the column quoting for all database drivers. This protects your application from some mass assignment vulnerabilities when **not** using the `fillable` property on models. If you are using the `fillable` property on your models to protect against mass assignment, your application is not vulnerable. However, if you are using `guarded` and are passing a user controlled array into an "update" or "save" type function, you should upgrade to `4.1.29` immediately as your application may be at risk of mass assignment.

To upgrade to Laravel 4.1.29, simply `composer update`. No breaking changes are introduced in this release.

<a name="upgrade-4.1.26"></a>
## Upgrading To 4.1.26 From <= 4.1.25

Laravel 4.1.26 introduces security improvements for "remember me" cookies. Before this update, if a remember cookie was hijacked by another malicious user, the cookie would remain valid for a long period of time, even after the true owner of the account reset their password, logged out, etc.

This change requires the addition of a new `remember_token` column to your `users` (or equivalent) database table. After this change, a fresh token will be assigned to the user each time they login to your application. The token will also be refreshed when the user logs out of the application. The implications of this change are: if a "remember me" cookie is hijacked, simply logging out of the application will invalidate the cookie.

### Upgrade Path

First, add a new, nullable `remember_token` of VARCHAR(100), TEXT, or equivalent to your `users` table.

Next, if you are using the Eloquent authentication driver, update your `User` class with the following three methods:

	public function getRememberToken()
	{
		return $this->remember_token;
	}

	public function setRememberToken($value)
	{
		$this->remember_token = $value;
	}

	public function getRememberTokenName()
	{
		return 'remember_token';
	}

> **Note:** All existing "remember me" sessions will be invalidated by this change, so all users will be forced to re-authenticate with your application.

### Package Maintainers

Two new methods were added to the `Illuminate\Auth\UserProviderInterface` interface. Sample implementations may be found in the default drivers:

	public function retrieveByToken($identifier, $token);

	public function updateRememberToken(UserInterface $user, $token);

The `Illuminate\Auth\UserInterface` also received the three new methods described in the "Upgrade Path".

<a name="upgrade-4.1"></a>
## 從 4.0 升級至 4.1

### 升級你的 Composer 相依性

升級你的應用程式至 Laravel 4.1，將 `composer.json` 裡的 `laravel/framework` 版本更改至 `4.1.*`。

### 檔案置換

將你的 `public/index.php` 置換成 [這個 repository 的乾淨版本](https://github.com/laravel/laravel/blob/master/public/index.php)。

同樣的，將你的 `artisan` 置換成 [這個 repository 的乾淨版本](https://github.com/laravel/laravel/blob/master/artisan)。

### 新增設定檔案及選項

更新你在設定檔 `app/config/app.php` 裡的 `aliases` 和 `providers` 陣列。而更新的選項值可以在[這個檔案](https://github.com/laravel/laravel/blob/master/app/config/app.php)中找到。請確定將你後來加入自定和套件所需的 providers / aliases 加回陣列中。

從 [這個 repository](https://github.com/laravel/laravel/blob/master/app/config/remote.php)增加 `app/config/remote.php` 檔案。

在你的 `app/config/session.php` 增加新的選項 `expire_on_close`。而預設值為 `false`。

在你的 `app/config/queue.php` 檔案裡新增 `failed` 設定區塊。以下為區塊的預設值：

	'failed' => array(
		'database' => 'mysql', 'table' => 'failed_jobs',
	),

**（非必要）** 在你的 `app/config/view.php` 裡，將 `pagination` 設定選項更新為 `pagination::slider-3`。

### 更新控制器（Controllers）

如果 `app/controllers/BaseController.php` 有 `use` 語句在最上面，將 `use Illuminate\Routing\Controllers\Controller;` 改為 `use Illuminate\Routing\Controller;`。

### 更新密碼提醒

密碼提醒功能已經大幅修正擁有更大的彈性。你可以執行 Artisan 指令 `php artisan auth:reminders-controller` 來檢查新的存根控制器。你也可以瀏覽 [更新文檔](/docs/security#password-reminders-and-reset) 然後相應的更新你的應用程式。

更新你的 `app/lang/en/reminders.php` 語系檔案來符合[這個新版檔案](https://github.com/laravel/laravel/blob/master/app/lang/en/reminders.php)。

### 更新環境偵測

為了安全因素，不再使用網域網址來偵測辨別應用程式的環境。因為這些直很容易被偽造欺騙，繼而讓攻擊者透過請求來達到變更環境。所以你必須改為使用機器的 hostname（在 Mac & Ubuntu 下執行 `hostname` 出來的值）

（譯按：的確原有方式有安全性考量，但對於現行 VirtualHost 大量使用下，反而這樣改會造成不便）

### 更簡單的日誌文件

Laravel 目前只會產生單一的日誌文件: `app/storage/logs/laravel.log`。然而，你還是可以透過設定你的 `app/start/global.php` 檔案來更改他的行為。

### 刪除重定向結尾的斜線

在你的 `bootstrap/start.php` 檔案中，移除呼叫 `$app->redirectIfTrailingSlash()`。這個方法已不再需要了，因為之後將會改以框架內的 `.htaccess` 來處理。

然後，用[新版](https://github.com/laravel/laravel/blob/master/public/.htaccess)替換掉你 Apache 中的 `.htaccess` 檔案，來處理結尾的斜線問題。

### 取得目前路由

取得目前路由的方法由 `Route::getCurrentRoute()` 改為 `Route::current()`。

### Composer 更新

一旦你完成以上的更新，你可以執行 `composer update` 來更新應用程式的核心檔案。如果有 class load 錯誤，請在 `update` 之後加上 `--no-scripts`，如: `composer update --no-scripts`。

### Wildcard Event Listeners

The wildcard event listeners no longer append the event to your handler functions parameters. If you require finding the event that was fired you should use `Event::firing()`.
