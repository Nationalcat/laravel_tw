---
layout: post
title: artisan
tag: 4.0
---
# Artisan CLI

- [介紹](#introduction)
- [使用](#usage)

<a name="introduction"></a>
## 介紹

Artisan 是 Laravel 命令列的介面名稱，提供數個有用的指令，讓你方便開發應用程式，他是由強大的 Symfony Console 元素所驅動。

<a name="usage"></a>
## 使用

你可以使用 `list` 命令去檢視所有可用的 Artisan 指令:

**列出所有可用的指令**

	php artisan list

每個指令都包含了 "help" 指令，可以顯示且描述指令可用的參數及選項，為了檢視 "幫助" 畫面，只要在指令名稱前加上 `help` 即可:

**檢視指令的 "幫助" 畫面**

	php artisan help migrate

你可以在執行指令時使用 `--env` 參數，指定設定的環境:

**指定設定的環境**

	php artisan migrate --env=local

你也可以使用 `--version` 選項，檢視目前 Laravel 安裝的版本:

**檢視目前你使用的 Laravel 版本**

	php artisan --version
