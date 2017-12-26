---
layout: post
title: dusk
---
# 瀏覽器測試 (Laravel Dusk)

- [介紹](#introduction)
- [安裝](#installation)
    - [使用其他瀏覽器](#using-other-browsers)
- [入門](#getting-started)
    - [產生測試](#generating-tests)
    - [執行測試](#running-tests)
    - [環境處理](#environment-handling)
    - [建立瀏覽器](#creating-browsers)
    - [認證](#authentication)
- [與元素互動](#interacting-with-elements)
    - [Dusk 選擇器](#dusk-selectors)
    - [點擊連結](#clicking-links)
    - [文字、值與屬性](#text-values-and-attributes)
    - [使用表單](#using-forms)
    - [附加檔案](#attaching-files)
    - [使用鍵盤](#using-the-keyboard)
    - [使用滑鼠](#using-the-mouse)
    - [範圍選擇器](#scoping-selectors)
    - [等待元素](#waiting-for-elements)
    - [Vue 的斷言](#making-vue-assertions)
- [可用的斷言](#available-assertions)
- [頁面](#pages)
    - [產生頁面](#generating-pages)
    - [設定分頁](#configuring-pages)
    - [導航至分頁](#navigating-to-pages)
    - [快捷選擇器](#shorthand-selectors)
    - [頁面方法](#page-methods)
- [元件](#components)
    - [產生元件](#generating-components)
    - [使用元件](#using-components)
- [持續整合](#continuous-integration)
    - [Travis CI](#running-tests-on-travis-ci)
    - [CircleCI](#running-tests-on-circle-ci)
    - [Codeship](#running-tests-on-codeship)

<a name="introduction"></a>
## 介紹

Laravel Dusk 提供了一個可讀性高且易於瀏覽器自動化測試 API。預設的 Dusk 不需要在你的機器上安裝 JDK 或 Selenium。而是使用獨立安裝的 [ChromeDriver](https://sites.google.com/a/chromium.org/chromedriver/home)。然而，你可以自由的運用任何與 Selenium 相容的驅動。

<a name="installation"></a>
## 安裝

開始前，你應該把 `laravel/dusk` 增加到你的專案的 Composer 依賴項目：

    composer require --dev laravel/dusk

一旦安裝了 Dusk，你應該註冊 `Laravel\Dusk\DuskServiceProvider` 給服務提供者通常，這會透過 Laravel 的服務提供者自動完成註冊。

> {note} 如果你正在手動註冊 Dusk 的服務提供者，你千萬別在你的正式環境上註冊它們，因為這麼做可能會導致任意使用者能夠使用你的應用程式進行身份驗證。

安裝 Dusk 套件之後，執行 Artisan 指令的 `dusk:install`：

    php artisan dusk:install

`Browser` 目錄會在裡面建立 `tests` 目錄，並包含一個範例測試。接著，在你的 `.env` 檔案中設定 `APP_URL` 環境變數。這個值應該會你在瀏覽器中存取你的應用程式的 URL 匹配。

使用 Artisan 指令的 `dusk` 來執行你的測試，`dusk` 指令也接受 `phpunit` 指令和任何參數：

    php artisan dusk

<a name="using-other-browsers"></a>
### 使用其他瀏覽器

預設的 Dusk 使用 Google Chrome 和獨立安裝的 [ChromeDriver](https://sites.google.com/a/chromium.org/chromedriver/home) 來執行你的瀏覽器測試。然而，你可以啟動自己的 Selenium 伺服器，並針對你的需要的任何瀏覽器執行測試。

打開你的 `tests/DuskTestCase.php` 檔案，這是應用程式的基本 Dusk 測試範例。在這個檔案中，你可以移除 `startChromeDriver` 方法的回呼。這會禁止 Dusk 自動啟用 ChromeDriver：

    /**
     * 準備執行 Dusk 測試。
     *
     * @beforeClass
     * @return void
     */
    public static function prepare()
    {
        // static::startChromeDriver();
    }

接著，你可以簡單地修改 `driver` 方法來選擇你要連結的 URL 和 port。除此之外，你可以修改本來要傳給 WebDriver 的「期望功能」：

    /**
     * 建立 RemoteWebDriver 實例。
     *
     * @return \Facebook\WebDriver\Remote\RemoteWebDriver
     */
    protected function driver()
    {
        return RemoteWebDriver::create(
            'http://localhost:4444/wd/hub', DesiredCapabilities::phantomjs()
        );
    }

<a name="getting-started"></a>
## 入門

<a name="generating-tests"></a>
### 產生測試

使用 Artisan 指令的 `dusk:make` 來產生 Dusk 測試。剛產生的測試會放置在 `tests/Browser` 目錄中：

    php artisan dusk:make LoginTest

<a name="running-tests"></a>
### 執行測試

使用 Artisan 指令的 `dusk` 來執行你的瀏覽器測試：

    php artisan dusk

`dusk` 指令可以接受 PHPUnit 測試所用的任何參數，像是你也可以只為給定[群組](https://phpunit.de/manual/current/en/appendixes.annotations.html#appendixes.annotations.group)做測試等：

    php artisan dusk --group=foo

#### 手動啟動 ChromeDriver

預設的 Dusk 會自己嘗試啟動 ChromeDriver。如果這不適用於你的系統，你可以在手動啟動 ChromeDriver 之前執行 `dusk` 指令。如果你選擇手動啟動 ChromeDriver ，你應該在 `tests/DuskTestCase.php` 檔案把下列範例的那行註解取消掉：

    /**
     * 準備執行 Dusk 測試。
     *
     * @beforeClass
     * @return void
     */
    public static function prepare()
    {
        // static::startChromeDriver();
    }

此外，如果你在 Port 9515 以外啟動 ChromeDriver，你應該修改一些類別的 `driver` 方法：

    /**
     * 建立 RemoteWebDriver 實例。
     *
     * @return \Facebook\WebDriver\Remote\RemoteWebDriver
     */
    protected function driver()
    {
        return RemoteWebDriver::create(
            'http://localhost:9515', DesiredCapabilities::chrome()
        );
    }

<a name="environment-handling"></a>
### 環境處理

要強制 Dusk 在執行測試時使用自己的環境檔案，在你的專案根目錄建立 `.env.dusk.{environment}` 檔案。例如，如果你會從你的 `local` 環境啟動 `dusk` 指令，你應該建立 `.env.dusk.local` 檔案。

在執行測試的時候，Dusk 會備份你的 `.env` 檔案和重新命名你的 Dusk 環境到 `.env`。一旦完成測試，就會恢復你的 `.env` 檔案。

<a name="creating-browsers"></a>
### 建立瀏覽器

讓我們撰寫一個測試，就從測試我們是否可以登入到自己的應用程式開始吧！產生一個測試之後，我們可以修改成讓它導航到登入頁面，並輸入一些憑證和點擊「登入」的按鈕。呼叫 `browse` 方法來建立一個瀏覽器實例：

    <?php

    namespace Tests\Browser;

    use App\User;
    use Tests\DuskTestCase;
    use Laravel\Dusk\Chrome;
    use Illuminate\Foundation\Testing\DatabaseMigrations;

    class ExampleTest extends DuskTestCase
    {
        use DatabaseMigrations;

        /**
         * 基本的瀏覽器測試範例。
         *
         * @return void
         */
        public function testBasicExample()
        {
            $user = factory(User::class)->create([
                'email' => 'taylor@laravel.com',
            ]);

            $this->browse(function ($browser) use ($user) {
                $browser->visit('/login')
                        ->type('email', $user->email)
                        ->type('password', 'secret')
                        ->press('Login')
                        ->assertPathIs('/home');
            });
        }
    }

正如你在上面範例所看到的，`browse` 方法接收一個回呼。瀏覽器實例會自動被 Dusk 傳入這個回呼，並用於與你應用程式進行交互和斷言的主要對象。

> {tip} 這個測試可以被用於測試使用 Artisan 指令的 `make:auth` 產生的登入畫面。

#### 建立多個瀏覽器

有時你可能需要多個瀏覽器才能執行真正想要的測試。例如，可能需要多個瀏覽器來測試 websocket 雙向作業的聊天視窗。要建立多個瀏覽器，只需要在定給的 `browse` 方法的回呼中「加入」多個瀏覽器：

    $this->browse(function ($first, $second) {
        $first->loginAs(User::find(1))
              ->visit('/home')
              ->waitForText('Message');

        $second->loginAs(User::find(2))
               ->visit('/home')
               ->waitForText('Message')
               ->type('message', 'Hey Taylor')
               ->press('Send');

        $first->waitForText('Hey Taylor')
              ->assertSee('Jeffrey Way');
    });

#### 調整瀏覽器的視窗大小

你可以使用 `resize` 方法來調整瀏覽器視窗大小：

    $browser->resize(1920, 1080);

`maximize` 方法可以被用於最大化瀏覽器視窗：

    $browser->maximize();

<a name="authentication"></a>
### 認證

如果你經常要測試需要認證的頁面，你可以使用 Dusk 的 `loginAs` 方法，以避免每次在測試的時候與登入畫面發生衝突。`loginAs` 方法接受使用者 ID 或使用者模型實例：

    $this->browse(function ($first, $second) {
        $first->loginAs(User::find(1))
              ->visit('/home');
    });

> {note} 使用 `loginAs` 方法之後，使用者的 session 會保留在檔案中的所有測試。

<a name="interacting-with-elements"></a>
## 與元素互動

<a name="dusk-selectors"></a>
### Dusk 選擇器

選擇好的 CSS 選擇器與元素互動是寫 Dusk 測試最艱難的其中一部分。隨著時間的推移，前端的更動可能導致像下面這樣，CSS 選擇器會破壞你的測試：

    // HTML...

    <button>Login</button>

    // Test...

    $browser->click('.login-page .container div > button');

Dusk 選擇器可以讓你專心撰寫有效的測試，而不是死記 CSS 選擇器。可以新增 `dusk` 屬性到你的 HTML 元素來定義選擇器。然後，在 Dusk 測試中的附加元素上使用 `@` 前綴到要操作的選擇器上：

    // HTML...

    <button dusk="login-button">Login</button>

    // Test...

    $browser->click('@login-button');

<a name="clicking-links"></a>
### 點擊連結

你可以在瀏覽器實例上使用 `clickLink` 方法來點擊連結。`clickLink` 方法會點擊給定有顯示文字的連結：

    $browser->clickLink($linkText);

> {note} 這個方法與 jQuery 互相影響。如果 jQuery 在頁面上不能用，Dusk 會自動注入，以便在測試期間使用。

<a name="text-values-and-attributes"></a>
### 文字、值與屬性

#### 取得和設定值

Dusk 提供了幾種與目前顯示文字、值和元素屬性在頁面上互相作用的方法。例如，要拿到與給定選擇器匹配的元素值，可以使用 `value` 方法：

    // 取得值...
    $value = $browser->value('selector');

    // 設定值...
    $browser->value('selector', 'value');

#### 取得文字

`text` 方法可用於取得與給定選擇器匹配的元素所顯示的文字：

    $text = $browser->text('selector');

#### 取得屬性

最後，`attribute` 方法可用於取得與選擇器匹配的元素屬性：

    $attribute = $browser->attribute('selector', 'value');

<a name="using-forms"></a>
### 使用表單

#### 輸入值

Dusk 提供了多種與表單和輸入元素互動的方法。首先，讓我們看一下輸入文字到輸入欄位：

    $browser->type('email', 'taylor@laravel.com');

注意一下，儘管該方法會在必要的時候接受它，我們就不需要在把 CSS 選擇器傳入該類型方法。如果沒有提供 CSS 選擇器，Dusk 會尋找具有給定名稱屬性的輸入欄位。最後，Dusk 會嘗試找到具有給定 `name` 屬性的 `textarea`。

要將文字附加到欄位而不清除它的內容，你可以使用 `append` 方法：

    $browser->type('tags', 'foo')
            ->append('tags', ', bar, baz');

你可以使用 `clear` 方法清除輸入的值：

    $browser->clear('email');

#### 下拉選單

你可以使用 `select` 方法在你的下拉選擇一個值。像是 `type` 方法和 `select` 方法不需要完整的 CSS 選擇器。當傳送值給 `select` 方法時，你應該傳入底層選項值，而不是顯示文字：

    $browser->select('size', 'Large');

你可以省略第二個參數來隨機選擇一個選項：

    $browser->select('size');

#### 複選框

要「勾選」一個複選欄位，你可以使用 `check` 方法。像其他許多輸入相關的方法，其實並不需要完整的 CSS 選擇器。如果找不到準確的選擇器匹配，Dusk 會尋找與 `name` 屬性匹配的複選框：

    $browser->check('terms');

    $browser->uncheck('terms');

#### 單選按鈕

你可以使用 `radio` 方法來「選擇」一個單選按鈕的選項。像是其他許多輸入相關方法，其實並不需要完整的 CSS 選擇器。如果找不到準確的選擇器匹配，Dusk 會尋找與 `name` 和 `value` 屬性匹配的單選按鈕：

    $browser->radio('version', 'php7');

<a name="attaching-files"></a>
### 附加檔案

`attach` 方法可被用於附加一個檔案到 `input` 輸入元素。像是其他許多輸入相關方法，其實並不需要完整的 CSS 選擇器。如果找不到準確的選擇器匹配，Dusk 會尋找與 `name` 屬性匹配的檔案輸入元素：

    $browser->attach('photo', __DIR__.'/photos/me.png');

<a name="using-the-keyboard"></a>
### 使用鍵盤

`keys` 方法可以讓你為給定元素提供比一般 `type` 方法更複雜的輸入順序。例如，你可以修改鍵盤輸入的值。在這個範例中，當 `taylor` 進入與給定選擇器匹配的元素時，`shift` 鍵會被按住。在 `taylor` 輸入完之後，`otwell` 則會正常輸入，不會按下任何按鍵：

    $browser->keys('selector', ['{shift}', 'taylor'], 'otwell');

你甚至可以將「熱鍵」發送到包含你應用程式的主要 CSS 選擇器：

    $browser->keys('.app', ['{command}', 'j']);

> {tip} 所有修改按鍵都放在 `{}` 裡面，並與 `Facebook\WebDriver\WebDriverKeys` 類別中的定義常數匹配，這些常數可以在 [GitHub 上找到](https://github.com/facebook/php-webdriver/blob/community/lib/WebDriverKeys.php)。

<a name="using-the-mouse"></a>
### 使用滑鼠

#### 點擊元素

`click` 方法可被用於「點擊」與給定選擇器匹配的元素：

    $browser->click('.selector');

#### 滑鼠移動觸發

`mouseover` 方法可被用於當你需要移動滑鼠移到給定選擇器的元素上：

    $browser->mouseover('.selector');

#### 滑鼠的拖曳與放開

`drag` 方法可被用於拖曳與給定選擇器匹配的元素到另一個元素上：

    $browser->drag('.from-selector', '.to-selector');

或者，你可以在同一方向上拖曳元素：

    $browser->dragLeft('.selector', 10);
    $browser->dragRight('.selector', 10);
    $browser->dragUp('.selector', 10);
    $browser->dragDown('.selector', 10);

<a name="scoping-selectors"></a>
### 範圍選擇器

有時你可能希望執行給定選擇器範圍內的所有操作。例如，你可能希望斷言某些僅存於一個表格中的文字，然後點擊該表單中的一個按鈕。你可以使用 `with` 方法來達成目的。在給定的 `with` 方法回呼中，將執行原始選擇器範圍中的所有操作：

    $browser->with('.table', function ($table) {
        $table->assertSee('Hello World')
              ->clickLink('Delete');
    });

<a name="waiting-for-elements"></a>
### 等待元素

當廣泛使用 JavaScript 大量測試應用程式時，通常在進行測時前，必須「等待」載入一些元素和資料。Dusk 將這個問題弄的很簡單。使用各種方法時，你可以等待元素在頁面上呈現，等待給定的 JavaScript 表達式計算結果為 `true`。

#### 等待

如果你需要為暫停給定的毫秒數時間，請使用 `pause` 方法：

    $browser->pause(1000);

#### 等待選擇器

`waitFor` 方法可被用於暫停執行測試，直到頁面上與給定 CSS 選擇器匹配的元素顯示為止。預設會在拋出異常之前，最多暫停五秒。如果有必要，你可以傳入一個自訂的超時門檻作為方法的第二個參數：

    // 最多等待選擇器五秒鐘...
    $browser->waitFor('.selector');

    // 最多等待選擇器一秒鐘...
    $browser->waitFor('.selector', 1);

你也可以等待給定的選擇器直到頁面中遺失。

    $browser->waitUntilMissing('.selector');

    $browser->waitUntilMissing('.selector', 1);

#### 可用的範圍選擇器

有時你可能希望等待給定的選擇器，然後與選擇器匹配的元素互相交互。例如，你可能希望等待模型視窗，然後案下模型中的「確定」按鈕。在這種情況下，可以使用 `whenAvailable` 方法。在給定的回呼中，只會執行原始選擇器範圍內的所有元素操作：

    $browser->whenAvailable('.modal', function ($modal) {
        $modal->assertSee('Hello World')
              ->press('OK');
    });

#### 等待文字

`waitForText` 方法可被用於等待給定的文字被顯示在頁面上：

    // 最多等待文字五秒鐘...
    $browser->waitForText('Hello World');

    // 最多等待文字一秒鐘...
    $browser->waitForText('Hello World', 1);

#### 等待連結

`waitForLink` 方法可被用於等待給定的連結文字被顯示在頁面上：

    // 最多等待連結五秒鐘...
    $browser->waitForLink('Create');

    // 最多等待連結一秒鐘...
    $browser->waitForLink('Create', 1);

#### 在頁面位置上等待

當要為一個像是 `$browser->assertPathIs('/home')` 路徑斷言時，如果 `window.location.pathname` 無法同步更新，斷言就可能會失敗。你可以使用 `waitForLocation` 方法來等待該頁面回拿到的值：

    $browser->waitForLocation('/secret');

#### 等待頁面重新載入

如果你需要在頁面重新載入後才開始斷言，可以使用 `waitForReload` 方法：

    $browser->click('.some-action')
            ->waitForReload()
            ->assertSee('something');

#### 在 JavaScript 表達式上等待

有時你可能希望暫停執行一個測試，一直到一個給定的 JavaScript 表達式的計算結果為 `true`。你可以使用 `waitUntil` 方法來輕鬆完成此操作。將表達式傳入給此方法時，不需要使用 `return` 關鍵字或結尾用的分號：

    // 最多等待五秒鐘，表達式結果為 true...
    $browser->waitUntil('App.dataLoaded');

    $browser->waitUntil('App.data.servers.length > 0');

    // 最多等待一秒鐘，表達式結果為 true...
    $browser->waitUntil('App.data.servers.length > 0', 1);

#### 等待與回呼

Dusk 有許多「等待」方法依賴於底層的 `waitUsing` 方法。你可以使用這個方法來等待給定的回呼回傳 `true`。`waitUsing` 方法接受等待的最大秒數值、計算閉包的間隔時間、閉包和一個可選的失敗訊息：

    $browser->waitUsing(10, 1, function () use ($something) {
        return $something->isReady();
    }, "Something wasn't ready in time.");

<a name="making-vue-assertions"></a>
### Vue 的斷言

Dusk 甚至可以讓你斷言 [Vue](https://vuejs.org) 的元件資料狀態。例如，想像你的應用程式有以下 Vue 元件：

    // HTML...

    <profile dusk="profile-component"></profile>

    // Component Definition...

    Vue.component('profile', {
        template: '<div>{% raw %} {{ user.name }} {% endraw %}</div>',

        data: function () {
            return {
                user: {
                  name: 'Taylor'
                }
            };
        }
    });

你可以在 Vue 元件的狀態上像是這樣斷言：

    /**
     * 一個基礎的 Vue 測試範例。
     *
     * @return void
     */
    public function testVue()
    {
        $this->browse(function (Browser $browser) {
            $browser->visit('/')
                    ->assertVue('user.name', 'Taylor', '@profile-component');
        });
    }

<a name="available-assertions"></a>
## 可用的斷言

Dusk 為你的應用程式提供了各式各樣的斷言。所有可用的斷言都被記錄在下表中：

斷言  | 說明
------------- | -------------
`$browser->assertTitle($title)`  |  斷言頁面標題是否與給定文字匹配。
`$browser->assertTitleContains($title)`  |  斷言頁面標題是否包含給定的文字。
`$browser->assertPathBeginsWith($path)`  |  斷言目前的 URL 路徑是否開始於給定的路徑。
`$browser->assertPathIs('/home')`  |  斷言目前路徑是否與給定路徑匹配。
`$browser->assertPathIsNot('/home')`  |  斷言目前路徑是否不與給定路徑匹配。
`$browser->assertRouteIs($name, $parameters)`  |  斷言目前 URL 是否與給定名稱路由的 URL 匹配。
`$browser->assertQueryStringHas($name, $value)`  |  斷言給定查詢字串參數是否存在給定的值。
`$browser->assertQueryStringMissing($name)`  |  斷言給定字串參數使否遺失。
`$browser->assertHasQueryStringParameter($name)`  |  斷言給定字串參數是否存在。
`$browser->assertHasCookie($name)`  |  斷言給定 Cookie 是否存在。
`$browser->assertCookieMissing($name)`  |  斷言給定 Cookie 是否不存在。
`$browser->assertCookieValue($name, $value)`  |  斷言 Cookie 是否有給定的值。
`$browser->assertPlainCookieValue($name, $value)`  |  斷言未加密的 Cookie 是否有給定的值。
`$browser->assertSee($text)`  |  斷言給定的文字是否在頁面上存在。
`$browser->assertDontSee($text)`  |  斷言給定的文字是否在頁面上不存在。
`$browser->assertSeeIn($selector, $text)`  |  斷言給定的文字是否在選擇器中存在。
`$browser->assertDontSeeIn($selector, $text)`  |  斷言給定的文字是否在選擇器中不存在。
`$browser->assertSourceHas($code)`  |  斷言給定的文字是否在選擇器中不存在。
`$browser->assertSourceMissing($code)`  |  斷言給定原始碼是否在頁面上不存在。
`$browser->assertSeeLink($linkText)`  |  斷言給定連結是否在頁面上存在。
`$browser->assertDontSeeLink($linkText)`  |  斷言給定連結是否在頁面上不存在。
`$browser->assertInputValue($field, $value)`  |  斷言給定輸入字斷是否有給定的值。
`$browser->assertInputValueIsNot($field, $value)`  |  斷言給定輸入字段是否沒有給定的值。
`$browser->assertChecked($field)`  |  斷言給定複選框是否被勾選。
`$browser->assertNotChecked($field)`  |  斷言給定複選框是否沒被勾選。
`$browser->assertRadioSelected($field, $value)`  |  斷言給定單選題是否被選擇。
`$browser->assertRadioNotSelected($field, $value)` |  斷言給定單選題是否沒被選擇。
`$browser->assertSelected($field, $value)`  |  斷言給定下拉選單是否有選擇給定的值。
`$browser->assertNotSelected($field, $value)`  |  斷言給定下拉選單是否沒有選擇給定的值。
`$browser->assertSelectHasOptions($field, $values)`  |  斷言給定的陣列值是否可被選擇。
`$browser->assertSelectMissingOptions($field, $values)`  |  斷言給定的陣列值是否不可被選擇斷言。
`$browser->assertSelectHasOption($field, $value)`  |  斷言給定的值是否可被給定的字段選擇。
`$browser->assertValue($selector, $value)`  |  斷言與給定選擇器匹配的元素是否有給定的值。
`$browser->assertVisible($selector)`  |  斷言與給定選擇器匹配的元素是否可見。
`$browser->assertMissing($selector)`  |  斷言與給定選擇器匹配的元素是否不可見。
`$browser->assertDialogOpened($message)`  |  斷言有給定訊息的 JavaScript 對話框是否被打開。
`$browser->assertVue($property, $value, $component)`  |  斷言給定的 Vue 元件資料屬性是否與給定的值匹配。
`$browser->assertVueIsNot($property, $value, $component)`  |  斷言給定的 Vue 元件資料屬性是否與給定的值不匹配。

<a name="pages"></a>
## Page

有時測試需要依序執行幾個複雜的操作。這會使你的測試更難閱讀與理解。Page 可以讓你定義表達式操作，然後可以使用單一方法在給定頁面上執行。Page 還可以讓你為應用程式或單一頁面定義共用選擇器的最快方式。

<a name="generating-pages"></a>
### 產生 Page

可以使用 Artisan 指令的 `dusk:page` 來產生 Page 物件。所有 Page 物件會被放置在 `tests/Browser/Pages` 目錄中：

    php artisan dusk:page Login

<a name="configuring-pages"></a>
### 設定 Page

預設 Page 會有三個方法：`url`、`assert` 和 `elements`。我們現在開始討論 `url` 和 `assert` 方法。`elements` 方法會[在下面更詳細地討論](#shorthand-selectors).

#### `url` 方法

`url` 方法應該回傳表示頁面的 URL，導航到瀏覽器頁面時，Dusk 會使用此 URL：

    /**
     * 為頁面取得 URL。
     *
     * @return string
     */
    public function url()
    {
        return '/login';
    }

#### `assert` 方法

`assert` 方法可以做出任何必要的斷言來驗證瀏覽器是否在給定的頁面上，所以沒必要完成這個方法。然而，如果你希望，你可以自由地做出這些斷言。導航到頁面時，這些斷言會自動執行：

    /**
     * 斷言瀏覽器是否在此頁面
     *
     * @return void
     */
    public function assert(Browser $browser)
    {
        $browser->assertPathIs($this->url());
    }

<a name="navigating-to-pages"></a>
### 導航至分頁

一旦頁面被設定，你可以使用 `visit` 方法導航到該頁面：

    use Tests\Browser\Pages\Login;

    $browser->visit(new Login);

有時你可能已經在給定的頁面上，需要將頁面的選擇器和方法「引入」到當前的測試環境中。這常見於按下按鈕並重導到給定的頁面而沒有明確的導航時。在這種情況下，你可以使用 `on` 方法來載入頁面：

    use Tests\Browser\Pages\CreatePlaylist;

    $browser->visit('/dashboard')
            ->clickLink('Create Playlist')
            ->on(new CreatePlaylist)
            ->assertSee('@create');

<a name="shorthand-selectors"></a>
### 快捷選擇器

Page 的 `elements` 方法，可以讓你為頁面上的任何 CSS 選擇器定義既快速又好記的快捷方式。例如，讓我們為應用程式登入頁面的「email」輸入字段定義一個快捷方式：

    /**
     * 取得該頁面元素的快捷方式。
     *
     * @return array
     */
    public function elements()
    {
        return [
            '@email' => 'input[name=email]',
        ];
    }

現在，你可以在任何地方使用這個快捷選擇器來使用完整的 CSS 選擇器：

    $browser->type('@email', 'taylor@laravel.com');

#### 全域快捷選擇器

安裝 Dusk 之後，基本的 `Page` 類別會放置在 `tests/Browser/Pages` 目錄中。這個類別包含一個 `siteElements` 方法，可被用來定義應用程式中每個 Page 都可以用的全域快捷選擇器：

    /**
     * 取得該網站的全域元素快捷方式。
     *
     * @return array
     */
    public static function siteElements()
    {
        return [
            '@element' => '#selector',
        ];
    }

<a name="page-methods"></a>
### Page 方法

除了在 Page 上定義的預設方法外，你可以定義在整個測試中會用到的額外方法。例如，讓我們想像我們正在建構一個音樂管理應用程式。應用程式的第一個頁常見的頁面操作可能會是建立一個播放清單。但千萬別為每個測試重新編寫建立播放清單的邏輯，你可以在 Page 類別上定義 `createPlaylist` 方法：

    <?php

    namespace Tests\Browser\Pages;

    use Laravel\Dusk\Browser;

    class Dashboard extends Page
    {
        // 其他 page 方法...

        /**
         * 建立新播放清單。
         *
         * @param  \Laravel\Dusk\Browser  $browser
         * @param  string  $name
         * @return void
         */
        public function createPlaylist(Browser $browser, $name)
        {
            $browser->type('name', $name)
                    ->check('share')
                    ->press('Create Playlist');
        }
    }

如果已經定義好該方法，你就可以在任何使用該 Page 的測試中使用它。瀏覽器實例會自動傳入該 Page 的方法：

    use Tests\Browser\Pages\Dashboard;

    $browser->visit(new Dashboard)
            ->createPlaylist('My Playlist')
            ->assertSee('My Playlist');

<a name="components"></a>
## 元件

元件類似 Dusk 的「頁面物件」，但能讓整個應用程式的 UI 和 功能可被重複使用，像是導航列或通知視窗。因此，元件並不會綁定到特定的 URL。

<a name="generating-components"></a>
### 產生元件

可以使用 Artisan 指令的 `dusk:component` 來產生元件。新元件被放置在 `test/Browser/Components` 目錄中：

    php artisan dusk:component DatePicker

如上所示，「日期選擇器」是可能存在於整個應用程式中各種頁面上的組件的範例。在整個測試套件中，手動撰寫數十個瀏覽器自動化測試邏輯來選擇日期，這顯得繁瑣。所以，我們可以定義一組 Dusk 元件來表示日期選擇器，從而讓我們將該邏輯封裝在元件中：

    <?php

    namespace Tests\Browser\Components;

    use Laravel\Dusk\Browser;
    use Laravel\Dusk\Component as BaseComponent;

    class DatePicker extends BaseComponent
    {
        /**
         * 取得元件的根目錄選擇器。
         *
         * @return string
         */
        public function selector()
        {
            return '.date-picker';
        }

        /**
         * 斷言瀏覽器頁面包含的元件。
         *
         * @param  Browser  $browser
         * @return void
         */
        public function assert(Browser $browser)
        {
            $browser->assertVisible($this->selector());
        }

        /**
         * 取得元件的元素快捷方式。
         *
         * @return array
         */
        public function elements()
        {
            return [
                '@date-field' => 'input.datepicker-input',
                '@month-list' => 'div > div.datepicker-months',
                '@day-list' => 'div > div.datepicker-days',
            ];
        }

        /**
         * 選擇給定的日期。
         *
         * @param  \Laravel\Dusk\Browser  $browser
         * @param  int  $month
         * @param  int  $year
         * @return void
         */
        public function selectDate($browser, $month, $year)
        {
            $browser->click('@date-field')
                    ->within('@month-list', function ($browser) use ($month) {
                        $browser->click($month);
                    })
                    ->within('@day-list', function ($browser) use ($day) {
                        $browser->click($day);
                    });
        }
    }

<a name="using-components"></a>
### 使用元件

一旦元件被定義了，我們就可以從任何測試中輕易地在日期選擇器中選擇一個日期。而且，如果選擇日期所需要的邏輯要更改，我們只需要更新元件：

    <?php

    namespace Tests\Browser;

    use Tests\DuskTestCase;
    use Laravel\Dusk\Browser;
    use Tests\Browser\Components\DatePicker;
    use Illuminate\Foundation\Testing\DatabaseMigrations;

    class ExampleTest extends DuskTestCase
    {
        /**
         * 一個基本元件測試範例。
         *
         * @return void
         */
        public function testBasicExample()
        {
            $this->browse(function (Browser $browser) {
                $browser->visit('/')
                        ->within(new DatePicker, function ($browser) {
                            $browser->selectDate(1, 2018);
                        })
                        ->assertSee('January');
            });
        }
    }

<a name="continuous-integration"></a>
## 持續整合

<a name="running-tests-on-travis-ci"></a>
### Travis CI

要在 Travis CI 上執行 Dusk 測試，我們會需要使用到 「sudo-enabled」 Ubuntu 14.04 (Trusty) 環境。因為 Travis CI 不是圖形環境，我們會需要採用一些額外的步驟來啟動 Chrome 瀏覽器。另外，我們會使用 `php artisan serve` 來啟動 PHP 內建的網頁伺服器：

    sudo: required
    dist: trusty

    addons:
       chrome: stable

    install:
       - cp .env.testing .env
       - travis_retry composer install --no-interaction --prefer-dist --no-suggest

    before_script:
       - google-chrome-stable --headless --disable-gpu --remote-debugging-port=9222 http://localhost &
       - php artisan serve &

    script:
       - php artisan dusk

<a name="running-tests-on-circle-ci"></a>
### CircleCI

#### CircleCI 1.0

如果你使用 CircleCI 1.0 來執行 Dusk 測試，你可以使用這個設定檔作為開始。像是 TravisCI，我們會使用 `php artisan serve` 來啟動 PHP 內建的網頁伺服器：

	dependencies:
	  pre:
	      - curl -L -o google-chrome.deb https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
	      - sudo dpkg -i google-chrome.deb
	      - sudo sed -i 's|HERE/chrome\"|HERE/chrome\" --disable-setuid-sandbox|g' /opt/google/chrome/google-chrome
	      - rm google-chrome.deb

    test:
        pre:
            - "./vendor/laravel/dusk/bin/chromedriver-linux":
                background: true
            - cp .env.testing .env
            - "php artisan serve":
                background: true

        override:
            - php artisan dusk

 #### CircleCI 2.0

 如果你使用 CircleCI 2.0 來執行 Dusk 測試，你可以新增這些步驟來建構：

     version: 2
     jobs:
         build:
             steps:
                - run: sudo apt-get install -y libsqlite3-dev
                - run: cp .env.testing .env
                - run: composer install -n --ignore-platform-reqs
                - run: npm install
                - run: npm run production
                - run: vendor/bin/phpunit

                - run:
                   name: Start Chrome Driver
                   command: ./vendor/laravel/dusk/bin/chromedriver-linux
                   background: true

                - run:
                   name: Run Laravel Server
                   command: php artisan serve
                   background: true

                - run:
                   name: Run Laravel Dusk Tests
                   command: php artisan dusk

<a name="running-tests-on-codeship"></a>
### Codeship

在 [Codeship](https://codeship.com) 上執行 Dusk 測試，增加以下指令到你的 Codeship 專案。當然，這些指令只是一個開始，你可以根據實際需要來新增其他指令：

    phpenv local 7.1
    cp .env.testing .env
    composer install --no-interaction
    nohup bash -c "./vendor/laravel/dusk/bin/chromedriver-linux 2>&1 &"
    nohup bash -c "php artisan serve 2>&1 &" && sleep 5
    php artisan dusk
