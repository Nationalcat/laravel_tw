---
layout: post
title: contributing
tag: 4.0
---
# 為 Laravel 貢獻

- [介紹](#introduction)
- [請求合併](#pull-requests)
- [開發準則](#coding-guidelines)

<a name="introduction"></a>
## 介紹

Laravel 是免費的開源軟體，意味著任何人都可以對 Laravel 的發展與進步做出貢獻，Laravel 原始碼託管在[Github](https://github.com/laravel)， Github提供一個簡單的方法去開發專案分支(fork)及合併你對專案的貢獻。

<a name="pull-requests"></a>
## 請求合併

Laravel 的新功能(New Features)及臭蟲回報(Bugs)的 Pull 請求的過程中並不相同，在發出新功能(New Features)的 Pull 請求，你應該先建立一個有 `[Proposal]` 為標題的議題，這個提議議題中，必須描述新功能的特徵，以及實作的想法，則該提議的議題將被審查是否通過或拒絕，一旦通過審查， Pull 請求將會被實作在新的功能中，若 Pull請求沒有遵循這樣的準則，則此 Pull請求的議題將會被立即關閉。

在發出臭蟲回報(Bugs)的 Pull 請求或許不會建立任何提議的議題，如果你知道已經提交到github檔案中，任何臭蟲的解決方案，請留言詳細說明您建議修復的細節資訊給我們。

文件的增加與修正也可以透過 Github 上的 [文件源](https://github.com/laravel/docs) 來進行編修。

### 功能請求(Feature Requests)

如果你有想要加入新功能到 Laravel 的點子的話，你或許可以在 Github 建立一個有 `[Request]` 為標題的議題，這個 Laravel 核心的貢獻成員將會審查您提出的功能請求。

<a name="coding-guidelines"></a>
## 開發準則

Laravel 遵循 [PSR-0](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-0.md) 及 [PSR-1](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-1-basic-coding-standard.md) 的程式碼標準規範，除了這些標準，下面的列表是其他應被遵循的程式碼標準:

- 命名空間 (Namespace) 的定義應該與 `<?php` 在同一行。
- 類別 (Class) 的起始大括號 `{` 應該和類別名稱(Class Name)在同一行。
- 函式 (Function) 及控制結構 (control structure) 的起始大括號 `{` 應該在不同行中呈現
- 介面 (Interface) 名稱的後綴字必須要有 `Interface` (`FooInterface`)
