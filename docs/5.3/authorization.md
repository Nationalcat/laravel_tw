---
layout: post
title: authorization
tag: 5.3
---
# 授權

- [簡介](#introduction)
- [Gates](#gates)
    - [編寫 Gates](#writing-gates)
    - [授權行為](#authorizing-actions-via-gates)
- [建立原則（policy）](#creating-policies)
    - [產生原則](#generating-policies)
    - [註冊原則](#registering-policies)
- [編寫原則](#writing-policies)
    - [原則方法](#policy-methods)
    - [方法中不使用模型](#methods-without-models)
    - [原則過濾器](#policy-filters)
- [使用原則授權行為](#authorizing-actions-using-policies)
    - [透過使用者模型](#via-the-user-model)
    - [透過中介層](#via-middleware)
    - [透過控制器輔助方法](#via-controller-helpers)
    - [透過 Blade 模板](#via-blade-templates)

<a name="introduction"></a>
## 簡介

除了內建提供的[認證](/laravel_tw/docs/5.3/authentication)服務外，Laravel 還提供簡單的方式來授權使用者對特定資源的行為（action）。類似使用者認證，Laravel 有二種主要方式來實現使用者的行為授權：gate 和原則（policy）。

可以把 gate 和原則想像成路由和控制器的關係。Gate 提供了一個簡單、基於閉包的方式來授權認證。原則和控制器類似，在特定的模型或者資源中透過分組來實現授權認證的邏輯。我們先來看看 gate，然後再看原則。

在應用程式中，不需要將 gate 和原則當作相互排斥的方式。大部分的應用程式很可能同時包含 gate 和原則，並且能很好的工作。Gate 大部分應用在和模型或資源無關的行為，比如檢視管理員的面板。與此相比，原則應該用在和模型或資源相關的特定行為。

<a name="gates"></a>
## Gates

<a name="writing-gates"></a>
### 編寫 Gates

Gates 是一個閉包，用來決定使用者是否授權給定的行為，而典型的做法是在 `App\Providers\AuthServiceProvider` 類別中使用 `Gate` facade 來定義。Gates 的第一個參數為使用者實例，並且選擇性地接受更多參數，例如一個有相關的 Eloquent 模型：

    /**
     * 註冊任何的認證或授權服務。
     *
     * @return void
     */
    public function boot()
    {
        $this->registerPolicies();

        Gate::define('update-post', function ($user, $post) {
            return $user->id == $post->user_id;
        });
    }

<a name="authorizing-actions-via-gates"></a>
### 授權行為

使用 gate 來授權行為時，應當使用 `allows` 或 `denies` 方法。值得注意的是，並不需要傳遞當前認證過的使用者給這些方法。Laravel 會自動處理好當前使用者，然後傳遞給 gate 閉包：

    if (Gate::allows('update-post', $post)) {
        // 當前使用者可以更新此篇 $post...
    }

    if (Gate::denies('update-post', $post)) {
        // 當前使用者不可以更新此篇 $post...
    }

如果需要指定特定的使用者被授權執行某個行為，可以使用 `Gate` facade 中的 `forUser` 方法：

    if (Gate::forUser($user)->allows('update-post', $post)) {
        // 特定 $user 可以更新此篇 $post...
    }

    if (Gate::forUser($user)->denies('update-post', $post)) {
        // 特定 $user 不可以更新此篇 $post...
    }

<a name="creating-policies"></a>
## 建立原則（Policy）

<a name="generating-policies"></a>
### 產生原則

原則是個對特定模型或資源，安排授權邏輯的類別。例如，應用程式是個部落格的話，會需要有 `Post` 模型和對應的 `PostPolicy` 來授權使用者 的行為，例如建立或更新張貼的文章。

可以透過 `make:policy` [artisan 指令](/laravel_tw/docs/5.3/artisan) 產生一個原則。產生的原則會被放置於 `app/Policies` 目錄中，若這個目錄不存在，Laravel 則會自動建立：

    php artisan make:policy PostPolicy

`make:policy` 會產生空的原則類別。如果希望這個類別包含基本的「CRUD」原則方法，可以在使用指令時加上 `--model` 參數：

    php artisan make:policy PostPolicy --model=Post

> {tip} 所有授權原則透過 Laravel [服務容器](/laravel_tw/docs/5.3/container) 解析，意指你可以在授權原則的建構子對任何需要的依賴使用型別提示，它們將會被自動注入。

<a name="registering-policies"></a>
### 註冊原則

一旦該原則存在，還需要將它進行註冊。`AuthServiceProvider` 包含了 `policies` 屬性，可將 Eloquent 模型對應至管理它們的原則。當給定模型時，註冊原則將引導 Laravel 去判斷授權的行為，該使用何種原則：

    <?php

    namespace App\Providers;

    use App\Post;
    use App\Policies\PostPolicy;
    use Illuminate\Support\Facades\Gate;
    use Illuminate\Foundation\Support\Providers\AuthServiceProvider as ServiceProvider;

    class AuthServiceProvider extends ServiceProvider
    {
        /**
         * 應用程式的原則對應
         *
         * @var array
         */
        protected $policies = [
            Post::class => PostPolicy::class,
        ];

        /**
         * 註冊應用程式的任何認證或授權服務
         *
         * @return void
         */
        public function boot()
        {
            $this->registerPolicies();

            //
        }
    }

<a name="writing-policies"></a>
## 編寫原則（Policy）

<a name="policy-methods"></a>
### 原則方法

一旦原則被註冊，接著可以為每個所授權的行為增加方法。例如，讓我們在 `PostPolicy` 中定義一個 `update` 方法，它會判斷給定的 `User` 是否可以更新一筆 `Post`。

`update` 方法接受 `User` 和 `Post` 實例作為參數，並且應當回傳 true 或 false 來指明使用者是否授權更新給定的 `Post`。因此，這個例子中，我們判斷使用者的 `id` 是否和 post 中的 `user_id` 匹配：

    <?php

    namespace App\Policies;

    use App\User;
    use App\Post;

    class PostPolicy
    {
        /**
         * Determine if the given post can be updated by the user.
         *
         * @param  \App\User  $user
         * @param  \App\Post  $post
         * @return bool
         */
        public function update(User $user, Post $post)
        {
            return $user->id === $post->user_id;
        }
    }

你可以接著在此授權原則定義額外的方法，作為各種行為的許可權所需要的原則。例如，你可以定義 `view` 或 `delete` 方法來授權 `Post` 的多種行為。也可以為自定原則方法使用自己喜歡的名字。

> {tip} 如果在 Artisan 指令列產生原則時使用 `--model` 選項，會自動包含 `view`、`create`、`update` 和 `delete` 行為。

<a name="methods-without-models"></a>
### 方法中不使用模型

有些原則方法只需當前認證使用者參數，而不用傳入授權的模型實例。最常見的狀況為授權 `create` 行為。例如，在建立一篇部落格時，你可能希望檢查一下當前使用者是否被授權建立任意的文章(post)。

當定義一個不需要傳入模型實例的原則方法時，比如 `create` 方法，不用傳入模型實例，只需定義期望被授權的使用者：

    /**
     * 判斷給定使用者是否可以建立文章
     *
     * @param  \App\User  $user
     * @return bool
     */
    public function create(User $user)
    {
        //
    }

 {tip} 如果使用 `--model` 選項產生原則，會在產生的原則中自動定義好相關的「CRUD」原則方法。

<a name="policy-filters"></a>
### 原則過濾器

對特定使用者，可能希望透過指定的原則授權所有行為，要達到這個目的，可以在原則中定義一個 `before` 方法。`before` 方法會在任何其他方法之前執行，在實際呼叫預期原則方法之前，為您提供授權行為的機會。此功能最常用於對已授權的應用程式管理員執行任何操作：

    public function before($user, $ability)
    {
        if ($user->isSuperAdmin()) {
            return true;
        }
    }

如果想拒絕使用者的所有授權，應該從 `before` 方法回傳 `false`。如果回傳 `null`，授權機制將進入原則方法。

<a name="authorizing-actions-using-policies"></a>
## 使用原則（Policy）授權行為 

<a name="via-the-user-model"></a>
### 透過使用者模型

Laravel 應用程式內建的 User 模型包含二個有用的方法來授權行為：`can` 和 `cant`。`can` 方法需要指定期望被授權的行為和相關的模型。例如，判斷使用者是否授權更新給定的 Post 模型：

    if ($user->can('update', $post)) {
        //
    }

如果給定模型的[原則已註冊](#registering-policies)，`can` 方法會自動呼叫適當的原則方法並且回傳布林值。如果沒有原則註冊到這個模型，`can` 方法會嘗試呼叫和行為名稱相匹配的 Gate 閉包。

#### 不需指定模型的行為

還記得吧，有些行為，比如 `create`，並不需要指定模型實例。在這種情況下，可傳遞類別名稱給 can 方法。當授權該行為時，這個類別名將被用來判斷使用哪個原則：

    use App\Post;

    if ($user->can('create', Post::class)) {
        // 在對應的原則中執行「create」方法...
    }

<a name="via-middleware"></a>
### 透過中介層

Laravel 包含一個中介層，可以在請求到達路由或控制器之前就做行為授權。預設情況下，`Illuminate\Auth\Middleware\Authorize` 中介層的 `can` 鍵被指定到你的 `App\Http\Kernel` 類別中。我們用一個授權使用者更新部落格的例子來講解 `can` 中介層的使用：

    use App\Post;

    Route::put('/post/{post}', function (Post $post) {
        // 當前使用者可以更新（update）文章（post）...
    })->middleware('can:update,post');

在這個例子中，我們傳遞給 `can` 中介層二個參數。第一個是需要授權的行為的名稱，第二個是我們希望傳遞給原則方法的路由參數。這裡因為使用了[隱式模型綁定](/laravel_tw/docs/5.3/routing#implicit-binding)，`Post` 模型會被傳入原則方法。如果使用者沒被授權執行指定的 行為，這個中介層會回應帶有 403 狀態碼的 HTTP 回應。

#### 不需指定模型的行為

同樣的，像是 `create` 這種不需模型實例的行為。在這情況下，可以傳入類別名到中介層。當授權該行為時，類別名將用於判斷要使用的原則：

    Route::post('/post', function () {
        // 當前使用者可以建立（create）文章（post）...
    })->middleware('can:create,App\Post');

<a name="via-controller-helpers"></a>
### 透過控制器輔助方法

除了在 `User` 模型中提供輔助方法外，Laravel 也為所有繼承 `App\Http\Controllers\Controller` 基礎類別的控制器提供了一個有用的 `authorize` 方法。如同 `can` 方法，這個方法接收需要授權的行為和對應的模型作為參數。如果行為不被授權，`authorize` 方法會丟出 `Illuminate\Auth\Access\AuthorizationException` 異常，然後被 Laravel 預設的異常處理器轉化為帶有 `403` 狀態碼的 HTTP 回應：

    <?php

    namespace App\Http\Controllers;

    use App\Post;
    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;

    class PostController extends Controller
    {
        /**
         * Update the given blog post.
         *
         * @param  Request  $request
         * @param  Post  $post
         * @return Response
         */
        public function update(Request $request, Post $post)
        {
            $this->authorize('update', $post);

            // 當前使用者可以更新（update）部落格文章...
        }
    }

#### 不需指定模型的行為

和之前討論的一樣，某些行為如 `create` 並不需要指定模型實例。在這種情況下，可傳入類別名給 `authorize` 方法。當授權該行為時，這個類別名將被用來判斷使用哪個原則：

    /**
     * 建立新的部落格文章 Create a new blog post.
     *
     * @param  Request  $request
     * @return Response
     */
    public function create(Request $request)
    {
        $this->authorize('create', Post::class);

        // 當前使用者可以建立（create）部落格文章...
    }

<a name="via-blade-templates"></a>
### 透過 Blade 模板

當編寫 Blade 模板時，若使用者被授權執行給定的行為，你可能希望只展示頁面的一部分給他。例如，你可能希望只展示更新表單給有權更新部落格文章的使用者。這種情況下，你可以直接使用 `@can` 和 `@cannot` 指令。

    @can('update', $post)
        <!-- 當前使用者可以更新部落格文章 -->
    @elsecan('create', $post)
        <!-- 當前使用者可以建立新的部落格文章 -->
    @endcan

    @cannot('update', $post)
        <!-- 當前使用者不可以更新部落格文章 -->
    @elsecannot('create', $post)
        <!-- 當前使用者不可以建立新的部落格文章 -->
    @endcannot

這些指令為 `@if` 和 `@unless` 的寫法提供了方便的縮寫。上述的 `@can` 和 `@cannot` 各自轉換為如下宣告：

    @if (Auth::user()->can('update', $post))
        <!-- 當前使用者可以更新部落格文章 -->
    @endif

    @unless (Auth::user()->can('update', $post))
        <!-- 當前使用者不可以更新部落格文章 -->
    @endunless

#### 不需指定模型的行為

和大部分其他的授權方法類似，當行為不需要模型實例時，則可以傳入類別名給 `@can` 和 `@cannot` 指令：

    @can('create', Post::class)
        <!-- 當前使用者可以建立新的部落格文章 -->
    @endcan

    @cannot('create', Post::class)
        <!-- 當前使用者不可以建立新的部落格文章 -->
    @endcannot
