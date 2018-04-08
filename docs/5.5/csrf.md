---
layout: post
title: csrf
tag: 5.5
---
# CSRF 保護

- [介紹](#csrf-introduction)
- [排除 URI](#csrf-excluding-uris)
- [X-CSRF-Token](#csrf-x-csrf-token)
- [X-XSRF-Token](#csrf-x-xsrf-token)

<a name="csrf-introduction"></a>
## 介紹

Laravel 可以輕易地保護你的應用程式免受到[跨網站偽造請求攻擊](https://en.wikipedia.org/wiki/Cross-site_request_forgery)（CSRF）。跨網站偽造請求是一種惡意的攻擊，會偽造已認證的使用者執行未授權的指令。

Laravel 透過應用程式自動產生一個 CSRF「token」來管理每個活躍的使用者 session。這個 token 用於驗證已認證使用者是否實際向應用程式發出請求。

在你每次定義 HTML 表單的時候，應該在表單中插入隱藏的 CSRF token，這樣用於預防 CSRF 攻擊的中介層可以驗證表單請求。你可以使用 `csrf_field` 輔助函式來產生 token 到表單中：

    <form method="POST" action="/profile">
        {% raw %} {{ csrf_field() }} {% endraw %}
        ...
    </form>

包含在 `web` 中介層群組的 `VerifyCsrfToken` [中介層](/laravel_tw/docs/5.5/middleware)，會自動驗證表單請求的 token 是否與儲存在 session 的 token 一致。

#### CSRF Tokens 與 JavaScript

建構 JavaScript 應用程式的時候，可以用很便利的方式讓你的 JavaScript HTTP 函式庫也能自動附加 CSRF token 到每個對外的請求。預設上，`resources/assets/js/bootstrap.js` 檔案註冊了已寫入 `csrf-token` 屬性標籤值的 Axios HTTP 函式庫。如果你不要使用這個函式庫，則需要為應用程式手動設定此行為。

<a name="csrf-excluding-uris"></a>
## 從 CSRF 保護中排除 URI

有些時候，你可能希望從 CSRF 保護範圍中忽略一組 URI。舉例來說，如果你正使用 [Stripe](https://stripe.com) 來處理付款且還使用他們的 webhook 系統，你會需要從 CSRF 保護範圍中忽略 Stripe 的處理路由，因為 Stripe 不會知道要發 CSRF token 給你的路由。

通常來說，你應該將這類型的路由放置於 `web` 中介層群組之外，是因為 `RouteServiceProvider` 會使用 `web` 中介層群組並應用在 `routes/web.php` 裡的全部路由。不過，你也可以在 `VerifyCsrfToken` 中介層中新增他們的 URI 到 `$except` 屬性來設置白名單：

    <?php

    namespace App\Http\Middleware;

    use Illuminate\Foundation\Http\Middleware\VerifyCsrfToken as Middleware;

    class VerifyCsrfToken extends Middleware
    {
        /**
         * URI 將排除於 CSRF 驗證流程
         *
         * @var array
         */
        protected $except = [
            'stripe/*',
        ];
    }

<a name="csrf-x-csrf-token"></a>
## X-CSRF-TOKEN

除了把 CSRF token 作為 POST 參數來檢查外，`VerifyCsrfToken` 中介層也會檢查 `X-CSRF-TOKEN` 請求標頭。如範例所見，你可以將 token 存放在 HTML 的 `meta` 標籤裡：

    <meta name="csrf-token" content="{% raw %} {{ csrf_token() }} {% endraw %}">

然後，一旦你建好 `meta` 標籤，就能指示類似 jQuery 的函式庫自動新增 token 到每個請求標頭。這能為你的 AJAX 應用程式提供既簡單又便利的 CSRF 保護：

    $.ajaxSetup({
        headers: {
            'X-CSRF-TOKEN': $('meta[name="csrf-token"]').attr('content')
        }
    });

> {tip} 預設上，`resources/assets/js/bootstrap.js` 檔案會使用 Axios HTTP 函式庫來註冊 `csrf-token` meta 標籤的值。如果你想使用這個函式庫，則需要為應用程式手動設定此行為。

<a name="csrf-x-xsrf-token"></a>
## X-XSRF-TOKEN

Laravel 會在每個由框架產生的回應中包含一組 `XSRF-TOKEN` cookie，並將當前的 CSRF token 儲存於此。你能使用 cookie 的值去設定 `X-XSRF-TOKEN` 的請求標頭。

之所以這個 cookie 能方便地被發送，主要是因為一些 JavaScript 框架和函式庫（像是 Angular 和 Axios）會自動將其值放入 `X-XSRF-TOKEN` 標頭。
