---
layout: post
title: encryption
tag: 5.5
---
# 加密

- [介紹](#introduction)
- [設定](#configuration)
- [使用加密](#using-the-encrypter)

<a name="introduction"></a>
## 介紹

Laravel 的加密器是使用 OpenSSL 來提供 AES-256 和 AES-128。強烈建議使用 Laravel 內建的加密功能，而非自己「徒手打造」加密演算法。所有 Laravel 加密值都會用到訊息認證碼（MAC），以便原始值在加密後不被再次修改。

<a name="configuration"></a>
## 設定

在使用 Laravel 加密器之前，請務必在 `config/app.php` 設定檔設定 `key` 選項。你應該使用 `php artisan key:generate` 指令來產生這個金鑰，因為這個 Artisan 指令會使用 PHP 的安全隨機字元產生器來產生金鑰。如果這個值還未設定，所有 Laravel 加密的值都不會是安全的。

<a name="using-the-encrypter"></a>
## 使用加密

#### 加密一個值

你可以使用 `encrypt` 輔助函式來加密一個值。所有被加密的值都會使用 OpenSSL 和 `AES-256-CBC` 來加密。此外，所有加密的值都會使用訊息認證碼（MAC）來進行簽證，並用來檢測對加密字串的任何修改：、

    <?php

    namespace App\Http\Controllers;

    use App\User;
    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;

    class UserController extends Controller
    {
        /**
         * 為使用者儲存私人訊息。
         *
         * @param  Request  $request
         * @param  int  $id
         * @return Response
         */
        public function storeSecret(Request $request, $id)
        {
            $user = User::findOrFail($id);

            $user->fill([
                'secret' => encrypt($request->secret)
            ])->save();
        }
    }

#### 沒有序列化的加密

被加密的值是在加密期間經過 `serialize` 的傳入，這會讓物件和陣列被加密。因此，非 PHP 客戶端接收到被加密的值就會需要 `unserialize` 的資料。如果你希望加密和解密的值沒被序列化，你可以使用 `Crypt` facade 的 `encryptString` 和 `decryptString` 方法：

    use Illuminate\Support\Facades\Crypt;

    $encrypted = Crypt::encryptString('Hello world.');

    $decrypted = Crypt::decryptString($encrypted);

#### 解密一個值

你可以使用 `decrypt` 輔助函式來將值給解碼。如果這個值還沒準備解碼，像是當 MAC 是無效時，會拋出 `Illuminate\Contracts\Encryption\DecryptException` ：

    use Illuminate\Contracts\Encryption\DecryptException;

    try {
        $decrypted = decrypt($encryptedValue);
    } catch (DecryptException $e) {
        //
    }
