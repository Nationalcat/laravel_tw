---
layout: post
title: redirects
tag: 5.5
---
# HTTP 重導

- [建立重導](#creating-redirects)
- [重導向被命名的路由](#redirecting-named-routes)
- [重導向到 Controller 的 Action](#redirecting-controller-actions)
- [重導與被快閃的 Session 資料](#redirecting-with-flashed-session-data)

<a name="creating-redirects"></a>
## 建立重導

重導回應是 `Illuminate\Http\RedirectResponse` class 的實例，並包含了將使用者重導到其他 URL 所需要的正確 header。有許多方式可以產生一個 `RedirectResponse` 實例。最簡單的方式是使用全域 `redirect` 輔助函式：

    Route::get('dashboard', function () {
        return redirect('home/dashboard');
    });

有時候你或許希望重導使用者到先前的位置，像是當我們提交了無效的表單，你可以使用全域 `back` 輔助函式來執行。由於這個 feature 利用了 [session](/laravel_tw/docs/5.5/session)，確保這個呼叫 `back` 的函式路由是使用 `web` 中介層群組或是已經套用了所有 session 中介層：

    Route::post('user/profile', function () {
        // 驗證請求...

        return back()->withInput();
    });

<a name="redirecting-named-routes"></a>
## 重導向被命名的路由

當你不帶任何參數呼叫 `redirect` 輔助函式時，會回傳一個 `Illuminate\Routing\Redirector` 實例，允許你去呼叫任何在 `Redirector` 實例上的方法。例如，要產生一個 `RedirectResponse` 到一個被命名的路由，你可以使用 `route` 方法：

    return redirect()->route('login');

如果你的路由有參數的話，你可以傳送它們作為 `route` 方法的第二個參數：

    // 以下的路由 URI 規則：profile/{id}

    return redirect()->route('profile', ['id' => 1]);

#### 藉由 Eloquent 模型來填充參數

如果你從一個 Eloquent 模型填充一個「ID」參數重導到一個路由，你可以透過模型本身傳送。ID 將會自動地被提取：

    // 以下的路由 URI 規則：profile/{id}

    return redirect()->route('profile', [$user]);

如果你想要自定義路由參數中放置的值，你可以覆寫 Eloquent 模型內的 `getRouteKey` 方法：

    /**
     * 取得模型路由 key 的值。
     *
     * @return mixed
     */
    public function getRouteKey()
    {
        return $this->slug;
    }

<a name="redirecting-controller-actions"></a>
## 重導向到 Controller 的 Action

你也許會產生重導向到 [controller 的 action](/laravel_tw/docs/5.5/controllers)。如果需要的話，傳送 controller 和 action 名稱到 `action` 方法。記住，你不需要指定完整的命名空間，Laravel 的 `RouteServiceProvider` 將會自動幫你設定好基礎的 controller 命名空間：

    return redirect()->action('HomeController@index');

如果你的 controller 路由需要參數，你可以把它們作為 `action` 方法的第二個參數：

    return redirect()->action(
        'UserController@profile', ['id' => 1]
    );

<a name="redirecting-with-flashed-session-data"></a>
## 重導與被快閃的 Session 資料

通常在重導向到新的 URL 也會把[資料快閃到 session](/laravel_tw/docs/5.5/session#flash-data)。通常當你成功執行一個 action 後，你會快閃成功訊息到 session。你可以藉由單一的語法鏈結流暢的建立一個 `RedirectResponse` 實例並快閃資料到 session：

    Route::post('user/profile', function () {
        // 更新使用者的個人資訊...

        return redirect('dashboard')->with('status', 'Profile updated!');
    });

在使用者重導向後，你可以從 [session](/laravel_tw/docs/5.5/session) 顯示被快閃訊息。例如，使用 [Blade 語法](/laravel_tw/docs/5.5/blade)：

    @if (session('status'))
        <div class="alert alert-success">
            {% raw %} {{ session('status') }} {% endraw %}
        </div>
    @endif
