---
layout: post
title: mix
---
# 編譯資源 (Laravel Mix)

- [介紹](#introduction)
- [安裝與設定](#installation)
- [執行 Mix](#running-mix)
- [使用樣式表](#working-with-stylesheets)
    - [Less](#less)
    - [Sass](#sass)
    - [Stylus](#stylus)
    - [PostCSS](#postcss)
    - [純 CSS](#plain-css)
    - [URL 的處理](#url-processing)
    - [Source Maps](#css-source-maps)
- [使用 JavaScript](#working-with-scripts)
    - [Vendor 抽取](#vendor-extraction)
    - [React](#react)
    - [原生 JS](#vanilla-js)
    - [自訂 Webpack 設定](#custom-webpack-configuration)
- [複製檔案與目錄](#copying-files-and-directories)
- [版控與快取的清除](#versioning-and-cache-busting)
- [Browsersync 重新載入](#browsersync-reloading)
- [環境變數](#environment-variables)
- [通知](#notifications)

<a name="introduction"></a>
## 介紹

[Laravel Mix](https://github.com/JeffreyWay/laravel-mix) 提供了一個優雅的 API 來為你的應用程式定義 Webpack 建構步驟，並使用幾個常見的 CSS 和 JavaScript 預處理器。通過簡單的方法鏈結，你能優雅的定義你的資源管道。例如：

    mix.js('resources/assets/js/app.js', 'public/js')
       .sass('resources/assets/sass/app.scss', 'public/css');

如果你正在困惑不知道如何開始編譯 Webpack 和資源，那麼你將會愛上 Laravel Mix。當然，這並不強迫你使用它來開發你的應用程式，你可以自由地使用任何想要的資源管道工具，甚至可以完全不用。

<a name="installation"></a>
## 安裝與設定

#### 安裝 Node.js

在開始使用 Mix 之前，首先你必須先確認 Node.js 和 NPM 是否已經安裝在你的開發環境上。

    node -v
    npm -v

預設的 Laravel Homestead 已包含你需要的一切，如果你不使用 Vagrant，那麼你可以從[它們的下載頁面](https://nodejs.org/en/download/)中簡單的圖形介面來輕易的安裝最新版本的 Node.js 和 NPM。

#### Laravel Mix

剩下最後一個步驟是安裝 Laravel Mix。在新安裝的 Laravel 中，你會在你的根目錄結構找到 `package.json` 檔案。預設的 `package.json` 檔案包含能讓你開始的一切需要。就像你的 `composer.json` 檔案，只是它是定義 Node.js 依賴項目，而不是 PHP 的。你可以執行安裝它引用的依賴項目：

    npm install


<a name="running-mix"></a>
## 執行 Mix

Mix 是 [Webpack](https://webpack.js.org) 最上層的設定層。如果你要執行 Mix 任務，你只需要執行預設的 Laravel `package.json` 檔案中其中一個 NPM 腳本指令：

    // 執行所有 Mix 任務...
    npm run dev

    // 執行所有 Mix 任務和壓縮輸出...
    npm run production

#### 即時監控資源的變化

`npm run watch` 指令會在你的終端機持續執行，並監控所有相關檔案的變化。在 Webpack 檢測變化的時候，會自動重新編譯的你的資源：

    npm run watch

在你的檔案改變時，你可以發現在某些環境下 Webpack 不會被更新。如果在你的系統上發生，請考慮使用 `watch-poll` 指令：

    npm run watch-poll

<a name="working-with-stylesheets"></a>
## 使用樣式表

`webpack.mix.js` 檔案是所有資源編譯的入口。可以把它想成是 Webpack 的輕量級設定封裝器。Mix 任務可以一起被鏈結使用，並精準定義你的資源應該如何被編譯。

<a name="less"></a>
### Less

`less` 方法可被用於編譯 [Less](http://lesscss.org/) 到 CSS。讓我們來將我們主要的 `app.less` 檔案編譯到 `public/css/app.css`。

    mix.less('resources/assets/less/app.less', 'public/css');

多次呼叫 `less` 方法可被用於編譯多個檔案：

    mix.less('resources/assets/less/app.less', 'public/css')
       .less('resources/assets/less/admin.less', 'public/css');

如果你希望自訂編譯 CSS 的檔案名稱，你可以傳入完整的檔案路徑作為 `less` 方法的第二個參數：

    mix.less('resources/assets/less/app.less', 'public/stylesheets/styles.css');

如果你需要覆寫[底層的 Less 插件選項](https://github.com/webpack-contrib/less-loader#options)，你可以傳入一個物件作為 `mix.less()` 的第三個參數：

    mix.less('resources/assets/less/app.less', 'public/css', {
        strictMath: true
    });

<a name="sass"></a>
### Sass

`sass` 方法可以讓你編譯 [Sass](http://sass-lang.com/) 到 CSS。你可以使用該方法像是：

    mix.sass('resources/assets/sass/app.scss', 'public/css');

再說一次，類似 `less` 方法，你可以編譯多個 Sass 檔案到他們所屬的 CSS 檔案，甚至可以自訂產生的 CSS 的輸出目錄：

    mix.sass('resources/assets/sass/app.sass', 'public/css')
       .sass('resources/assets/sass/admin.sass', 'public/css/admin');

額外的 [Node-Sass 插件選項](https://github.com/sass/node-sass#options)可以作為第三個參數：

    mix.sass('resources/assets/sass/app.sass', 'public/css', {
        precision: 5
    });

<a name="stylus"></a>
### Stylus

類似於 Less 和 Sass，`stylus` 方法可以讓你編譯 [Stylus](http://stylus-lang.com/) 到 CSS：

    mix.stylus('resources/assets/stylus/app.styl', 'public/css');

你也可以安裝額外的 Stylus 插件，像是 [Rupture](https://github.com/jescalan/rupture)。首先，通過 NPM (`npm install rupture`) 來安裝該插件，接著在你呼叫 `mix.stylus()` 中需要它：

    mix.stylus('resources/assets/stylus/app.styl', 'public/css', {
        use: [
            require('rupture')()
        ]
    });

<a name="postcss"></a>
### PostCSS

[PostCSS](http://postcss.org/)，是一個用來轉換 CSS 的強大工具，並包含在 Laravel Mix 中，所以你隨時都可以使用。預設的 Mix 利用流行的 [Autoprefixer](https://github.com/postcss/autoprefixer) 插件來自動應用所有必要的 CSS3 vendor 的前綴。然而，你可以任意的新增任何適合你應用程式的額外的插件。首先，通過 NPM 安裝想要的插件，然後在你的 `webpack.mix.js` 檔案中引入它們：

    mix.sass('resources/assets/sass/app.scss', 'public/css')
       .options({
            postCss: [
                require('postcss-css-variables')()
            ]
       });

<a name="plain-css"></a>
### 純 CSS

如果你只想將一些純 CSS 樣式合併成一個檔案，你可以使用 `styles` 方法。

    mix.styles([
        'public/css/vendor/normalize.css',
        'public/css/vendor/videojs.css'
    ], 'public/css/all.css');

<a name="url-processing"></a>
### URL 處理

因為 Laravel Mix 建構在 Webpack 上，所以理解一些 Webpack 概念是很重要的。為了 CSS 編譯，Webpack 會在你的樣式表中重寫並優化任何 `url()` 呼叫。 雖然剛開始聽起來很起怪，但這是一個非常強大的功能。試想一下，我們想要編譯包含圖片相對路徑的 Sass：

    .example {
        background: url('../images/example.png');
    }

> {note} 任何給定的 `url()` 的絕對路徑會被排除在路徑重寫之外。例如，`url('/images/thing.png')` 或 `url('http://example.com/images/thing.png')` 不會被修改。

預設的 Laravel Mix 和 Webpack 會找到 `example.png`，接著複製它到你的 `public/images` 資料夾，然後重寫 `url()` 中路徑到你產生的樣時表。因此，你編譯的 CSS 會是：

    .example {
      background: url(/images/example.png?d41d8cd98f00b204e9800998ecf8427e);
    }

如果你對現有的資料夾結構的設定方式感到滿意，那麼這個功能會是有用的。如果是在這個情況下，你可以像這樣停用 `url()` 覆寫：

    mix.sass('resources/assets/app/app.scss', 'public/css')
       .options({
          processCssUrls: false
       });

除了你的 `webpack.mix.js` 檔案，Mix 會不在匹配任何 `url()` 或複製資源到你的 public 目錄。換句話說，編譯的 CSS 看起來會像是你最初撰寫的那樣：

    .example {
        background: url("../images/thing.png");
    }

<a name="css-source-maps"></a>
### Source Maps

預設雖然會停用，Source Maps 可以在你的 `webpack.mix.js` 檔案中呼叫 `mix.sourceMaps()` 來啟用。雖然它會增加編譯和處理的成本負擔，但這會在使用編譯資源時為瀏覽器的開發工具提供額外的除錯資訊。

    mix.js('resources/assets/js/app.js', 'public/js')
       .sourceMaps();

<a name="working-with-scripts"></a>
## 使用 JavaScript

Mix 提供幾個功能來協助你使用 JavaScript 檔案，像是編譯 ECMAScript 2015，模組封裝、壓縮，並簡單的合併純 JavaScript 檔案。更好的是這一切流程都不需要自訂設定：

    mix.js('resources/assets/js/app.js', 'public/js');

就只有這一行程式碼，你現在可以使用下列內容：

<div class="content-list" markdown="1">
- ES2015 語法
- 模組
- `.vue` 檔案的編譯
- 為正式上線環境壓縮檔案
</div>

<a name="vendor-extraction"></a>
### 抽取 Vendor

將所有應用程式特定的 JavaScript 與 Vendor 函式庫綁定再一起的潛在缺點，就是要進行長期快取時會很困難。例如，對你的應用程式的程式碼只做一次更新就要強制瀏覽器重新下載所有 vendor 函式庫，就算其他內容沒有異動。

如果打算頻繁的更新應用程式的 JavaScript，你應該考慮將所有的 Vendor 函式庫抽取到自己的檔案中。這個方式，對應用程式的程式碼的更動不會影響到大型 `vendor.js` 檔案的快取。你可以使用 Mix 的 `extract` 方法來輕易做到：

    mix.js('resources/assets/js/app.js', 'public/js')
       .extract(['vue'])

`extract` 接受一組所有函式庫或你想要抽取到 `vendor.js` 的模組陣列。使用上面內容為範例，Mix 會產生下列的檔案：

<div class="content-list" markdown="1">
- `public/js/manifest.js`: *The Webpack manifest runtime*
- `public/js/vendor.js`: *Your vendor libraries*
- `public/js/app.js`: *Your application code*
</div>

要避免 JavaScript 發生錯誤，請確保有依正確順序載入這些檔案：

    <script src="/js/manifest.js"></script>
    <script src="/js/vendor.js"></script>
    <script src="/js/app.js"></script>

<a name="react"></a>
### React

Mix 能自動安裝 React 所需的 Babel 插件。請呼叫 `mix.react()` 來取代 `mix.js()`：

    mix.react('resources/assets/js/app.jsx', 'public/js');

Mix 會在後台下載對應版本的 `babel-preset-react` Babel 插件。

<a name="vanilla-js"></a>
### 原生 JS

類似於使用 `mix.styles()` 將樣式表組合，你也可以使用 `scripts()` 方法來組合與壓縮任意數量的 JavaScript 檔案：

    mix.scripts([
        'public/js/admin.js',
        'public/js/dashboard.js'
    ], 'public/js/all.js');

這個選項對於不需要 JavaScript 撰寫 webpack 的舊專案特別好用。

> {tip} `mix.scripts()` 的微小差別是 `mix.babel()`。該方法簽署與 `scripts` 相同。然而，相連的檔案會接收 Babel 編譯，將任何 ES2015 程式碼翻成所有瀏覽器都能理解的原生 JavaScript。

<a name="custom-webpack-configuration"></a>
### 自訂 Webpack 設定

在啟動之前，Laravel Mix 引用前先設定的 `webpack.config.js` 檔案來為你盡快啟動與執行。有時，你可能需要手動修改這個檔案。你可能需要引用一個特殊的載入器或插件，或者你可能比較喜歡使用 Stylus，而不是 Sass。在這種情況下，你有兩種選擇：

#### 合併自訂設定

Mix 提供一個好用的 `webpackConfig` 方法，可以讓你合併任何大小的 Webpack 設定來覆寫原本的設定。這是一個特別吸引人的選擇，因為它根本不需要你複製和維護 `webpack.config.js` 檔案。`webpackConfig` 方可接受一個物件，這物件應該具有任何你希望應用的 [Webpack-specific 設定](https://webpack.js.org/configuration/)。

    mix.webpackConfig({
        resolve: {
            modules: [
                path.resolve(__dirname, 'vendor/laravel/spark/resources/assets/js')
            ]
        }
    });

#### 自訂設定檔

如果你想要編譯自訂 Webpack 設定，複製 `node_modules/laravel-mix/setup/webpack.config.js` 檔案到專案的根目錄。接著，將 `package.json` 檔案中的所有 `--config` 引用到最近複製的設定檔。如果你選擇這種方式自訂，就必須手動將 Mix 的 `webpack.config.js` 上層任何之後的更新合併到你自訂的檔案。

<a name="copying-files-and-directories"></a>
## 複製檔案與目錄

`copy` 方法可被用於複製檔案和目錄到新的地方。這方法有助於你在 `node_modules` 目錄中的特定資源可以被重新放置到你的 `public` 資料夾。

    mix.copy('node_modules/foo/bar.css', 'public/css/bar.css');

在複製目錄的時候，`copy` 方法會將目錄的結構給扁平化。要維持目錄的原始結構，你應該使用 `copyDirectory` 方法：

    mix.copyDirectory('assets/img', 'public/img');

<a name="versioning-and-cache-busting"></a>
## 版控與快取的清除

許多開發者會使用時間戳或唯一的 Token 來後綴他們的資源，為了強制瀏覽器載入資源的更新，而不是舊的程式碼副本。Mix 能使用 `version` 方法來會你處理。

`version` 方法會自動在所有編譯過的檔案的名稱附加一個唯一的雜湊來命名，可以讓你更方便地清除快取：

    mix.js('resources/assets/js/app.js', 'public/js')
       .version();

在產生版控檔案之後，你將不知道確切的檔案名稱。所以你應該[視圖](/laravel_tw/docs/5.5/views)中使用 Laravel 全域的 `mix` 函式來正確的載入被雜湊的資源。`mix` 函式會自動確認雜湊檔案的目前名稱：

    <link rel="stylesheet" href="{% raw %} {{ mix('/css/app.css') }} {% endraw %}">

因為版控檔案通常在開發中並不必要，所以你可以指示只有在執行 `npm run production` 期間執行版控：

    mix.js('resources/assets/js/app.js', 'public/js');

    if (mix.inProduction()) {
        mix.version();
    }

<a name="browsersync-reloading"></a>
## Browsersync 重新載入

[BrowserSync](https://browsersync.io/) 能自動監控檔案的變化，並將該變化注入到瀏覽器，而不需要再重新整理。你可以呼叫 `mix.browserSync()` 方法來啟用這個輔助工具：

    mix.browserSync('my-domain.dev');

    // 或者...

    // https://browsersync.io/docs/options
    mix.browserSync({
        proxy: 'my-domain.dev'
    });

你可以傳入字串（proxy）或物件（BrowserSync 設定）其中一項到這個方法。接著，使用 `npm run watch` 指令來啟動 Webpack 的開發伺服器。現在，當你修改腳本或 PHP 檔案時，瀏覽器會即時重新整理該頁面讓你看到剛剛做的更改。

<a name="environment-variables"></a>
## 環境變數

你可以在你的 `.env` 檔案中加上 `MIX_` 的前綴來將環境變數注入到 Mix：

    MIX_SENTRY_DSN_PUBLIC=http://example.com

在你的 `.env` 檔案定義一個變數之後，你可以透過 `process.env` 物件來存取。如果在你執行 `watch` 任務時需要變更該值，你將需要重新啟動任務：

    process.env.MIX_SENTRY_DSN_PUBLIC

<a name="notifications"></a>
## 通知

有空的時候，Mix 會自動顯示每次封裝的作業系統通知。這會讓你即時得知編譯的成功與否。然而，你可能還是會有想要停用這些東西的時候，像是在正式上線主機上觸發了 Mix。若想要停用通知，可以透過 `disableNotifications` 方法。

    mix.disableNotifications();
