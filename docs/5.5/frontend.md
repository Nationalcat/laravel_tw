---
layout: post
title: frontend
tag: 5.5
---
# JavaScript 與 CSS 的起手式

- [介紹](#introduction)
- [撰寫 CSS](#writing-css)
- [撰寫 JavaScript](#writing-javascript)
    - [撰寫 Vue 元件](#writing-vue-components)
    - [使用 React](#using-react)

<a name="introduction"></a>
## 介紹

雖然 Laravel 並不規定你應該要使用哪個 JavaScript 或 CSS 的預處理器，不過預設會使用 [Bootstrap](https://getbootstrap.com/) 與 [Vue](https://vuejs.org) 來協助開發許多應用程式。預設的 Laravel 可以使用 [NPM](https://www.npmjs.org) 來安裝這兩款前端框架。

#### CSS

[Laravel Mix](/laravel_tw/docs/5.5/mix) 提供一個整齊且直觀的 API 來編譯 SASS 或 Less，這可以新增變數、mixins 和其他強大的功能來延伸純 CSS 內容。在本文件中，我們會來簡略的討論一般 CSS 編譯。然而，你應該去查看完整的 [Laravel Mix 文件](/laravel_tw/docs/5.5/mix)來得到更多關於編譯 SASS 或 Less 的資訊。

#### JavaScript

Laravel 不會強迫你使用特定的 JavaScript 框架或函式庫來建構你的應用程式。實際上，你也可以完全不使用 JavaScript。不過，Laravel 已經建構了一些基本的前端架構，你可以直接使用 [Vue](https://vuejs.org) 函式庫來撰寫現代的 JavaScript。Vue 為了使用元件來建構健全的 JavaScript 而提供非常直觀的 API。如同 CSS，我們可以使用 Laravel Mix 來輕易將 JavaScript 元件編譯成一個可用於瀏覽器讀取的 JavaScript 檔案。

#### 移除前端架構

如果你想要從應用程式中移除前端架構，你可以使用 Artisan 的 `preset` 指令。這個指令會與 `none` 選項搭配的時候，從你的應用程式中移除 Bootstrap 和 Vue 框架，並指留下一個空空的 SASS 檔案和一些常用的 JavaScript 函式庫：

    php artisan preset none

<a name="writing-css"></a>
## 撰寫 CSS

Laravel 的 `package.json` 檔案已引入 `bootstrap-sass` 套件來協助你使用 Bootstrap 來建構前端原型。不過，你可以根據實際需要來新增或刪除 `package.json` 檔案中的套件。你並不一定要使用 Bootstrap 框架來建構應用程式的前端——那只是提供給願意選擇使用他的人一個很好的起點。

在編譯你的 CSS 之前，請使用 [Node package manager (NPM)](https://www.npmjs.org) 來安裝你專案的前端依賴項目：

    npm install

如果你使用 `npm install` 成功的安裝了依賴項目，你就能使用 [Laravel Mix](/laravel_tw/docs/5.5/mix#working-with-stylesheets) 來將你的 SASS 檔案編譯到純 CSS 中。`npm run dev` 指令會處理在 `webpack.mix.js` 檔案的指令。通常來說，你編譯過的 CSS 會被放置於 `public/css` 目錄中：

    npm run dev

預設的 `webpack.mix.js` 已引入 Laravel 編譯過的 `resources/assets/sass/app.scss` SASS 檔案。這個 `app.scss` 檔案導入一個 SASS 變數的檔案，並載入 Bootstrap，這會為大多數的應用程式提供一個很好的開始。你可以自由的自訂 `app.scss` 檔案，然而你可以透過[設定 Laravel Mix](/laravel_tw/docs/5.5/mix)來期望使用完全不同的預處理器。

<a name="writing-javascript"></a>
## 撰寫 JavaScript

應用程式需要的所有 JavaScript 依賴項目都可以在專案根目錄中的 `package.json` 檔案裡找到。這個檔案類似於 `composer.json` 檔案，只不過它是針對 JavaScript 依賴項目，而不是 PHP 的依賴項目。你能使用 [Node 套件管理（NPM）](https://www.npmjs.org)來安裝這些依賴項目：

    npm install

> {tip} 預設的 Laravel `package.json` 檔案包含一些套件，像是 `vue` 和 `axios` 來協助你開始建構 JavaScript 應用程式。你可以根據實際需要來新增或刪除 `package.json` 檔案中的套件。

套件一旦被安裝，你就能使用 `npm run dev` 指令來[編譯你的資源](/laravel_tw/docs/5.5/mix)。Webpack 是現在 JavaScript 應用程式的模組封裝器。在你執行 `npm run dev` 指令的時候，Webpack 會執行在 `webpack.mix.js` 檔案中的指令：

    npm run dev

預設的 Laravel `webpack.mix.js` 檔案會去編譯你的 SASS 和 `resources/assets/js/app.js` 檔案。在 `app.js` 檔案中，你可以註冊 Vue 元件，亦或者是喜歡別的框架，來設定自己的 JavaScript 應用程式。你編譯過的 JavaScript 通常會被放置在 `public/js` 目錄中。

> {tip} `app.js` 檔載入 `resources/assets/js/bootstrap.js` 檔案，這會引導和設定 Vue、Axios、jQuery 與其他所有 JavaScript 依賴項目。如果你有額外的 JavaScript 依賴項目要設定，你可以在這個檔案中設定。

<a name="writing-vue-components"></a>
### 撰寫 Vue 元件

預設剛建立的 Laravel 應用程式會包含 `ExampleComponent.vue` Vue 元件，並放在 `resources/assets/js/components` 目錄中。 `ExampleComponent.vue` 檔案是[一個 Vue 元件](https://vuejs.org/guide/single-file-components)的範例，這將 JavaScript 和 HTML 模板定義到同一個檔案中。這一個檔案元件提供了一個非常方便的方法來建構 JavaScript 主導的應用程式。該元件範例會被註冊在 `app.js` 檔案中：

    Vue.component(
        'example-component',
        require('./components/ExampleComponent.vue')
    );

要在應用程式中使用該元件，你可以將它放入 HTML 模板。例如，在執行 Artisan 的 `make:auth` 指令來產生應用程式的認證與註冊畫面後，你就可以將元件放入 `home.blade.php` Blade 模板中：

    @extends('layouts.app')

    @section('content')
        <example-component></example-component>
    @endsection

> {tip} 請記得，你應該在每次更新 Vue 元件的時候就執行 `npm run dev` 指令。亦或者是，你可以執行 `npm run watch` 指令來監控和動自重新編譯元件的每次修改。

當然，如果你想要了解更多關於撰寫 Vue 元件的資訊，你可以去查閱 [Vue 官方文件](https://vuejs.org/guide/)，這提供了更全面且容易閱讀的整個 Vue 框架資訊。

<a name="using-react"></a>
### 使用 React

如果你偏好使用 React 來建構你的 JavaScript 應用程式，Laravel 可以讓你更容易的將 Vue 框架換成 React 框架。在任何剛建立的 Laravel 應用程式上，你可以使用 `preset` 指令並搭配 `react` 選項：

    php artisan preset react

這一個指令會移除 Vue 框架，並替換成 React 框架，也包括一個元件範例。
