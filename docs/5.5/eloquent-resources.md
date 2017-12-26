---
layout: post
title: eloquent-resources
---
# Eloquent：API 資源

- [介紹](#introduction)
- [產生資源](#generating-resources)
- [概念簡述](#concept-overview)
- [寫入資源](#writing-resources)
    - [資料包裝](#data-wrapping)
    - [分頁](#pagination)
    - [有條件的屬性](#conditional-attributes)
    - [有條件的關聯](#conditional-relationships)
    - [新增 Meta Data](#adding-meta-data)
- [資源回應](#resource-responses)

<a name="introduction"></a>
## 介紹

當你在建立一個 API 時，可能會需要一個位於 Eloquent 模型和實際回傳給使用者的 JSON 回應之間的轉換層。Laravel 的資源類別可以讓你更直觀且容易的將你的模型和模型集合轉換成 JSON。

<a name="generating-resources"></a>
## 產生資源

你可以使用 Artisan 的 `make:resource` 指令來產生一個資源類別。預設的資源會放置於你的 `app/Http/Resources` 應用程式目錄中。資源會繼承 `Illuminate\Http\Resources\Json\Resource` 類別：

    php artisan make:resource User

#### 資源集合

除了產生用於轉換個別模型的資源外，你可以為了轉換模型集合而產生專用的資源。這可以讓你的回應中包含與給定資源的整個集合所相關的連結和其他更多資訊。

要建立一個資源集合，你應該在建立資源時使用 `--collection` 選項。或是只在資源名稱中包含 `Collection` 單字，這會告 Laravel 應該建立一個資源集合。資源集合會繼承 `Illuminate\Http\Resources\Json\ResourceCollection` 類別：

    php artisan make:resource Users --collection

    php artisan make:resource UserCollection

<a name="concept-overview"></a>
## 概念簡述

> {tip} 這是一個關於資源與資源集合的高階簡述。強烈建議你閱讀文件的其他部分，以便深入理解資源為你提供自訂的能力。

在你撰寫資源之前，請你深入了解所有可用的選項，讓我們先來看一下 Laravel 如何運用資源。資源類別表示單個模型需要被轉換成陣列結構。例如，這裡是簡易的 `User` 資源類別：

    <?php

    namespace App\Http\Resources;

    use Illuminate\Http\Resources\Json\Resource;

    class User extends Resource
    {
        /**
         * 將資源轉換成陣列。
         *
         * @param  \Illuminate\Http\Request
         * @return array
         */
        public function toArray($request)
        {
            return [
                'id' => $this->id,
                'name' => $this->name,
                'email' => $this->email,
                'created_at' => $this->created_at,
                'updated_at' => $this->updated_at,
            ];
        }
    }

在發送回應時，每個資源類別都定義了一個 toArray 方法，當回傳回應時應該轉換為 JSON 屬性的陣列。要注意一點，我們能直接從 `$this` 變數中存取模型屬性。這是因為資源類別會為了方便存取而自動代理屬性和方法來存取底層的模型。資源一旦被定義，它將可以從路由或控制器中回傳：

    use App\User;
    use App\Http\Resources\User as UserResource;

    Route::get('/user', function () {
        return new UserResource(User::find(1));
    });

### 資源集合

如果你正要回傳資源集合或分頁回應，你可以在建立資源實例時在你的路由或控制器中使用 `collection` 方法：

    use App\User;
    use App\Http\Resources\User as UserResource;

    Route::get('/user', function () {
        return UserResource::collection(User::all());
    });

當然，這麼做不允許附加可能需要與集合一起回傳的 meta data，如果你想要自訂資源集合的回應，你可以建立一個專門用來表示集合的資源：

    php artisan make:resource UserCollection

資源集合類別一旦被產生，你可以輕易的定義應該被包含在回應中的任何資料的資料：

    <?php

    namespace App\Http\Resources;

    use Illuminate\Http\Resources\Json\ResourceCollection;

    class UserCollection extends ResourceCollection
    {
        /**
         * 將資源集合轉換成陣列。
         *
         * @param  \Illuminate\Http\Request
         * @return array
         */
        public function toArray($request)
        {
            return [
                'data' => $this->collection,
                'links' => [
                    'self' => 'link-value',
                ],
            ];
        }
    }

定義資源集合之後，它可以從路由或控制器中回傳：

    use App\User;
    use App\Http\Resources\UserCollection;

    Route::get('/users', function () {
        return new UserCollection(User::all());
    });

<a name="writing-resources"></a>
## 寫入資源

> {tip} 如果你還沒看過[概念簡述](#concept-overview)，在繼續看以下文件之前，強烈建議你回頭看一下。

在本質上，資源並不複雜。他們只需要將給定模型轉換成陣列。因此，每個資源都會包含 `toArray` 方法，該方法會將模型的屬性轉換成對 API 友善的陣列，並回傳給你的使用者：

    <?php

    namespace App\Http\Resources;

    use Illuminate\Http\Resources\Json\Resource;

    class User extends Resource
    {
        /**
         * 將資源轉換成陣列。
         *
         * @param  \Illuminate\Http\Request
         * @return array
         */
        public function toArray($request)
        {
            return [
                'id' => $this->id,
                'name' => $this->name,
                'email' => $this->email,
                'created_at' => $this->created_at,
                'updated_at' => $this->updated_at,
            ];
        }
    }

資源一旦被定義，它將可以直接從路由或控制器中回傳：

    use App\User;
    use App\Http\Resources\User as UserResource;

    Route::get('/user', function () {
        return new UserResource(User::find(1));
    });

#### 關聯

如果你想要在你的回應中包含關聯的資源，你只需要將它們新增到 `toArray` 方法來回傳。在本範例中，我們會使用 `Post`資源的 `collection` 方法來新增使用者的部落格文章到資源回應：

    /**
     * 將資源轉換成陣列。
     *
     * @param  \Illuminate\Http\Request
     * @return array
     */
    public function toArray($request)
    {
        return [
            'id' => $this->id,
            'name' => $this->name,
            'email' => $this->email,
            'posts' => Post::collection($this->posts),
            'created_at' => $this->created_at,
            'updated_at' => $this->updated_at,
        ];
    }

> {tip} 如果你只想要在已經載入的情況下引入關聯，請查看在[條件關聯](#conditional-relationships)的文件。

#### 資源集合

資源將單個模型轉換成陣列，而資源集合會將多個模型的集合轉換成陣列。並沒有必要為每個模型類別定義一個資源集合類別，因為所有的資源會提供 `collection` 方法來即時產生「臨時」的資源集合：

    use App\User;
    use App\Http\Resources\User as UserResource;

    Route::get('/user', function () {
        return UserResource::collection(User::all());
    });

然而，如果你需要自定義與集合一起回傳的 meta data，必須定義一個資源集合：

    <?php

    namespace App\Http\Resources;

    use Illuminate\Http\Resources\Json\ResourceCollection;

    class UserCollection extends ResourceCollection
    {
        /**
         * 將資源集合轉換成陣列。
         *
         * @param  \Illuminate\Http\Request
         * @return array
         */
        public function toArray($request)
        {
            return [
                'data' => $this->collection,
                'links' => [
                    'self' => 'link-value',
                ],
            ];
        }
    }

像是單個資源一樣，資源集合可以從路由或控制器中回傳：

    use App\User;
    use App\Http\Resources\UserCollection;

    Route::get('/users', function () {
        return new UserCollection(User::all());
    });

<a name="data-wrapping"></a>
### 資料包裝

當資源回應要轉換成 JSON 時，預設的最外層資源將會被包裝在 `data` 鍵中。例如，通常資源集合回應會像是以下範例：

    {
        "data": [
            {
                "id": 1,
                "name": "Eladio Schroeder Sr.",
                "email": "therese28@example.com",
            },
            {
                "id": 2,
                "name": "Liliana Mayert",
                "email": "evandervort@example.com",
            }
        ]
    }

如果你想要禁止包裝最外層的資源，你可以在基本 Resource 類別上使用 `withoutWrapping`。通常，你應該會從你的 `AppServiceProvider` 或另一個會在每個請求中都會被載入的服務提供者](/laravel_tw/docs/5.5/providers)中呼叫這個方法：

    <?php

    namespace App\Providers;

    use Illuminate\Support\ServiceProvider;
    use Illuminate\Http\Resources\Json\Resource;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * 執行服務註冊後啟動。
         *
         * @return void
         */
        public function boot()
        {
            Resource::withoutWrapping();
        }

        /**
         * 在容器中註冊綁定。
         *
         * @return void
         */
        public function register()
        {
            //
        }
    }

> {note} `withoutWrapping`方法只影響最外層的回應，並不會移除你手動新增到自己的資源集合的 `data` 鍵。

### 包裝巢狀的資源

你可以非常自由的決定你的資源關聯該如何被包裝。如果你想要所有資源集合被包裝在 `data` 鍵，無論如何被巢狀化，你都應該為每一個資源和回傳的集合中的 `data` 鍵定義資源集合。

當然，你可以會懷疑這麼做是否會導致最外層的資源被包裝在兩個 `data` 鍵中。別擔心，Laravel 永遠不會讓你發生資源被二次包裝，所以你不必擔心正在轉換的資源集合的嵌入層數：

    <?php

    namespace App\Http\Resources;

    use Illuminate\Http\Resources\Json\ResourceCollection;

    class CommentsCollection extends ResourceCollection
    {
        /**
         * 將資源集合轉換成陣列。
         *
         * @param  \Illuminate\Http\Request
         * @return array
         */
        public function toArray($request)
        {
            return ['data' => $this->collection];
        }
    }

### 資料包裝和分頁

當在一個資源回應中回傳分頁集合時，Laravel 會把你的資源資料包裝在 `key` 中，就算它呼叫了 `withoutWrapping` 方法也一樣。這是因為分頁回應總會包含關於分頁狀態資訊的 `meta` 和 `links` 鍵：

    {
        "data": [
            {
                "id": 1,
                "name": "Eladio Schroeder Sr.",
                "email": "therese28@example.com",
            },
            {
                "id": 2,
                "name": "Liliana Mayert",
                "email": "evandervort@example.com",
            }
        ],
        "links":{
            "first": "http://example.com/pagination?page=1",
            "last": "http://example.com/pagination?page=1",
            "prev": null,
            "next": null
        },
        "meta":{
            "current_page": 1,
            "from": 1,
            "last_page": 1,
            "path": "http://example.com/pagination",
            "per_page": 15,
            "to": 10,
            "total": 10
        }
    }

<a name="pagination"></a>
### 分頁

你可以總是傳入分頁實例到資源或自訂資源集合的 `collection` 方法：

    use App\User;
    use App\Http\Resources\UserCollection;

    Route::get('/users', function () {
        return new UserCollection(User::paginate());
    });

已分頁回應總會包含關於分頁狀態資訊的 `meta` 和 `links` 鍵：

    {
        "data": [
            {
                "id": 1,
                "name": "Eladio Schroeder Sr.",
                "email": "therese28@example.com",
            },
            {
                "id": 2,
                "name": "Liliana Mayert",
                "email": "evandervort@example.com",
            }
        ],
        "links":{
            "first": "http://example.com/pagination?page=1",
            "last": "http://example.com/pagination?page=1",
            "prev": null,
            "next": null
        },
        "meta":{
            "current_page": 1,
            "from": 1,
            "last_page": 1,
            "path": "http://example.com/pagination",
            "per_page": 15,
            "to": 10,
            "total": 10
        }
    }

<a name="conditional-attributes"></a>
### 有條件的屬性

有時你可能希望只有在滿足給定條件的資源回應中引入屬性。例如，你可能希望只有目前使用者是「管理者」時才引入值。Laravel 在這種情況下提供了各種輔助的方法。`when` 方法可被用於有條件的新增屬性到資源回應：

    /**
     * 將資源轉換成陣列。
     *
     * @param  \Illuminate\Http\Request
     * @return array
     */
    public function toArray($request)
    {
        return [
            'id' => $this->id,
            'name' => $this->name,
            'email' => $this->email,
            'secret' => $this->when($this->isAdmin(), 'secret-value'),
            'created_at' => $this->created_at,
            'updated_at' => $this->updated_at,
        ];
    }

在這個範例中，只有在 `$this->isAdmin` 方法回傳 `true`時，`secret` 鍵才會在最後的資源回應中被回傳。如果該方法回傳 `false`，`secret` 鍵會從資源回應中完全刪除，然後再發送回客戶端。`when` 方法可以讓你在建構陣列時，不需要在使用條件語句來定義你的資源。

`when` 方法也接收閉包作為它的第二個參數，只有在給定條件為 `true`時，才可以讓你計算結果的值：

    'secret' => $this->when($this->isAdmin(), function () {
        return 'secret-value';
    }),

> {tip} 請記得，在資源上呼叫的方法將被代理到底層模型實例。所以在這個情況下，`isAdmin` 方法是代理最初提供給資源的底層 Eloquent 模型的方法。

#### 合併有條件的屬性

有時候，你可能有幾個屬性應該只包含在基於相同條件的資源回應中。在這種情況下，你可以只在給定條件為 `true` 的回應時，使用 `mergeWhen` 方法來引入屬性：

    /**
     * 將資源轉換成陣列。
     *
     * @param  \Illuminate\Http\Request
     * @return array
     */
    public function toArray($request)
    {
        return [
            'id' => $this->id,
            'name' => $this->name,
            'email' => $this->email,
            $this->mergeWhen($this->isAdmin(), [
                'first-secret' => 'value',
                'second-secret' => 'value',
            ]),
            'created_at' => $this->created_at,
            'updated_at' => $this->updated_at,
        ];
    }

如果給定條件為 `false`，在它發送給客戶端前，這些屬性會從資源回應中被完整刪除。

> {note} `mergeWhen` 方法不應該被用於混合字串和數字鍵的陣列中。此外，它不應該被用於沒有依序排列的數字鍵的陣列。

<a name="conditional-relationships"></a>
### 有條件的關聯

除了載入有條件的屬性外，你可能會根據關聯是否已經載入到模型上，而有條件的在你的 資源回應中引入關聯。這可以讓你的控制器決定應該載入哪一個關聯到模型上，並讓你的資源能輕易的在它們確實要被載入時引入它們。

最終，這麼做可以較容易避免在你的資源中發生「N+1」的查詢問題。`whenLoaded` 方法可以被用於有條件的載入關聯。為了避免不必要的載入關聯，這個方法可以接受關聯名稱，而非關聯本身。

    /**
     * 將資源轉換成陣列。
     *
     * @param  \Illuminate\Http\Request
     * @return array
     */
    public function toArray($request)
    {
        return [
            'id' => $this->id,
            'name' => $this->name,
            'email' => $this->email,
            'posts' => Post::collection($this->whenLoaded('posts')),
            'created_at' => $this->created_at,
            'updated_at' => $this->updated_at,
        ];
    }

在本範例中，如果關聯沒有被載入，在它被傳送到客戶端前，`posts` 鍵會從資源回應中被完全刪除。

#### 有條件的中介資訊

除了在你的資源回應中有條件的引入關聯資訊，你可以有條件的從多對多關聯的中介表中使用 `whenPivotLoaded` 方法來引入資料。`whenPivotLoaded` 方法接受中介表名稱作為它的第一個參數。第二個參數應該會是閉包，它定義了模型上如果中介資訊是可用時就回傳該值：

    /**
     * 將該值轉換成陣列。
     *
     * @param  \Illuminate\Http\Request
     * @return array
     */
    public function toArray($request)
    {
        return [
            'id' => $this->id,
            'name' => $this->name,
            'expires_at' => $this->whenPivotLoaded('role_users', function () {
                return $this->pivot->expires_at;
            }),
        ];
    }

<a name="adding-meta-data"></a>
### 新增 Meta Data

有些 JSON API 標準會要求將 Meta Data 新增到你的資源和資源集合回應中。通常包含了像是 `links` 到資源或關聯資源、或關於資源本身資料的資料。如果你需要回傳有關資源的其他 Meta Data，只要將它引入到 toArray 方法。例如，你在轉換資源集合時需要引入的 `link` 資訊：

    /**
     * 將資源轉換成陣列。
     *
     * @param  \Illuminate\Http\Request
     * @return array
     */
    public function toArray($request)
    {
        return [
            'data' => $this->collection,
            'links' => [
                'self' => 'link-value',
            ],
        ];
    }

當你從資源回傳額外資料的資料時，你從不用擔心意外的覆寫 `links` 或 `meta` 鍵，因為 Laravel 會自動在回傳分頁回應時新增。你定義的任何額外的 `links` 會簡單地與分頁器提供的連結合併。

#### 最上層的 Meta Data

有時你可以希望只有資源最外層的資源被回傳時引入某幾個資源回應資料的資料。通常，這包含整個回應資訊的資訊。要定義這個資料的資料，新增 `with` 方法到你的資源類別中。只有當資源的最外層資源被回傳時，該方法會回傳資料的 Meta Data 陣列來引入到資源回應中。

    <?php

    namespace App\Http\Resources;

    use Illuminate\Http\Resources\Json\ResourceCollection;

    class UserCollection extends ResourceCollection
    {
        /**
         * 將資源集合轉換成陣列。
         *
         * @param  \Illuminate\Http\Request
         * @return array
         */
        public function toArray($request)
        {
            return parent::toArray($request);
        }

        /**
         * 取得應該與資源陣列一起被回傳的額外的資料。
         *
         * @param \Illuminate\Http\Request  $request
         * @return array
         */
        public function with($request)
        {
            return [
                'meta' => [
                    'key' => 'value',
                ],
            ];
        }
    }

#### 在建構資源時新增 Meta Data 的資料

在你的路由或控制器建構資源實例時，你也可以新增最上層的資料。`additional` 方法在所有的資源都可以用，並接受應該被新增到資源回應的資料陣列：

    return (new UserCollection(User::all()->load('roles')))
                    ->additional(['meta' => [
                        'key' => 'value',
                    ]]);

<a name="resource-responses"></a>
## 資源回應

正如你已讀過的，資源可以從路由或控制器中回傳：

    use App\User;
    use App\Http\Resources\User as UserResource;

    Route::get('/user', function () {
        return new UserResource(User::find(1));
    });

然而，有時你可能需要在發送到客戶端之前自訂要輸出的 HTTP 回應。這有兩個方式來達成這個目的。首先，你可以鏈結 `response` 方法到資源上。這個方法會回傳 `Illuminate\Http\Response` 實例，可以讓你完全控制回應的標頭：

    use App\User;
    use App\Http\Resources\User as UserResource;

    Route::get('/user', function () {
        return (new UserResource(User::find(1)))
                    ->response()
                    ->header('X-Value', 'True');
    });

或者是你可以在資源中定義 `withResponse` 方法。這個方法會在資源作為回應中的最外層時被呼叫：

    <?php

    namespace App\Http\Resources;

    use Illuminate\Http\Resources\Json\Resource;

    class User extends Resource
    {
        /**
         * 將資料轉換成陣列。
         *
         * @param  \Illuminate\Http\Request
         * @return array
         */
        public function toArray($request)
        {
            return [
                'id' => $this->id,
            ];
        }

        /**
         * 為資源自訂要輸出的回應。
         *
         * @param  \Illuminate\Http\Request
         * @param  \Illuminate\Http\Response
         * @return void
         */
        public function withResponse($request, $response)
        {
            $response->header('X-Value', 'True');
        }
    }
