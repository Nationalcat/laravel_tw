---
layout: post
title: hashing
tag: 5.5
---
# 雜湊

- [介紹](#introduction)
- [基本用法](#basic-usage)

<a name="introduction"></a>
## 介紹

Laravel `Hash` [facade](/laravel_tw/docs/5.5/facades) 為儲存使用者密碼而提供安全性 Bcrypt 雜湊化。如果你正使用 Laravel 內建的 `LoginController` 和 `RegisterController` 類別，它們會自動使用 Bcrypt 加密來進行註冊與認證。

> {tip} 對於需要雜湊化的密碼來說，Bcrypt 會是個最佳的選擇，因為它的「加密係數」是可被調整的。這也代表著每次加密的時間可以隨著硬體的升級而再加長。

<a name="basic-usage"></a>
## 基本用法

你可以透過呼叫 `Hash` facade 上的 `make` 方法來雜湊密碼：

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;
    use Illuminate\Support\Facades\Hash;
    use App\Http\Controllers\Controller;

    class UpdatePasswordController extends Controller
    {
        /**
         * 更新使用者的密碼。
         *
         * @param  Request  $request
         * @return Response
         */
        public function update(Request $request)
        {
            // 驗證新密碼的長度...

            $request->user()->fill([
                'password' => Hash::make($request->newPassword)
            ])->save();
        }
    }

`make` 方法還可以讓你使用 `rounds` 選項來管理 bcrypt 雜湊演算法的加密係數。然而，大部分的應用程式只需要用到預設值：

    $hashed = Hash::make('password', [
        'rounds' => 12
    ]);

#### 根據雜湊值驗證密碼

`check` 方法可以讓你去驗證給定的純文字字串是否與給定的雜湊值相符合。然而，如果你是使用 [Laravel 內建的](/laravel_tw/docs/5.5/authentication) `LoginController`，你可能不需要直接使用它，因為這個控制器已經自動呼叫這個方法了：

    if (Hash::check('plain-text', $hashedPassword)) {
        // 該密碼相符於...
    }

#### 檢查密碼是否需要重新雜湊

`needsRehash` 函式可以讓你檢查已雜湊的密碼所使用的加密係數是否有被變動：

    if (Hash::needsRehash($hashed)) {
        $hashed = Hash::make('plain-text');
    }
