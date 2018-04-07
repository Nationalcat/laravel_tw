---
layout: post
title: billing
---
# Laravel Cashier

- [簡介](#introduction)
- [訂購](#subscriptions)
    - [建立訂購](#creating-subscriptions)
    - [確認訂購狀態](#checking-subscription-status)
    - [改變方案](#changing-plans)
    - [訂購數量](#subscription-quantity)
    - [訂購稅金](#subscription-taxes)
    - [取消訂購](#cancelling-subscriptions)
    - [恢復訂購](#resuming-subscriptions)
- [處理 Stripe Webhooks](#handling-stripe-webhooks)
    - [訂購失敗](#handling-failed-subscriptions)
    - [其他 Webhooks](#handling-other-webhooks)
- [一次性收費](#single-charges)
- [收據](#invoices)
    - [產生收據的 PDFs](#generating-invoice-pdfs)

<a name="introduction"></a>
## 簡介

Laravel Cashier 提供口語化，流暢的介面與 [Stripe 的](https://stripe.com)訂購交易服務介接。它幾乎處理了所有讓人退步三舍的訂購管理相關邏輯。除了基本的訂購管理，Cashier 還可以處理折價券，訂購轉換，管理訂購「數量」、取消寬限期，甚至產生收據的 PDF。

<a name="configuration"></a>
### 設定

#### Composer

首先，將 Cashier 套件新增至 `composer.json` 檔案並執行 `composer update` 指令：

    "laravel/cashier": "~6.0"

#### 服務提供者

接著，在 `app` 設定檔中註冊 `Laravel\Cashier\CashierServiceProvider` [服務提供者](/laravel_tw/docs/5.2/providers)。

#### 遷移

我們需要準備資料庫，建立幾個欄位到 `users` 資料表以及建立一個新的 `subscriptions` 資料表來儲存所有客戶的訂購資料：

    Schema::table('users', function ($table) {
        $table->string('stripe_id')->nullable();
        $table->string('card_brand')->nullable();
        $table->string('card_last_four')->nullable();
    });

    Schema::create('subscriptions', function ($table) {
        $table->increments('id');
        $table->integer('user_id');
        $table->string('name');
        $table->string('stripe_id');
        $table->string('stripe_plan');
        $table->integer('quantity');
        $table->timestamp('trial_ends_at')->nullable();
        $table->timestamp('ends_at')->nullable();
        $table->timestamps();
    });

一旦遷移檔被建立之後，只要執行 `migrate` 命令就可以完成遷移。

#### 模型設定

再來，將 `Billable` trait 新增至你的模型定義中：

    use Laravel\Cashier\Billable;

    class User extends Authenticatable
    {
        use Billable;
    }

#### Stripe 金鑰

最後，在你的 `services.php` 設定檔中設定 Stripe 金鑰：

    'stripe' => [
        'model'  => App\User::class,
        'secret' => env('STRIPE_API_SECRET'),
    ],

<a name="subscriptions"></a>
## 訂購

<a name="creating-subscriptions"></a>
### 建立訂購

要建立一個訂購，首先要取得可交易的模型實例，這通常會是 `App\User` 的實例。一旦你取得了模型實例，你可以使用 `newSubscription` 方法來管理模型的訂購：

    $user = User::find(1);

    $user->newSubscription('main', 'monthly')->create($creditCardToken);

傳遞給 `newSubscription` 方法的第一個參數應該為訂單的名稱。如果你的應用程式只提供一個單一的訂購方式，你可能會呼叫 `main` 或 `primary`。第二個參數是用於特定 Striple 方案的使用者訂購。這個值應該對應到在 Stripe 計畫的識別碼。

`create` 方法會自動建立與 Stripe 的交易，以及將 Stripe 客戶 ID 和其他相關帳款資訊更新到資料庫。如果你的方案有在 Stripe 設定試用期，試用到期日也會自動儲存至使用者的記錄。

#### 額外使用者詳細資料

如果你想自定額外的顧客詳細資料，你可以將資料陣列作為 `create` 方法的第二個參數傳入：

    $user->newSubscription('main', 'monthly')->create($creditCardToken, [
        'email' => $email, 'description' => '我們的第一個客戶'
    ]);

想知道更多 Stripe 支援的額外欄位，請查看 Stripe 的[建立顧客的文件](https://stripe.com/docs/api#create_customer)。

#### 折價券

如果你想在建立訂購的時候使用折價券，可以使用 `withCoupon` 方法：

    $user->newSubscription('main', 'monthly')
         ->withCoupon('code')
         ->create($creditCardToken);

<a name="checking-subscription-status"></a>
### 確認訂購狀態

一旦使用者在你的應用程式訂購，你可以使用多種便捷的方法，很簡單的檢查他們的訂購狀態。首先，當使用者擁有有效訂購時，`subscribed` 方法會回傳 `true`，即使該訂購目前在試用期間：

    if ($user->subscribed('main')) {
        //
    }

`subscribed` 方法很適合用在[路由中介層](/laravel_tw/docs/5.2/middleware)，讓你可以透過使用者的訂購狀態，過濾存取路由及控制器：

    public function handle($request, Closure $next)
    {
        if ($request->user() && ! $request->user()->subscribed('main')) {
            // 此使用者不是付費使用者...
            return redirect('billing');
        }

        return $next($request);
    }

如果你想確認使用者是否還在他們的試用期內，你可以使用 `onTrial` 方法。此方法在向使用者顯示他們還在試用期內的警告是很有用的：

    if ($user->subscription('main')->onTrial()) {
        //
    }

`onPlan` 方法可以用 Stripe ID 來確認使用者是否訂購某方案：

    if ($user->onPlan('monthly')) {
        //
    }

#### 取消訂購狀態

若要確認使用者是否曾經訂購，但是已經取消他們的訂購，你可以使用 `cancelled` 方法：

    if ($user->subscription('main')->cancelled()) {
        //
    }

你可能想確認使用者是否已經取消訂購，但是還在他們訂購完全到期前的「寬限期」。例如，如果使用者在三月五號取消了訂購，但是服務會到三月十號才過期。那麼使用者到三月十號前都是「寬限期」。注意，`subscribed` 方法在這個期間仍然會回傳 `true`。

    if ($user->subscription('main')->onGracePeriod()) {
        //
    }

<a name="changing-plans"></a>
### 改變方案

當使用者在你的應用程式訂購之後，他們有時可能想更改自己的訂購方案。使用 `swap` 方法可以把使用者轉換到新的訂購。舉個例子，我們可以簡單的將使用者切換至 `premium` 訂購：

    $user = App\User::find(1);

    $user->subscription('main')->swap('stripe-plan-id');

如果使用者還在試用期間，試用服務會跟之前一樣可用。此外，如果訂單「數量」還存在的話，會繼續保持這個數量。你可以使用 `invoice` 方法馬上開發票給改變方案的使用者：

    $user->subscription('main')->swap('stripe-plan-id');

    $user->invoice();

<a name="subscription-quantity"></a>
### 訂購數量

有時候訂購行為會跟「數量」有關。例如，你的應用程式可能會依照帳號的使用者人數，**每人**每月收取 10 元。你可以使用 `incrementQuantity` 和 `decrementQuantity` 方法簡單的調整訂購數量：

    $user = User::find(1);

    $user->subscription('main')->incrementQuantity();

    // 增加 5 個訂購數量...
    $user->subscription('main')->incrementQuantity(5);

    $user->subscription('main')->decrementQuantity();

    // 減少 5 個訂購數量...
    $user->subscription('main')->decrementQuantity(5);

另外，你也可以使用 `updateQuantity` 方法來設置指定的數量：

    $user->subscription('main')->updateQuantity(10);

有關訂購數量的更多資料，請參閱 [Stripe 文件](https://stripe.com/docs/guides/subscriptions#setting-quantities)。

<a name="subscription-taxes"></a>
### 訂購稅金

在 Cashier 中，可以很容易的提供發送至 Stripe 的 `tax_percent` 值。要指定一個使用者付費訂購時的稅金比例，請在你的交易模型中實作 `taxPercentage` 方法，並回傳一個介於 0 至 100 間，且不超過兩位小數的數值。

    public function taxPercentage() {
        return 20;
    }

這讓你基於個別模型去運用稅率，可能對橫跨多個國家的用戶群非常有幫助。

<a name="cancelling-subscriptions"></a>
### 取消訂購

要取消一個訂購，只需要在使用者的訂購呼叫 `cancel` 方法：

    $user->subscription('main')->cancel();

當訂購被取消時，Cashier 會自動更新資料庫的 `subscription_ends_at` 欄位。這個欄位會被用來判斷 `subscribed` 方法什麼時候該開始回傳  `false`。例如，如果顧客在三月一號取消訂購，但是服務可以使用到三月五號為止，那麼 `subscribed` 方法在三月五號前都會傳回 `true`。

你想確認使用者是否已經取消他們的訂購，但是還在他們的「寬限期」間，可以使用 `onGracePeriod` 方法：

    if ($user->subscription('main')->onGracePeriod()) {
        //
    }

<a name="resuming-subscriptions"></a>
### 恢復訂購

如果使用者已經取消了他們的訂購，但你想恢復訂購，可以使用 `resume` 方法。使用者**必須**要有恢復訂購的寬限期：

    $user->subscription('main')->resume();

如果客戶取消訂購後，且接著在服務完全過期前恢復訂購，他們將不會在當下被扣款。他們的訂購會被重新啟動，而付款則會依照平常的週期。

<a name="handling-stripe-webhooks"></a>
## 處理 Stripe Webhooks

<a name="handling-failed-subscriptions"></a>
### 訂購失敗

如果顧客的信用卡過期了呢？無需擔心，Cashier 包含了 Webhook 控制器，可以幫你簡單的取消顧客的訂單。只要在路由註冊控制器：

    Route::post(
        'stripe/webhook',
        '\Laravel\Cashier\Http\Controllers\WebhookController@handleWebhook'
    );

這樣就完成了！失敗的交易會經由控制器捕捉並進行處理。控制器在 Stripe 確認訂購已經失敗後 (通常在三次交易嘗試失敗後)，才會取消顧客的訂單。別忘了：你必須設定 Stripe 控制面板設定裡的 webhook URI。

由於 Stripe webhooks 必須繞過 Laravel 的 [CSRF 驗證](/laravel_tw/docs/5.2/routing#csrf-protection)，請確定在增加 URI 例外至你的 `VerifyCsrfToken` 中介層：

    protected $except = [
        'stripe/*',
    ];

<a name="handling-other-webhooks"></a>
### 其他 Webhooks

如果你想要處理額外的 Stripe webhook 事件，只需要繼承 Webhook 控制器。你的方法名稱要對應到 Cashier 預設的名稱，尤其是方法名稱應該使用 `handle` 前綴，並使用「駝峰式」命名法，後面接著你想要處理的 Stripe webhook。例如，如果你想要處理 `invoice.payment_succeeded` webhook，你應該增加一個 `handleInvoicePaymentSucceeded` 方法到控制器。

    <?php

    namespace App\Http\Controllers;

    use Laravel\Cashier\Http\Controllers\WebhookController as BaseController;

    class WebhookController extends BaseController
    {
        /**
         * 處理一個 stripe webhook。
         *
         * @param  array  $payload
         * @return Response
         */
        public function handleInvoicePaymentSucceeded($payload)
        {
            // 處理該事件
        }
    }

<a name="single-charges"></a>
## 一次性收費

如果你想對一個已訂購客戶的信用卡進行「一次性」收費，你可以對一個交易模型實例使用 `charge` 方法。`charge` 方法接受你想收取**應用程式使用貨幣的最低單位**的金額。所以，舉例來說，下方的例子將會對使用者的信用卡收取 100 美分，或是 1 美元：

    $user->charge(100);

`charge` 方法接受一個陣列作為第二個參數，你可以傳遞任何你希望的選項至底層的 Stripe 付費創建器：

    $user->charge(100, [
        'source' => $token,
        'receipt_email' => $user->email,
    ]);

當收費失敗時 `charge` 方法會回傳 `false`。這通常表示收費被拒絕：

    if ( ! $user->charge(100)) {
        // The charge was denied...
    }

如果收費成功，該方法會回傳完整的 Stripe 回應。

<a name="invoices"></a>
## 收據

你可以很簡單的透過 `invoices` 方法取得交易模型的收據資料陣列：

    $invoices = $user->invoices();

當列出收據給客戶時，你可以使用收據的輔助方法來顯示收據的相關資訊。舉例來說，你可能希望列出每個收據至表格中，讓使用者可以簡單的下載其中一個：

    <table>
        @foreach ($invoices as $invoice)
            <tr>
                <td>{% raw %} {{ $invoice->date()->toFormattedDateString() }} {% endraw %}</td>
                <td>{% raw %} {{ $invoice->dollars() }} {% endraw %}</td>
                <td><a href="/user/invoice/{% raw %} {{ $invoice->id }} {% endraw %}">Download</a></td>
            </tr>
        @endforeach
    </table>

<a name="generating-invoice-pdfs"></a>
#### 產生收據的 PDFs

在路由或是控制器中，使用 `downloadInvoice` 方法可以產生收據的 PDF 下載動作。此方法會自動產生正確的 HTTP 回應並發送下載動作至瀏覽器：

    Route::get('user/invoice/{invoice}', function ($invoiceId) {
        return Auth::user()->downloadInvoice($invoiceId, [
            'vendor'  => '你的公司',
            'product' => '你的產品',
        ]);
    });
