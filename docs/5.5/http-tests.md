---
layout: post
title: http-tests
tag: 5.5
---
# HTTP 測試

- [介紹](#introduction)
    - [自訂請求 Header](#customizing-request-headers)
- [Session 和認證](#session-and-authentication)
- [測試 JSON API](#testing-json-apis)
- [測試檔案上傳](#testing-file-uploads)
- [可用的斷言](#available-assertions)

<a name="introduction"></a>
## 介紹

Laravel 提供了一個非常優雅的 API 來向你的應用程式發出 HTTP 請求並檢查輸出。如下面定義的測試：

    <?php

    namespace Tests\Feature;

    use Tests\TestCase;
    use Illuminate\Foundation\Testing\RefreshDatabase;
    use Illuminate\Foundation\Testing\WithoutMiddleware;

    class ExampleTest extends TestCase
    {
        /**
         * 一個基本測試範例。
         *
         * @return void
         */
        public function testBasicTest()
        {
            $response = $this->get('/');

            $response->assertStatus(200);
        }
    }

`get` 方法會建立一個 `GET` 請求給應用程式，而 `assertStatus` 方法來斷言所回傳的回應會有給定的 HTTP 狀態碼。除了這個簡易的斷言，Laravel 也包含用來檢查回應的 Header、內容、JSON 結構等的各種斷言。

<a name="customizing-request-headers"></a>
### 自訂請求 Header

你可以在發送 Header 之前使用 `withHeaders` 方法來自訂請求的 Header。這可以讓你來新增想要的任何自訂請求 Header：

    <?php

    class ExampleTest extends TestCase
    {
        /**
         * 一個基本功能測試範例。
         *
         * @return void
         */
        public function testBasicExample()
        {
            $response = $this->withHeaders([
                'X-Header' => 'Value',
            ])->json('POST', '/user', ['name' => 'Sally']);

            $response
                ->assertStatus(200)
                ->assertJson([
                    'created' => true,
                ]);
        }
    }

<a name="session-and-authentication"></a>
## Session 和認證

Laravel 提供幾個在 HTTP 測試期間會搭配 Session 一起測試的輔助函式。首先，你可以使用 `withSession` 方法來設定 Session 資料來取得陣列。這樣有助於在發送請求到你的應用程式前載入 Session 與資料：

    <?php

    class ExampleTest extends TestCase
    {
        public function testApplication()
        {
            $response = $this->withSession(['foo' => 'bar'])
                             ->get('/');
        }
    }

當然，Session 常被用於維持已認證使用者的狀態。`actingAs` 輔助函式方法提供一個簡易的方式來認證當前的使用者。例如，我們使用[模型工廠](/laravel_tw/docs/5.5/database-testing#writing-factories) 來產生並認證使用者：

    <?php

    use App\User;

    class ExampleTest extends TestCase
    {
        public function testApplication()
        {
            $user = factory(User::class)->create();

            $response = $this->actingAs($user)
                             ->withSession(['foo' => 'bar'])
                             ->get('/');
        }
    }

你也可以透過將 guard 名稱作為第二個參數傳送給 actingAs 方法來指定應該使用那個守衛來驗證給定的使用者：

    $this->actingAs($user, 'api')

<a name="testing-json-apis"></a>
## 測試 JSON API

Laravel 還提供數個測試 JSON API 與回應的輔助函式。例如，`json`、`get`、`post`、`put`、`patch` 和 `delete` 方法可被用於發出各種 HTTP 動詞的請求。你還可以輕易的將資料和標頭傳入這些方法。讓我們開始撰寫一個測試來建立 `POST` 請求到 `/user` 並斷言已回傳的預期資料：

    <?php

    class ExampleTest extends TestCase
    {
        /**
         * 一個基本功能測試範例。
         *
         * @return void
         */
        public function testBasicExample()
        {
            $response = $this->json('POST', '/user', ['name' => 'Sally']);

            $response
                ->assertStatus(200)
                ->assertJson([
                    'created' => true,
                ]);
        }
    }

> {tip}`assertJson` 方法會把回應轉換成陣列，並採用 `PHPUnit::assertArraySubset` 來驗證剛回傳的 JSON 回應中是否存在給定的陣列。所以，如果 JSON 回應中有其他的屬性，只有給定的陣列值存在，這個測試仍然會通過。

<a name="verifying-exact-match"></a>
### 驗證是否與 JSON 完全匹配

如果你希望驗證給定的陣列是否與應用程式回傳的 JSON **完全**匹配，你應該使用 `assertExactJson` 方法：

    <?php

    class ExampleTest extends TestCase
    {
        /**
         * 一個基本功能測試範例。
         *
         * @return void
         */
        public function testBasicExample()
        {
            $response = $this->json('POST', '/user', ['name' => 'Sally']);

            $response
                ->assertStatus(200)
                ->assertExactJson([
                    'created' => true,
                ]);
        }
    }

<a name="testing-file-uploads"></a>
## 測試檔案上傳

`Illuminate\Http\UploadedFile` 類別提供 `fake` 方法可以用來產生虛擬檔案或圖像來進行測試。若與 `Storage` facade 的 `fake` 方法的搭配組合可以大大簡化檔案上傳的測試。例如，你可以組合這兩個功能來輕易的測試頭像上傳表單：

    <?php

    namespace Tests\Feature;

    use Tests\TestCase;
    use Illuminate\Http\UploadedFile;
    use Illuminate\Support\Facades\Storage;
    use Illuminate\Foundation\Testing\RefreshDatabase;
    use Illuminate\Foundation\Testing\WithoutMiddleware;

    class ExampleTest extends TestCase
    {
        public function testAvatarUpload()
        {
            Storage::fake('avatars');

            $response = $this->json('POST', '/avatar', [
                'avatar' => UploadedFile::fake()->image('avatar.jpg')
            ]);

            // 斷言檔案使否有儲存...
            Storage::disk('avatars')->assertExists('avatar.jpg');

            // 斷言檔案是否不存在...
            Storage::disk('avatars')->assertMissing('missing.jpg');
        }
    }

#### 自訂假檔案

使用 `fake` 方法來建立檔案時，你可以為了更好的測試驗證規則而指定圖像的寬、高和其他尺寸：

    UploadedFile::fake()->image('avatar.jpg', $width, $height)->size(100);

除了建立圖片，你可以使用 `create` 方法來建立任何類型的檔案：

    UploadedFile::fake()->create('document.pdf', $sizeInKilobytes);

<a name="available-assertions"></a>
## 可用的斷言

Laravel 為你的 [PHPUnit](https://phpunit.de/) 測試提供了各種自訂的斷言方法。這些斷言可以存取從 `json`、`get`、`post`、`put` 和 `delete` 測試方法所回傳的回應：

方法  | 說明
------------- | -------------
`$response->assertSuccessful();`  |  斷言回應是否有成功的狀態碼。
`$response->assertStatus($code);`  |  斷言回應是否有給定的狀態碼。
`$response->assertRedirect($uri);`  |  斷言回應是否導回給定的 URI。
`$response->assertHeader($headerName, $value = null);`  |  斷言給定的標頭是否存在於回應上。
`$response->assertCookie($cookieName, $value = null);`  |  斷言回應是否包含給定的 cookie。
`$response->assertPlainCookie($cookieName, $value = null);`  |  斷言回應是否包含給定的 cookie (已加密)。
`$response->assertCookieExpired($cookieName);`  |  斷言回應是否包含給定的 cookie 與有效期限。
`$response->assertCookieMissing($cookieName);`  |  斷言回應有沒有不包含給定的 cookie
`$response->assertSessionHas($key, $value = null);`  |  斷言 session 是否包含給的資料片段。
`$response->assertSessionHasErrors(array $keys, $format = null, $errorBag = 'default');`  |  斷言 session 是否包含給定字段的錯誤。
`$response->assertSessionMissing($key);`  |  斷言 session 是否不包含給定的金鑰。
`$response->assertJson(array $data);`  |  斷言回應是否包含給定的 JSON 資料。
`$response->assertJsonFragment(array $data);`  |  斷言回應是否包含給定的 JSON 片段。
`$response->assertJsonMissing(array $data);`  |   斷言回應是否不包含給定的 JSON 片段。
`$response->assertExactJson(array $data);`  |  斷言回應是否與包含給定的 JSON 資料完全匹配。
`$response->assertJsonStructure(array $structure);`  |  斷言回應是否有給定的 JSON 結構。
`$response->assertViewIs($value);`  |  斷言給定視圖是否從路由回傳。
`$response->assertViewHas($key, $value = null);`  |  斷言回應的視圖是否有給定的資料片段。
`$response->assertViewHasAll(array $data);`  |  斷言回應的視圖是否有給定的資料列表。
`$response->assertViewMissing($key);`  |  斷言回應的視圖是否缺少範圍內的資料。
`$response->assertSee($value);`  |  斷言給定的字串是否包含在回應中。
`$response->assertDontSee($value);`  |  斷言給定的字串是否不包含在回應中。
`$response->assertSeeText($value);`  |  斷言給定的字串是否包含在回應文本中。
`$response->assertDontSeeText($value);`  |  斷言給定字串是否不包含在回應文本中。
