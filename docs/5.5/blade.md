---
layout: post
title: blade
tag: 5.5
---
# Blade 模板

- [介紹](#introduction)
- [模板繼承](#template-inheritance)
    - [定義頁面佈局](#defining-a-layout)
    - [繼承模板佈局](#extending-a-layout)
- [元件 & Slots](#components-and-slots)
- [顯示資料](#displaying-data)
    - [Blade & JavaScript 框架](#blade-and-javascript-frameworks)
- [控制結構](#control-structures)
    - [If 陳述句](#if-statements)
    - [Switch 陳述句](#switch-statements)
    - [迴圈](#loops)
    - [迴圈變數](#the-loop-variable)
    - [註解](#comments)
    - [PHP](#php)
- [引入子視圖](#including-sub-views)
    - [為集合渲染視圖](#rendering-views-for-collections)
- [Stacks](#stacks)
- [服務注入](#service-injection)
- [擴充 Blade](#extending-blade)
    - [自訂 If 陳述句](#custom-if-statements)

<a name="introduction"></a>
## 介紹

Blade 是 Laravel 所提供的簡單且強大的模板引擎。相較於其它知名的 PHP 模板引擎，Blade 並不會限制你必須在視圖中使用 PHP 程式碼。所有 Blade 視圖會被編譯成一般的 PHP 程式碼並快取直到它們被更動為止，這代表著基本上 Blade 不會對你的應用程式產生負擔。Blade 視圖檔案使用 `.blade.php` 做為副檔名，且通常儲存於 `resources/views` 資料夾。

<a name="template-inheritance"></a>
## 模板繼承

<a name="defining-a-layout"></a>
### 定義頁面框架

使用 Blade 模板的兩個主要優點為**模板繼承**與**區塊**。讓我們先看一個簡單的範例來上手。首先，我們確認一下「主要的」頁面佈局。由於大多數的網頁應用程式在不同頁面都保持著相同的佈局方式，這便於定義這個佈局為單一的 Blade 視圖：

    <!-- Stored in resources/views/layouts/app.blade.php -->

    <html>
        <head>
            <title>App Name - @yield('title')</title>
        </head>
        <body>
            @section('sidebar')
                This is the master sidebar.
            @show

            <div class="container">
                @yield('content')
            </div>
        </body>
    </html>

如你所見，這個檔案包含了傳統的 HTML 語法。不過，請注意 `@section` 與 `@yield` 指令。正如其名， `@section` 指令定義一個內容區塊，而 `@yield` 指令被用來顯示給定區塊的內容。

現在，我們已經定義了這個應用程式的佈局，讓我們來定義一個繼承此佈局的子頁面。

<a name="extending-a-layout"></a>
### 繼承頁面框架

定義子視圖時，請使用 `@extends` 指令告訴子視圖應該要「繼承」哪個佈局。而繼承 Blade 佈局的視圖可以使用 `@section` 指令將內容注入佈局中。提醒一下，上面範例出現的 `@yield` 會用來顯示子視圖使用 `@section` 裡的內容：

    <!-- Stored in resources/views/child.blade.php -->

    @extends('layouts.app')

    @section('title', 'Page Title')

    @section('sidebar')
        @parent

        <p>這裡放置側欄</p>
    @endsection

    @section('content')
        <p>在這裡放主要內容</p>
    @endsection

在這個範例中，`sidebar` 區塊利用了 `@parent` 指令增加（而不是覆蓋）內容至佈局的側邊欄。`@parent` 指令會在視圖輸出時被置換成佈局的內容。

> {tip} 有一點要澄清一下，這邊的 `sidebar` 結束時是使用 `@endsection` 而不是 `@show`。 `@endsection` 指令只會定義一個區塊，而 `@show` 會**直接產生**這個區塊的內容。

你可以在路由使用全域的 `view` 輔助函式來取得 Blade 視圖：

    Route::get('blade', function () {
        return view('child');
    });

<a name="components-and-slots"></a>
## 元件 & Slots

元件與 slots 提供類似佈局的 `@section` 的用法，然後你會發現這種用法的可讀性很高。首先，讓我們設想一個可以在整個應用程式中被重複使用的「提醒」元件：

    <!-- /resources/views/alert.blade.php -->

    <div class="alert alert-danger">
        {% raw %} {{ $slot }} {% endraw %}
    </div>

`{% raw %} {{ $slot }} {% endraw %}` 變數會把我們希望放入的內容注入到元件中。這時候，我們能使用 Blade 中的 `@component` 指令來建構這個元件。

    @component('alert')
        <strong>Whoops!</strong> Something went wrong!
    @endcomponent

有時候元件定義多個 slots 是很有幫助的。讓我們修改一下剛剛的寫的「提醒元件」，讓他能再注入一個「title」。可以通過簡單地「呼叫」他們對應的變數名稱讓內容顯示在 `@slots` 區塊內：

    <!-- /resources/views/alert.blade.php -->

    <div class="alert alert-danger">
        <div class="alert-title">{% raw %} {{ $title }} {% endraw %}</div>

        {% raw %} {{ $slot }} {% endraw %}
    </div>

現在，我們終於能使用 `@slot` 指令注入內容到對應變數名稱的 slot 中。任何不在 `@slot` 區塊內的內容都會放到 `$slot` 這個變數裡：

    @component('alert')
        @slot('title')
            Forbidden
        @endslot

        You are not allowed to access this resource!
    @endcomponent

#### 傳送額外的資料到元件

有時你可能需要傳遞額外資料給元件。出於這個需求，你能傳遞一組陣列到 `@component` 的第二個參數。全部的資料將以變數的形式提供給元件的模板。

    @component('alert', ['foo' => 'bar'])
        ...
    @endcomponent

<a name="displaying-data"></a>
## 顯示資料

你可以使用大括號包住變數以顯示傳遞至 Blade 視圖的資料。舉例而言，就像以下的路由設定：

    Route::get('greeting', function () {
        return view('welcome', ['name' => 'Samantha']);
    });

你可以像這樣顯示 `name` 變數的內容：

    Hello, {% raw %} {{ $name }} {% endraw %}.

當然，也不是一定只能顯示傳遞至視圖的變數內容。你也可以顯示 PHP 函式的結果。實際上，你可以放置任何你需要的 PHP 程式碼到 Blade 顯示語法裡面：

    目前的 UNIX 時間戳記為 {% raw %} {{ time() }} {% endraw %}.

> {tip} Blade 的 `{% raw %} {{}} {% endraw %}` 語法已經自動以 PHP 的 `htmlentites` 函式防禦 XSS 攻擊。

#### 顯示未跳脫的資料

在預設下，Blade `{% raw %} {{ }} {% endraw %}` 語法會自己使用 PHP `htmlspecialchars` 原生函式來避免 XSS 攻擊。如果你不想要你的資料被跳脫處理，可以使用下面的語法。

    Hello, {!! $name !!}.

> {note} 當你的應用程式列印資料時，要特別小心由使用者提供的內容，最好使用跳脫的雙大括號語法來防止 XSS 攻擊。

#### 渲染 JSON

有時候能會傳送一組陣列到你的視圖，目的是將陣列渲染成 JSON 來初始化 JavaScript 變數，例如：

    <script>
        var app = <?php echo json_encode($array); ?>;
    </script>

你可以使用 @json Blade 指令：

    <script>
        var app = @json($array);
    </script>

<a name="blade-and-javascript-frameworks"></a>
### Blade & JavaScript 框架

由於許多 JavaScript 框架也使用「大」括號在瀏覽器中顯示給定的表達式，你可以使用 `@` 符號來告知 Blade 渲染引擎該表達式應該維持原樣。舉個例子：

    <h1>Laravel</h1>

    Hello, @{% raw %} {{ name }} {% endraw %}.

在這個範例中，`@` 符號會被 Blade 移除。而且，Blade 引擎會保留 `{% raw %} {{ name }} {% endraw %}` 表達式，如此一來便可讓其它 JavaScript 框架所應用。

#### `@verbatim` 指令

如果你的模板大部分要用來顯示 JavaScript 變數，你可以把 HTML 語法內容放到 `@verbatim` ，如此一來你就不用在每個 Blade echo 語句前加上 `@` 符號：

    @verbatim
        <div class="container">
            Hello, {% raw %} {{ name }} {% endraw %}.
        </div>
    @endverbatim

<a name="control-structures"></a>
## 控制結構

除了模板繼承與顯示資料功能以外，Blade 也提供了方便的縮寫給一般的 PHP 控制敘述，像是條件陳述式和迴圈。這些縮寫提供了乾淨、簡潔的方式來使用 PHP 的控制結構，同時還保留對應在 PHP 中熟悉且同樣的語法。

<a name="if-statements"></a>
### If 陳述式

你可以使用 `@if`、`@elseif`、`@else` 及 `@endif` 指令建構 if 陳述式。這些指令的功能等同於在 PHP 中的語法：

    @if (count($records) === 1)
        我有一條紀錄！
    @elseif (count($records) > 1)
        我有多條紀錄！
    @else
        我沒有任何記錄！
    @endif

為了方便，Blade 也提供了 `@unless` 指令：

    @unless (Auth::check())
        You are not signed in.
    @endunless

除了上面已經討論過的指令外，`@isset` 和 `@empty` 對於 PHP 原生函式提供更優雅的方法：

    @isset($records)
        // $records is defined and is not null...
    @endisset

    @empty($records)
        // $records is "empty"...
    @endempty

#### 優雅的認證寫法

`@auth` 和 `@guest` 可以用來快速確認當前使用者是否被認證或是未授權的訪客：

    @auth
        // 使用者已經被認證...
    @endauth

    @guest
        // 使用者尚未被認證...
    @endguest

如果有需要，當你使用 `@auth` 和 `@guest` 指令時，你可以指定[認證守衛](/laravel_tw/docs/5.5/authentication)來確認身份：

    @auth('admin')
        // 使用者已經被認證...
    @endauth

    @guest('admin')
        // 使用者尚未被認證...
    @endguest

<a name="switch-statements"></a>
### Switch 陳述式

可以使用 `@switch`、`@case`、`@break`、`@default` 和 `@endswitch` 來建構 Switch 語法：

    @switch($i)
        @case(1)
            First case...
            @break

        @case(2)
            Second case...
            @break

        @default
            Default case...
    @endswitch

<a name="loops"></a>
### 迴圈

除了條件句外，Blade 支援原生 PHP 迴圈的語法。再說一次，這些指令都能對應到原生 PHP 功能：

    @for ($i = 0; $i < 10; $i++)
        The current value is {% raw %} {{ $i }} {% endraw %}
    @endfor

    @foreach ($users as $user)
        <p>This is user {% raw %} {{ $user->id }} {% endraw %}</p>
    @endforeach

    @forelse ($users as $user)
        <li>{% raw %} {{ $user->name }} {% endraw %}</li>
    @empty
        <p>No users</p>
    @endforelse

    @while (true)
        <p>I'm looping forever.</p>
    @endwhile

> {tip} 正在執行迴圈時，你可以使用[迴圈變數](#the-loop-variable)來取得迴圈的訊息。例如你想知道目前是在迴圈的第一次還是最後一次。

正在使用迴圈的時候，你也可以終止或略過當前的迭代：

    @foreach ($users as $user)
        @if ($user->type == 1)
            @continue
        @endif

        <li>{% raw %} {{ $user->name }} {% endraw %}</li>

        @if ($user->number == 5)
            @break
        @endif
    @endforeach

你也可以在迴圈裡放入條件句：

    @foreach ($users as $user)
        @continue($user->type == 1)

        <li>{% raw %} {{ $user->name }} {% endraw %}</li>

        @break($user->number == 5)
    @endforeach

<a name="the-loop-variable"></a>
### 迴圈變數

正在執行迴圈時，可以在迴圈內使用 `$loop`。這個變數提供存取一些有用的資訊，像是目前迴圈索引以及當前迴圈是第一次還是最後一次迭代：

    @foreach ($users as $user)
        @if ($loop->first)
            This is the first iteration.
        @endif

        @if ($loop->last)
            This is the last iteration.
        @endif

        <p>This is user {% raw %} {{ $user->id }} {% endraw %}</p>
    @endforeach

如果你使用一個巢狀迴圈，你可以透過 `parent` 屬性去存取 `$loop` 變數：

    @foreach ($users as $user)
        @foreach ($user->posts as $post)
            @if ($loop->parent->first)
                This is first iteration of the parent loop.
            @endif
        @endforeach
    @endforeach

`$loop` 也包含了其他有用的屬性：

屬性  | 描述
------------- | -------------
`$loop->index`  |  當前迭代次數的索引（從 0 開始
`$loop->iteration`  |  當前迴圈的迭代（從 1 開始）
`$loop->remaining`  |  迴圈剩餘的迭代
`$loop->count`  |  計算迭代中的陣列項目總數
`$loop->first`  |  判斷是否是第一次迭代
`$loop->last`  |  判斷是否是最後一次迭代
`$loop->depth`  |  當前迴圈的巢狀級別
`$loop->parent`  |  在巢狀迴全中，使用父迴圈的變數

<a name="comments"></a>
### 註解

Blade 允許你在你的視圖中寫註解。然而，這不像 HTML 的註解。Blade 註解不是寫入 HTML 並返回到你的網頁上：

    {% raw %} {{-- 這裡的註解不會出現再渲染後的 HTML --}} {% endraw %}

<a name="php"></a>
### PHP

在一些情況下，你會想在視圖上直接寫 PHP 程式碼。你能使用 `@php` 這個Blade 指令在你的模板中執行 PHP ：

    @php
        //
    @endphp

> {tip} 雖然 Blade 提供這個功能，但過度使用會使你的應用程式變的不優雅唷！

<a name="including-sub-views"></a>
## 引入子視圖

`@include` 這個 Blade 指令可以讓你引入其他 Blade 的視圖，主要視圖使用到的變數可與子視圖共用：

    <div>
        @include('shared.errors')

        <form>
            <!-- Form Contents -->
        </form>
    </div>

儘管被引入的視圖會繼承父視圖中的所有資料，你也可以傳遞額外資料的陣列至被引入的頁面：

    @include('view.name', ['some' => 'data'])

當然，如果你嘗試 `@include` 一個不存在的視圖， Laravel 會拋出錯誤訊息。如果你想引入一個不一定存在的視圖，你應當使用 `@includeIf`：

    @includeIf('view.name', ['some' => 'data'])

如果你想要根據布林值來決定是否要 `@include` 視圖內容，你可以使用 `@includeWhen`：

    @includeWhen($boolean, 'view.name', ['some' => 'data'])

如果你想要從已存在的視圖陣列中引入第一個視圖，你可以使用 `@includeFirst` ：

    @includeFirst(['custom.admin', 'admin'], ['some' => 'data'])

> {note} 你應該避免在 Blade 視圖中使用 `__DIR__` 和 `__FILE__` 常數，因為他們會引用視圖被的快取位置。

<a name="rendering-views-for-collections"></a>
### 為集合渲染視圖

你可以使用 Blade 的 `@each` 指令將迴圈及引入結合成一行：

    @each('view.name', $jobs, 'job')

第一個參數為對陣列或集合的每個元素渲染的局部視圖。第二個參數為你要迭代的陣列或集合，而第三個參數為迭代時被分配至視圖中的變數名稱。所以，舉例來說，如果你迭代一個 `jobs` 陣列，通常你會希望在局部視圖中透過 `job` 變數存取每一個 `job`。目前迭代的 key 在你的視圖部份將會被作為 key 變數。

你也可以傳遞第四個參數至 `@each` 指令。此參數為當給定的陣列為空時，將會被渲染的視圖。

    @each('view.name', $jobs, 'job', 'view.empty')

> {note} 視圖透過 `@each` 渲染的視圖不會繼承父視圖的變數。如果子視圖需要這些變數，你應該改用 `@foreach` 和 `@include` 。

<a name="stacks"></a>
## Stacks

Blade 可以讓你使用 `@push` 將已命名的 Stack 在其他的視圖或佈局中渲染，這樣能更優雅的在子視圖中指定需要的 JavaScript 程式庫：

    @push('scripts')
        <script src="/example.js"></script>
    @endpush

你可以根據需求推送多個 `@stack`。你只需要將 Stack 名稱傳給 `@stack`，就能來渲染完整 stack 內容：

    <head>
        <!-- Head Contents -->

        @stack('scripts')
    </head>

<a name="service-injection"></a>
## 服務注入

`@inject` 指令可以取出 Laravel [服務容器](/laravel_tw/docs/5.5/container)中的服務。傳遞給 `@inject` 的第一個參數為置放該服務的變數名稱，而第二個參數為你想要解析的服務的類別或是介面的名稱：

    @inject('metrics', 'App\Services\MetricsService')

    <div>
        Monthly Revenue: {% raw %} {{ $metrics->monthlyRevenue() }} {% endraw %}.
    </div>

<a name="extending-blade"></a>
## 擴充 Blade

Blade 甚至可以讓你使用 `directive` 方法來自訂想要的指令。當 Blade 編譯器遇到自訂的指令時，它會呼叫所有註冊的指令提供的回呼函式。

以下範例建立一個 `@datetime($var)` Blade 指令，用於格式化給定的`$var`，它會是一個 `DateTime` 的實例：

    <?php

    namespace App\Providers;

    use Illuminate\Support\Facades\Blade;
    use Illuminate\Support\ServiceProvider;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * Perform post-registration booting of services.
         *
         * @return void
         */
        public function boot()
        {
            Blade::directive('datetime', function ($expression) {
                return "<?php echo ($expression)->format('m/d/Y H:i'); ?>";
            });
        }

        /**
         * Register bindings in the container.
         *
         * @return void
         */
        public function register()
        {
            //
        }
    }

如你所見，我們會串接 `format` 方法到任何你想要傳入指令的表達式。所以，在這個範例的指令最後產生的 PHP 會是：

    <?php echo ($var)->format('m/d/Y H:i'); ?>

> {note} 在更新 Blade 指令的邏輯後，你會需要刪除全部 Blade 視圖的快取。被快取的 Blade 視圖可以使用 Artisan 的 view:clear 指令來移除。

<a name="custom-if-statements"></a>
### 自訂 If 陳述句

在定義簡單的條件陳述句時，有時候會比編譯自訂的 Blade 指令更複雜。是因為 Blade 提供了 `Blade::if` 方法，它允許你使用閉包快速自訂 Blade 條件陳述句指令。例如，讓我們定義一個自訂的條件句來檢查應用程式當下的環境，我們可以在 `AppServiceProvider` 的 `boot` 方法這麼做：

    use Illuminate\Support\Facades\Blade;

    /**
     * Perform post-registration booting of services.
     *
     * @return void
     */
    public function boot()
    {
        Blade::if('env', function ($environment) {
            return app()->environment($environment);
        });
    }

一旦自訂了 Blade 條件句，我們就能在模板上輕易的使用它們：

    @env('local')
        // 應用程式在本機環境中...
    @elseenv('testing')
        // 應用程式在測試環境...
    @else
        // 應用程式都不在應用程式與本機環境...
    @endenv
