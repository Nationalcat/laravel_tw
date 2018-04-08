---
layout: post
title: templates
tag: 5.0
---
# 模板

- [Blade 模板](#blade-templating)
- [其他 Blade 控制語法結構](#other-blade-control-structures)

<a name="blade-templating"></a>
## Blade 模板

Blade 是 Laravel 所提供的一個簡單卻又非常強大的模板引擎。不像控制器頁面佈局，Blade 是使用 _模板繼承_（ template inheritance ）和 _區塊_（ sections ）。所有的 Blade 模板命名都要以 `.blade.php` 結尾。

#### 定義一個 Blade 頁面佈局

	<!-- Stored in resources/views/layouts/master.blade.php -->

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

#### 使用 Blade 頁面佈局

	@extends('layouts.master')

	@section('title', 'Page Title')

	@section('sidebar')
		@@parent

		<p>This is appended to the master sidebar.</p>
	@stop

	@section('content')
		<p>This is my body content.</p>
	@stop

請注意，`extend` Blade 頁面佈局的視圖，只是覆寫頁面佈局中定義的 section。如果在繼承的頁面裡，想顯示原本頁面佈局中 section 裡的內容，那就要在 section 中使用 `@@parent`  語法，把內容附加到頁面佈局中，像是側邊欄區塊或者頁尾區塊。

在某些時候，像是不確定 section 有沒有被定義，你可能會想要傳一個預設的值給 `@yield`。可以傳入第二個參數作為預設值：

	@yield('section', 'Default Content')

<a name="other-blade-control-structures"></a>
## 其他 Blade 控制語法結構

#### 印出資料

	Hello, {% raw %} {{ $name }} {% endraw %}.

	The current UNIX timestamp is {% raw %} {{ time() }} {% endraw %}.

#### 確認資料存在才印出

有時候您想要印出一個變數，但您不確定這個變數是否存在，基本上，你會想要這樣做：

	{% raw %} {{ isset($name) ? $name : 'Default' }} {% endraw %}

	然而，比起使用三元運算子，Blade 讓您可以使用下面這種更簡便的語法：

	{% raw %} {{ $name or 'Default' }} {% endraw %}

#### 使用大括號顯示文字

如果需要顯示一個被大括號包起來的字串，您可以在大括號之前加上 `@` 符號跳脫 Blade 的解析：

	@{% raw %} {{ This will not be processed by Blade }} {% endraw %}

如果您不想資料被轉義, 也可以使用如下語法：

	Hello, {!! $name !!}.

> **注意：** 在應用程式裡印出使用者所提供的內容時要非常小心。請記得永遠使用雙重大括號來轉義內容中的 HTML 字碼。

#### If 語法

	@if (count($records) === 1)
		I have one record!
	@elseif (count($records) > 1)
		I have multiple records!
	@else
		I don't have any records!
	@endif

	@unless (Auth::check())
		You are not signed in.
	@endunless

#### 迴圈

	@for ($i = 0; $i < 10; $i++)
		The current value is {% raw %} {{ $i }} {% endraw %}
	@endfor

	@foreach ($users as $user)
		<p>This is user {% raw %} {{ $user->id }} {% endraw %}</p>
	@endforeach

	@forelse($users as $user)
		<li>{% raw %} {{ $user->name }} {% endraw %}</li>
	@empty
		<p>No users</p>
	@endforelse

	@while (true)
		<p>I'm looping forever.</p>
	@endwhile

#### 載入子視圖

	@include('view.name')

您也可以傳入陣列資料傳遞到載入的子視圖：

	@include('view.name', ['some' => 'data'])

#### 覆寫區塊

如果想要重寫掉前面區塊中的內容，您可以使用 `overwrite` 語法：

	@extends('list.item.container')

	@section('list.item.content')
		<p>This is an item of type {% raw %} {{ $item->type }} {% endraw %}</p>
	@overwrite

#### 顯示語言行

	@lang('language.line')

	@choice('language.line', 1)

#### 註釋

	{% raw %} {{-- This comment will not be in the rendered HTML --}} {% endraw %}
