# 重設密碼

- [介紹](#introduction)
- [資料庫的注意事項](#resetting-database)
- [路由](#resetting-routing)
- [視圖](#resetting-views)
- [重設密碼之後](#after-resetting-passwords)
- [自訂](#password-customization)

<a name="introduction"></a>
## 介紹

> {tip} **想要快速入門？**只要在剛建立的 Laravel 應用程式中執行 `php artisan make:auth`，並瀏覽 `http://your-app.dev/register`  這個頁面或分配給應用程式的任何其他的連結。這一個指令會幫你建立好完整的認證系統，而且包含重設密碼！

大多數網頁應用程式提供使用者重設它們遺忘的密碼。Laravel 並沒有強迫你要在每個應用程式上實作這個功能，而是提供方便的方法來傳送密碼提示和處理密碼重設。

> {note} 在使用 Laravel 的密碼重設功能之前，你的 user 模型最好使用 `Illuminate\Notifications\Notifiable` trait。

<a name="resetting-database"></a>
## 資料庫的注意事項

在開始之前，驗證你的 App\User 模型是否實作 Illuminate\Contracts\Auth\CanResetPassword contract。當然，`App\User` 模型早就被框架實作這個介面，並使用 `Illuminate\Auth\Passwords\CanResetPassword` trait 來引入實作該介面所需的方法。

#### 產生重設密碼 Token 資料表的遷移

接著，你必須建立一個用來儲存密碼重設的 Token 的資料表。該資料表的遷移檔已經包含在 Laravel 中，並放置在 `database/migrations` 目錄。所以，你所要做的是執行你的資料庫遷移：

    php artisan migrate

<a name="resetting-routing"></a>
## 路由

Laravel 引入了 `Auth\ForgotPasswordController` 和 `Auth\ResetPasswordController` 類別，其中包含電子信箱的密碼重置連結和重設使用者密碼所需要的邏輯。所有處理密碼重設所需要的路由都可以使用 Artisan 的 `make:auth` 指令來產生：

    php artisan make:auth

<a name="resetting-views"></a>
## 視圖

Laravel 會在執行 `make:auth` 時產生所有密碼重設所需要的視圖。這些視圖被放置在 `resources/views/auth/passwords`。你可以根據應用程式的需要來自由的自訂它們。

<a name="after-resetting-passwords"></a>
## 重設密碼之後

你一旦定義了路由和視圖來重設使用者密碼，你可以在瀏覽器的 `/password/reset` 來簡單的存取你的路由。 框架引入的 `ForgotPasswordController` 已包含了發送密碼重設連結的 e-mail 邏輯，而 `ResetPasswordController` 則包含了重設使用者密碼的邏輯。

在密碼重設之後，該使用者會自動登入應用程式並重導到 `/home`。你能在 `ResetPasswordController` 上定義 `redirectTo` 屬性來自訂密碼重設後要重導的位置：

    protected $redirectTo = '/dashboard';

> {note} 預設的密碼重設 Token 會在一小時後失效。你可以透過 `config/auth.php` 檔案中的密碼重設的 `expire` 選項來改變有效期限。

<a name="password-customization"></a>
## 自訂

#### 自訂認證 Guard

在 `auth.php` 設定檔中，你可以設定多個「Guard」，可被用於定義多個使用者資料表的認證行為。你能在控制器上覆寫 `guard` 方法來自訂 `ResetPasswordController` 來使用你選擇的選擇 Guard。這個方法會回傳一個 Guard 實例：

    use Illuminate\Support\Facades\Auth;

    protected function guard()
    {
        return Auth::guard('guard-name');
    }

#### 自訂密碼 Broker

在你的 `auth.php` 設定檔中，你可以設定多個密碼「broker」，可被用於重置多個使用者資料表。你能覆寫 `broker` 方法來自訂 `ForgotPasswordController` 和 `ResetPasswordController` 來使用你所選擇的 Broker：

    use Illuminate\Support\Facades\Password;

    /**
     * 取得密碼重設期間的 broker。
     *
     * @return PasswordBroker
     */
    protected function broker()
    {
        return Password::broker('name');
    }

#### 自訂重設信件

你可以輕易地修改通知類別，並用在發送密碼重設連結給使用者。在開始之前，請在你的 `User` 模型上覆寫 `sendPasswordResetNotification` 方法。在這個方法中，你可以使用任何你選擇的通知類別來發送通知。該密碼重設的 `$token` 會是這個方法接收到的第一個參數：

    /**
     * 發送密碼重設通知。
     *
     * @param  string  $token
     * @return void
     */
    public function sendPasswordResetNotification($token)
    {
        $this->notify(new ResetPasswordNotification($token));
    }
