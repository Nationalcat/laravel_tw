---
layout: post
title: contributions
---
# 貢獻導引

- [Bug 回報](#bug-reports)
- [核心開發討論](#core-development-discussion)
- [提交分支的注意事項](#which-branch)
- [安全性漏洞](#security-vulnerabilities)
- [程式碼風格](#coding-style)
    - [PHPDoc](#phpdoc)
    - [StyleCI](#styleci)

<a name="bug-reports"></a>
## Bug 回報

為了鼓勵合作，Laravel 強烈的歡迎你提交 Pull request，而不只是只有回報 Bug。「Bug 回報」也可以是包含一個失敗測試的 Pull request。

然而，如果你要提出 bug 回報，你的 issue 應該要有一個標題以及清楚的問題描述。也盡可能的提供相關資訊和示範該問題的程式碼。Bug 回報的目的是為了使你和其他人能夠輕易的理解 Bug 的原因並且修復它。

請記得，Bug 回報這件事本身是希望遇到相同問題的其他人也能夠參與解決問題。請不要期待會有人突然跳出來修復這個 Bug 回報。建立 Bug 回報只用來協助你和其他人開始能共同解決問題的途徑。

Laravel 的原始碼就放在 GitHub 上，並且每個 Laravel 延伸套件都有各自的儲存庫：

<div class="content-list" markdown="1">
- [Laravel Application](https://github.com/laravel/laravel)
- [Laravel Art](https://github.com/laravel/art)
- [Laravel Documentation](https://github.com/laravel/docs)
- [Laravel Cashier](https://github.com/laravel/cashier)
- [Laravel Cashier for Braintree](https://github.com/laravel/cashier-braintree)
- [Laravel Envoy](https://github.com/laravel/envoy)
- [Laravel Framework](https://github.com/laravel/framework)
- [Laravel Homestead](https://github.com/laravel/homestead)
- [Laravel Homestead Build Scripts](https://github.com/laravel/settler)
- [Laravel Horizon](https://github.com/laravel/horizon)
- [Laravel Passport](https://github.com/laravel/passport)
- [Laravel Scout](https://github.com/laravel/scout)
- [Laravel Socialite](https://github.com/laravel/socialite)
- [Laravel Website](https://github.com/laravel/laravel.com)
</div>

<a name="core-development-discussion"></a>
## 核心開發討論

你可以在 Laravel 的 [issue 看板](https://github.com/laravel/internals/issues)上提出新的功能或改善現有的 Laravel 特性。如果提出一個新功能，請至少實作一些要完成該功能所需要的程式碼。

有關於 Bug、新功能和現有功能的實作的非正式討論，都請到 Slack [LaraChat](https://larachat.co) 的 `#internals` 頻道。Laravel 創始人 Taylor Otwell，通常會在平日的早上八點到下午五點出沒（太平洋時間-06:00 或 美國/芝加哥），其他時間則不一定會出現。

<a name="which-branch"></a>
## 提交分支的注意事項

**全部的** Bug 修復應該發送到最新的穩定分支或當前 LTS 分支（v5.5）。Bug 修復**不應該**直接發送到 `master` 分支，除非它們是修復只在即將發佈的功能。

能**完全向後相容**於當前 Laravel 版本的**次要**功能，可以提交到最新的穩定版分支。

**主要**新功能應該始終提交到 `master` 分支，其中包含即將發佈的 Laravel 版本。

如果你不清楚你的功能屬於主要或非主要的，請到 Slack [LaraChat](https://larachat.co) `#internals` 頻中詢問 Taylor Otwell。

<a name="security-vulnerabilities"></a>
## 安全性漏洞

如果你發現 Laravel 存在著安全性漏洞，請傳送郵件到 Taylor Otwell 的 <a href="mailto:taylor@laravel.com">taylor@laravel.com</a> 信箱。所有安全漏洞將會被盡快解決。

<a name="coding-style"></a>
## 程式碼風格

Laravel 遵循 [PSR-2](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-2-coding-style-guide.md) 編碼標準和 [PSR-4](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-4-autoloader.md) 自動載入標準。

<a name="phpdoc"></a>
### PHPDoc

以下區塊是 Laravel 有效的文件示範。請注意，`@param` 屬性後面需要空兩個空格，再來接上參數類型，再兩個空格，最後才是變數名稱：

    /**
     * 註冊與容器的綁定。
     *
     * @param  string|array  $abstract
     * @param  \Closure|string|null  $concrete
     * @param  bool  $shared
     * @return void
     */
    public function bind($abstract, $concrete = null, $shared = false)
    {
        //
    }

<a name="styleci"></a>
### StyleCI

如果你的程式碼風格並不完美，也別太擔心。在 Pull Request 被合併後，[StyleCI](https://styleci.io/) 將會自動修正程式碼格式到 Laravel 的儲存庫。這讓我們能更專注於貢獻程式碼，而非程式碼風格。
