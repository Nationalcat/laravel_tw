---
layout: post
title: localization
tag: 5.5
---
# 本地化

- [簡介](#introduction)
- [定義翻譯字串](#defining-translation-strings)
    - [使用短鍵值](#using-short-keys)
    - [以翻譯字串為鍵值](#using-translation-strings-as-keys)
- [取出翻譯字串](#retrieving-translation-strings)
    - [在翻譯字串中取代參數](#replacing-parameters-in-translation-strings)
    - [複數化](#pluralization)
- [覆寫套件的語言檔](#overriding-package-language-files)

<a name="introduction"></a>
## 簡介

Laravel 的本地化功能提供方便的方法來取得多語系的字串，讓你的網站可以簡單的支援多語系。語系字串存放在 `resources/lang` 資料夾的檔案裡。在此資料夾內，每個網站支援的語系都會對應到一個子目錄：

    /resources
        /lang
            /en
                messages.php
            /es
                messages.php

語系檔會回傳鍵值為字串的陣列。例如：

    <?php

    return [
        'welcome' => 'Welcome to our application'
    ];

### 切換語系

網站的預設語系儲存在 `config/app.php` 設定檔中。你隨時可以根據應用程式需求來修改這個值。也可以用 `App` facade 的 `setLocale` 方法來切換現行語系：

    Route::get('welcome/{locale}', function ($locale) {
        App::setLocale($locale);

        //
    });

你也可以設定「備用語系」，會在現行語言沒有給定的翻譯自傳時使用。如同預設語系，備用語系也可以在 `config/app.php` 設定檔中設定：

    'fallback_locale' => 'en',

#### 判斷當前語系

你可以使用 `App` facade 的 `getLocale` 和 `isLocal` 方法來判斷當前語系或檢查語系是否為給定值：

    $locale = App::getLocale();

    if (App::isLocale('en')) {
        //
    }

<a name="defining-translation-strings"></a>
## 定義翻譯字串

<a name="using-short-keys"></a>
### 使用短鍵值

翻譯字串通常儲存在 `resources/lang` 資料夾的檔案中。在此資料夾內，每個網站支援的語系都會對應到一個子目錄：

    /resources
        /lang
            /en
                messages.php
            /zh-TW
                messages.php

語系檔會回傳鍵值為字串的陣列。例如：

    <?php

    // resources/lang/en/messages.php

    return [
        'welcome' => 'Welcome to our application'
    ];

<a name="using-translation-strings-as-keys"></a>
### 以翻譯字串為鍵值

對有大量翻譯需求的網站來說，為每個字串定義一個「短鍵值」很快就會在視圖中使用時令人困惑。因此，Laravel 也支援以「預設」語系的字串來作為定義翻譯字串的鍵值。

以翻譯字串為鍵值的語系檔存放在`resources/land`的 JSON 檔案中。例如，如果網站想支援中文，應該要建立一個 `resources/lang/zh-TW.json` 檔案：

    {
        "I love programming.": "我愛寫程式。"
    }

<a name="retrieving-translation-strings"></a>
## 取出翻譯字串

你可以使用`__`輔助函式來從語系檔中取出內容。`__`方法第一個參數是檔案跟翻譯字串的鍵值。例如，從 `resources/lang/messages.php` 語系檔中取出 `welcome` 翻譯字串：

    echo __('messages.welcome');

    echo __('I love programming.');

如果你是使用 [Blade 模板引擎](/laravel_tw/docs/5.5/blade)，可以用`{% raw %} {{ }} {% endraw %}`語法或 `@land` 指令來回傳翻譯字串：

    {% raw %} {{ __('messages.welcome') }} {% endraw %}

    @lang('messages.welcome')

如果指定的翻譯字串不存在，`__`方法只會回傳翻譯字串鍵值。所以在使用上面的範例，`__`方法在翻譯字串不存在時翻譯會回傳 `messages.welcome`。

<a name="replacing-parameters-in-translation-strings"></a>
### 在翻譯字串中取代參數

你也可以在翻譯字串中定義佔位符。所有佔位符都要以`:`前綴。例如，可以定義一個有佔位名稱的歡迎訊息：

    'welcome' => 'Welcome, :name',

要取代翻譯字串中的佔位符，需把替代品的陣列傳給`__`方法的第二個參數：

    echo __('messages.welcome', ['name' => 'dayle']);

如果佔位符全部都是大寫，或只有第一個字母大寫，被翻譯出的值也會變成對應的大寫：

    'welcome' => 'Welcome, :NAME', // Welcome, DAYLE
    'goodbye' => 'Goodbye, :Name', // Goodbye, Dayle

<a name="pluralization"></a>
### 複數化

複數化是個複雜的問題，不同語言對於複數化有不同的規則。使用「|」字元，可以區分單複數字串格式：

    'apples' => 'There is one apple|There are many apples',

甚至可以建立更複雜的複數化規則給不同數字範圍的翻譯字串：

    'apples' => '{0} There are none|[1,19] There are some|[20,*] There are many',

定義好有複數化選項的翻譯字串後，便可以用 `trans_choice` 方法來取得指定「數量」的內容。在這個範例中，因為數量大於一，會複數格式的翻譯字串：

    echo trans_choice('messages.apples', 10);

你也可以在多個設定中定義佔位符屬性。這些佔位符可以透過傳入一組陣列作為第三個參數到 `trans_choice` 函式中替換內容：

    'minutes_ago' => '{1} :value minute ago|[2,*] :value minutes ago',

    echo trans_choice('time.minutes_ago', 5, ['value' => 5]);

<a name="overriding-package-language-files"></a>
## 覆寫套件的語言檔

部分套件帶有自己的語系檔，可以藉由放置檔案在 `resources/lang/vendor/{package}/{locale}` 來複寫它們，而不是直接修改套件的核心檔案。

例如，如果需要覆寫 `skyrim/hearthfire` 套件的英文語系檔 `messages.php`，您應該把檔案放置在： `resources/lang/vendor/hearthfire/en/messages.php`。在這個檔案中，只要去定義需要覆寫的翻譯字串，任何沒有覆寫的翻譯字串仍會從套件的語言檔載入。
