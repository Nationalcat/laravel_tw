---
layout: post
title: helpers
tag: 5.5
---
# 輔助函式

- [介紹](#introduction)
- [可用的方法](#available-methods)

<a name="introduction"></a>
## 介紹

Laravel 包含了各式各樣的全域「輔助」PHP 函式。很多函式都有在框架本身使用到。如果你也覺得很方便的話，可以在應用程式中隨意的使用它們。

<a name="available-methods"></a>
## 可用的方法

<style>
    .collection-method-list > p {
        column-count: 3; -moz-column-count: 3; -webkit-column-count: 3;
        column-gap: 2em; -moz-column-gap: 2em; -webkit-column-gap: 2em;
    }

    .collection-method-list a {
        display: block;
    }
</style>

### 陣列與物件

<div class="collection-method-list" markdown="1">

[array_add](#method-array-add)
[array_collapse](#method-array-collapse)
[array_divide](#method-array-divide)
[array_dot](#method-array-dot)
[array_except](#method-array-except)
[array_first](#method-array-first)
[array_flatten](#method-array-flatten)
[array_forget](#method-array-forget)
[array_get](#method-array-get)
[array_has](#method-array-has)
[array_last](#method-array-last)
[array_only](#method-array-only)
[array_pluck](#method-array-pluck)
[array_prepend](#method-array-prepend)
[array_pull](#method-array-pull)
[array_random](#method-array-random)
[array_set](#method-array-set)
[array_sort](#method-array-sort)
[array_sort_recursive](#method-array-sort-recursive)
[array_where](#method-array-where)
[array_wrap](#method-array-wrap)
[data_fill](#method-data-fill)
[data_get](#method-data-get)
[data_set](#method-data-set)
[head](#method-head)
[last](#method-last)
</div>

### 路徑

<div class="collection-method-list" markdown="1">

[app_path](#method-app-path)
[base_path](#method-base-path)
[config_path](#method-config-path)
[database_path](#method-database-path)
[mix](#method-mix)
[public_path](#method-public-path)
[resource_path](#method-resource-path)
[storage_path](#method-storage-path)

</div>

### 字串

<div class="collection-method-list" markdown="1">

[\__](#method-__)
[camel_case](#method-camel-case)
[class_basename](#method-class-basename)
[e](#method-e)
[ends_with](#method-ends-with)
[kebab_case](#method-kebab-case)
[preg_replace_array](#method-preg-replace-array)
[snake_case](#method-snake-case)
[starts_with](#method-starts-with)
[str_after](#method-str-after)
[str_before](#method-str-before)
[str_contains](#method-str-contains)
[str_finish](#method-str-finish)
[str_is](#method-str-is)
[str_limit](#method-str-limit)
[str_plural](#method-str-plural)
[str_random](#method-str-random)
[str_replace_array](#method-str-replace-array)
[str_replace_first](#method-str-replace-first)
[str_replace_last](#method-str-replace-last)
[str_singular](#method-str-singular)
[str_slug](#method-str-slug)
[str_start](#method-str-start)
[studly_case](#method-studly-case)
[title_case](#method-title-case)
[trans](#method-trans)
[trans_choice](#method-trans-choice)

</div>

### URL

<div class="collection-method-list" markdown="1">

[action](#method-action)
[asset](#method-asset)
[secure_asset](#method-secure-asset)
[route](#method-route)
[secure_url](#method-secure-url)
[url](#method-url)

</div>

### 其它

<div class="collection-method-list" markdown="1">

[abort](#method-abort)
[abort_if](#method-abort-if)
[abort_unless](#method-abort-unless)
[app](#method-app)
[auth](#method-auth)
[back](#method-back)
[bcrypt](#method-bcrypt)
[blank](#method-blank)
[broadcast](#method-broadcast)
[cache](#method-cache)
[class_uses_recursive](#method-class-uses-recursive)
[collect](#method-collect)
[config](#method-config)
[cookie](#method-cookie)
[csrf_field](#method-csrf-field)
[csrf_token](#method-csrf-token)
[dd](#method-dd)
[decrypt](#method-decrypt)
[dispatch](#method-dispatch)
[dispatch_now](#method-dispatch-now)
[dump](#method-dump)
[encrypt](#method-encrypt)
[env](#method-env)
[event](#method-event)
[factory](#method-factory)
[filled](#method-filled)
[info](#method-info)
[logger](#method-logger)
[method_field](#method-method-field)
[now](#method-now)
[old](#method-old)
[optional](#method-optional)
[policy](#method-policy)
[redirect](#method-redirect)
[report](#method-report)
[request](#method-request)
[rescue](#method-rescue)
[resolve](#method-resolve)
[response](#method-response)
[retry](#method-retry)
[session](#method-session)
[tap](#method-tap)
[today](#method-today)
[throw_if](#method-throw-if)
[throw_unless](#method-throw-unless)
[trait_uses_recursive](#method-trait-uses-recursive)
[transform](#method-transform)
[validator](#method-validator)
[value](#method-value)
[view](#method-view)
[with](#method-with)

</div>

<a name="method-listing"></a>
## 方法清單

<style>
    #collection-method code {
        font-size: 14px;
    }

    #collection-method:not(.first-collection-method) {
        margin-top: 50px;
    }
</style>

<a name="arrays"></a>
## 陣列與物件

<a name="method-array-add"></a>
#### `array_add()` {#collection-method .first-collection-method}

如果指定的 key 在陣列中不存在，`array_add` 函式就會把指定的 key / value 新增到陣列中：

    $array = array_add(['name' => 'Desk'], 'price', 100);

    // ['name' => 'Desk', 'price' => 100]

<a name="method-array-collapse"></a>
#### `array_collapse()` {#collection-method}

可以使用 `array_collapse` 函式將二維陣列合併成一維陣列：

    $array = array_collapse([[1, 2, 3], [4, 5, 6], [7, 8, 9]]);

    // [1, 2, 3, 4, 5, 6, 7, 8, 9]

<a name="method-array-divide"></a>
#### `array_divide()` {#collection-method}

`array_divide` 函式會回傳兩組陣列，一組是原本陣列的鍵，另一組是原本陣列的值：

    list($keys, $values) = array_divide(['name' => 'Desk']);

    // $keys: ['name']

    // $values: ['Desk']

<a name="method-array-dot"></a>
#### `array_dot()` {#collection-method}

`array_dot` 函式可以將多維陣列扁平化成一維陣列，並使用「點」來表示其深度：

    $array = ['products' => ['desk' => ['price' => 100]]];

    $flattened = array_dot($array);

    // ['products.desk.price' => 100]

<a name="method-array-except"></a>
#### `array_except()` {#collection-method}

`array_except` 函式可以從一維陣列中移除特定的鍵值對：

    $array = ['name' => 'Desk', 'price' => 100];

    $filtered = array_except($array, ['price']);

    // ['name' => 'Desk']

<a name="method-array-first"></a>
#### `array_first()` {#collection-method}

`array_first` 函式可以回傳陣列中第一個通過條件的元素：

    $array = [100, 200, 300];

    $first = array_first($array, function ($value, $key) {
        return $value >= 150;
    });

    // 200

可以傳入第三個參數作為預設值。如果陣列中沒有通過條件的元素，就會回傳這個預設值：

    $first = array_first($array, $callback, $default);

<a name="method-array-flatten"></a>
#### `array_flatten()` {#collection-method}

`array_flatten` 函式可以將多維陣列的值給扁平化成一維陣列：

    $array = ['name' => 'Joe', 'languages' => ['PHP', 'Ruby']];

    $flattened = array_flatten($array);

    // ['Joe', 'PHP', 'Ruby']

<a name="method-array-forget"></a>
#### `array_forget()` {#collection-method}

`array_forget` 函式可以使用「點」表示法來從多維的巢狀陣列中移除特定的鍵值對：

    $array = ['products' => ['desk' => ['price' => 100]]];

    array_forget($array, 'products.desk');

    // ['products' => []]

<a name="method-array-get"></a>
#### `array_get()` {#collection-method}

`array_get` 函式可以使用「點」表示法來從多維的巢狀陣列中取得一個值：

    $array = ['products' => ['desk' => ['price' => 100]]];

    $price = array_get($array, 'products.desk.price');

    // 100

`array_get` 函式也可以接受一個預設值，如果沒找到特定的鍵，就會回傳這個預設值：

    $discount = array_get($array, 'products.desk.discount', 0);

    // 0

<a name="method-array-has"></a>
#### `array_has()` {#collection-method}

`array_has` 函式可以使用「點」表示法來檢查給定的項目是否存在於陣列中：

    $array = ['product' => ['name' => 'Desk', 'price' => 100]];

    $contains = array_has($array, 'product.name');

    // true

    $contains = array_has($array, ['product.price', 'product.discount']);

    // false

<a name="method-array-last"></a>
#### `array_last()` {#collection-method}

`array_last` 函式會回傳陣列中最後一個通過條件的元素：

    $array = [100, 200, 300, 110];

    $last = array_last($array, function ($value, $key) {
        return $value >= 150;
    });

    // 300

可以傳入第三個參數作為預設值。如果陣列中沒有通過條件的元素，就會回傳這個預設值：

    $last = array_last($array, $callback, $default);

<a name="method-array-only"></a>
#### `array_only()` {#collection-method}

`array_only` 函式只會從陣列中回傳特定的鍵值對：

    $array = ['name' => 'Desk', 'price' => 100, 'orders' => 10];

    $slice = array_only($array, ['name', 'price']);

    // ['name' => 'Desk', 'price' => 100]

<a name="method-array-pluck"></a>
#### `array_pluck()` {#collection-method}

`array_pluck` 函式可以從陣列中取得給定鍵的所有值：

    $array = [
        ['developer' => ['id' => 1, 'name' => 'Taylor']],
        ['developer' => ['id' => 2, 'name' => 'Abigail']],
    ];

    $names = array_pluck($array, 'developer.name');

    // ['Taylor', 'Abigail']

你也可以指定其結果的鍵值形式：

    $names = array_pluck($array, 'developer.name', 'developer.id');

    // [1 => 'Taylor', 2 => 'Abigail']

<a name="method-array-prepend"></a>
#### `array_prepend()` {#collection-method}

`array_prepend` 函式能把一個項目擠到陣列的開頭位置：

    $array = ['one', 'two', 'three', 'four'];

    $array = array_prepend($array, 'zero');

    // ['zero', 'one', 'two', 'three', 'four']

如果有需要，你可以指定應該被用於該值的鍵：

    $array = ['price' => 100];

    $array = array_prepend($array, 'Desk', 'name');

    // ['name' => 'Desk', 'price' => 100]

<a name="method-array-pull"></a>
#### `array_pull()` {#collection-method}

`array_pull` 函式可以從陣列中回傳並移除一個鍵值對：

    $array = ['name' => 'Desk', 'price' => 100];

    $name = array_pull($array, 'name');

    // $name: Desk

    // $array: ['price' => 100]

可以傳入第三個參數作為預設值。要是該鍵不存在，就會回傳預設值：

    $value = array_pull($array, $key, $default);

<a name="method-array-random"></a>
#### `array_random()` {#collection-method}

`array_random` 函式可以從一組陣列中回傳一個隨機的值：

    $array = [1, 2, 3, 4, 5];

    $random = array_random($array);

    // 4 - (retrieved randomly)

你還可以指定該方法回傳的項目數量作為可選的第二個參數。請注意，一旦使用了該參數，其回傳的結果一定是陣列，縱使你只需要它回傳一個項目也一樣：

    $items = array_random($array, 2);

    // [2, 5] - (retrieved randomly)

<a name="method-array-set"></a>
#### `array_set()` {#collection-method}

`array_set` 函式可以使用「點」表示法在深層巢狀陣列中設定一個值：

    $array = ['products' => ['desk' => ['price' => 100]]];

    array_set($array, 'products.desk.price', 200);

    // ['products' => ['desk' => ['price' => 200]]]

<a name="method-array-sort"></a>
#### `array_sort()` {#collection-method}

`array_sort` 函式會依照陣列的值由小到大來排序：

    $array = ['Desk', 'Table', 'Chair'];

    $sorted = array_sort($array);

    // ['Chair', 'Desk', 'Table']

你也可以藉由給定閉包的結果來為陣列排序：

    $array = [
        ['name' => 'Desk'],
        ['name' => 'Table'],
        ['name' => 'Chair'],
    ];

    $sorted = array_values(array_sort($array, function ($value) {
        return $value['name'];
    }));

    /*
        [
            ['name' => 'Chair'],
            ['name' => 'Desk'],
            ['name' => 'Table'],
        ]
    */

<a name="method-array-sort-recursive"></a>
#### `array_sort_recursive()` {#collection-method}

`array_sort_recursive` 函式會使用 PHP 的 `sort` 函式來為一組陣列做一次遞迴排序：

    $array = [
        ['Roman', 'Taylor', 'Li'],
        ['PHP', 'Ruby', 'JavaScript'],
    ];

    $sorted = array_sort_recursive($array);

    /*
        [
            ['JavaScript', 'PHP', 'Ruby'],
            ['Li', 'Roman', 'Taylor'],
        ]
    */

<a name="method-array-where"></a>
#### `array_where()` {#collection-method}

`array_where` 函式可以使用給定的閉包來過濾陣列：

    $array = [100, '200', 300, '400', 500];

    $filtered = array_where($array, function ($value, $key) {
        return is_string($value);
    });

    // [1 => 200, 3 => 400]

<a name="method-array-wrap"></a>
#### `array_wrap()` {#collection-method}

`array_wrap` 函式會將給定的值包裝在一組陣列中。如果給定的值已經是一組陣列，那麼就不會做任何變動：

    $string = 'Laravel';

    $array = array_wrap($string);

    // ['Laravel']

<a name="method-data-fill"></a>
#### `data_fill()` {#collection-method}

可以在 `data_fill` 函式中使用「點」表示法來表示巢狀陣列或物件，並補齊當中所缺少的鍵值：

    $data = ['products' => ['desk' => ['price' => 100]]];

    data_fill($data, 'products.desk.price', 200);

    // ['products' => ['desk' => ['price' => 100]]]

    data_fill($data, 'products.desk.discount', 10);

    // ['products' => ['desk' => ['price' => 100, 'discount' => 10]]]

這個函式還接受星號作為萬用字元，並補齊預期的鍵值：

    $data = [
        'products' => [
            ['name' => 'Desk 1', 'price' => 100],
            ['name' => 'Desk 2'],
        ],
    ];

    data_fill($data, 'products.*.price', 200);

    /*
        [
            'products' => [
                ['name' => 'Desk 1', 'price' => 100],
                ['name' => 'Desk 2', 'price' => 200],
            ],
        ]
    */

<a name="method-data-get"></a>
#### `data_get()` {#collection-method}

可以在 `data_get` 函式中使用「點」符號來表示巢狀陣列或物件，並從中取得值：

    $data = ['products' => ['desk' => ['price' => 100]]];

    $price = data_get($data, 'products.desk.price');

    // 100

`data_get` 函式還可以接受一個預設值，這會在找不到指定的鍵時回傳它。

    $discount = data_get($data, 'products.desk.discount', 0);

    // 0

<a name="method-data-set"></a>
#### `data_set()` {#collection-method}

可以在 `data_set` 函式中使用「點」符號來表示巢狀陣列或物件，並給它們設定一個值：

    $data = ['products' => ['desk' => ['price' => 100]]];

    data_set($data, 'products.desk.price', 200);

    // ['products' => ['desk' => ['price' => 200]]]

這個函式還可以接受萬用字元，並設定預期的鍵值：

    $data = [
        'products' => [
            ['name' => 'Desk 1', 'price' => 100],
            ['name' => 'Desk 2', 'price' => 150],
        ],
    ];

    data_set($data, 'products.*.price', 200);

    /*
        [
            'products' => [
                ['name' => 'Desk 1', 'price' => 200],
                ['name' => 'Desk 2', 'price' => 200],
            ],
        ]
    */

預設會覆寫任何現有的值。如果你只是想為那些沒有值的鍵賦予一個值，你可以傳入 `false` 到該方法的第三個參數：

    $data = ['products' => ['desk' => ['price' => 100]]];

    data_set($data, 'products.desk.price', 200, false);

    // ['products' => ['desk' => ['price' => 100]]]

<a name="method-head"></a>
#### `head()` {#collection-method}

`head` 函式可以回傳給定陣列的第一個元素：

    $array = [100, 200, 300];

    $first = head($array);

    // 100

<a name="method-last"></a>
#### `last()` {#collection-method}

`last` 函式可以回傳給定陣列的最後一個元素：

    $array = [100, 200, 300];

    $last = last($array);

    // 300

<a name="paths"></a>
## 路徑

<a name="method-app-path"></a>
#### `app_path()` {#collection-method}

`app_path` 函式會回傳 `app` 目錄的完整路徑。你也可以使用 `app_path` 函式來產生一個在 app 目錄中給定檔案的完整路徑：

    $path = app_path();

    $path = app_path('Http/Controllers/Controller.php');

<a name="method-base-path"></a>
#### `base_path()` {#collection-method}

`base_path` 函式會回傳專案根目錄的完整路徑。你也可以使用 `base_path` 函式來產生一個在專案根目錄中給定檔案的完整路徑：

    $path = base_path();

    $path = base_path('vendor/bin');

<a name="method-config-path"></a>
#### `config_path()` {#collection-method}

`config_path` 函式會回傳 `config` 目錄的完整路徑。你也可以使用 `config_path` 函式來產生一個在應用程式設定目錄中給定檔案的完整路徑：

    $path = config_path();

    $path = config_path('app.php');

<a name="method-database-path"></a>
#### `database_path()` {#collection-method}

`database_path` 函式會回傳 `database` 目錄的完整路徑。你也可以使用 `database_path` 函式來產生一個在 database 目錄中給定檔案的完整路徑：

    $path = database_path();

    $path = database_path('factories/UserFactory.php');

<a name="method-mix"></a>
#### `mix()` {#collection-method}

`mix` 函式會回傳一個[版控化的 Mix 檔案](/laravel_tw/docs/5.5/mix)路徑:

    $path = mix('css/app.css');

<a name="method-public-path"></a>
#### `public_path()` {#collection-method}

`public_path` 函式會回傳 `public` 目錄的完整路徑。你也可以使用 `public_path` 函式來產生一個在 public 目錄中給定檔案的完整路徑：

    $path = public_path();

    $path = public_path('css/app.css');

<a name="method-resource-path"></a>
#### `resource_path()` {#collection-method}

`resource_path` 函式會回傳 `resources` 目錄的完整路徑。你也可以使用 `resource_path` 函式來產生一個在 resources 目錄中給定檔案的完整路徑：

    $path = resource_path();

    $path = resource_path('assets/sass/app.scss');

<a name="method-storage-path"></a>
#### `storage_path()` {#collection-method}

`storage_path` 函式會回傳 `storage` 目錄的完整路徑。你也可以使用 `storage_path` 函式來產生一個在 storage 目錄中給定檔案的完整路徑：

    $path = storage_path();

    $path = storage_path('app/file.txt');

<a name="strings"></a>
## 字串

<a name="method-__"></a>
#### `__()` {#collection-method}

`__` 函式會使用[本地化檔案](/laravel_tw/docs/5.5/localization)來翻譯給定的語系字串或語系鍵：

    echo __('Welcome to our application');

    echo __('messages.welcome');

如果被指定的語系字串或鍵不存在的話，`__` 函式只會回傳給定的值。所以，以上面的範例來說，`__` 方法如果找不到語系字串時就會直接回傳 `messages.welcome` 字串。

<a name="method-camel-case"></a>
#### `camel_case()` {#collection-method}

`camel_case` 函式會將給定字串轉換成`駝峰式命名`：

    $converted = camel_case('foo_bar');

    // fooBar

<a name="method-class-basename"></a>
#### `class_basename()` {#collection-method}

`class_basename` 回傳不包含命名空間的類別名稱：

    $class = class_basename('Foo\Bar\Baz');

    // Baz

<a name="method-e"></a>
#### `e()` {#collection-method}

`e` 函式會去執行 PHP 的 `htmlspecialchars` 函式，並將 `double_encode` 選項設為 `false`：

    echo e('<html>foo</html>');

    // &lt;html&gt;foo&lt;/html&gt;

<a name="method-ends-with"></a>
#### `ends_with()` {#collection-method}

`ends_with` 函式可以去檢查給定字串是否是以指定內容來做結尾：

    $result = ends_with('This is my name', 'name');

    // true

<a name="method-kebab-case"></a>
#### `kebab_case()` {#collection-method}

`kebab_case` 函式可以將給定字串轉換成 `kebab-case`:

    $converted = kebab_case('fooBar');

    // foo-bar

<a name="method-preg-replace-array"></a>
#### `preg_replace_array()` {#collection-method}

`preg_replace_array` 函式可以使用一組陣列的順序來替換字串中的給定模式：

    $string = 'The event will take place between :start and :end';

    $replaced = preg_replace_array('/:[a-z_]+/', ['8:30', '9:00'], $string);

    // The event will take place between 8:30 and 9:00

<a name="method-snake-case"></a>
#### `snake_case()` {#collection-method}

`snake_case` 函式可以將給定字串轉換成`蛇式命名`:

    $converted = snake_case('fooBar');

    // foo_bar

<a name="method-starts-with"></a>
#### `starts_with()` {#collection-method}

`starts_with` 函式可以檢查給定字串的開頭是否為指定內容：

    $result = starts_with('This is my name', 'This');

    // true

<a name="method-str-after"></a>
#### `str_after()` {#collection-method}

`str_after` 函式會回傳在指定內容字串之後的所有內容：

    $slice = str_after('This is my name', 'This is');

    // ' my name'

<a name="method-str-before"></a>
#### `str_before()` {#collection-method}

`str_before` 函式會回傳在指定內容字串之前的所有內容：

    $slice = str_before('This is my name', 'my name');

    // 'This is '

<a name="method-str-contains"></a>
#### `str_contains()` {#collection-method}

`str_contains` 函式可以檢查給定字串是否包含指定的內容：

    $contains = str_contains('This is my name', 'my');

    // true

你也可以傳入一組陣列的值來檢查給定字串是否包含指定內容：

    $contains = str_contains('This is my name', ['my', 'foo']);

    // true

<a name="method-str-finish"></a>
#### `str_finish()` {#collection-method}

`str_finish` 函式可以將指定的內容新增到字串的結尾，如果結尾已有指定內容則不變動：

    $adjusted = str_finish('this/string', '/');

    // this/string/

    $adjusted = str_finish('this/string/', '/');

    // this/string/

<a name="method-str-is"></a>
#### `str_is()` {#collection-method}

`str_is` 函式會去檢查給定的字串是否與給定的字串形式相匹配。可以使用星號作為萬用字元：

    $matches = str_is('foo*', 'foobar');

    // true

    $matches = str_is('baz*', 'foobar');

    // false

<a name="method-str-limit"></a>
#### `str_limit()` {#collection-method}

`str_limit` 函式可以指定字串長度，並省略過多的內容：

    $truncated = str_limit('The quick brown fox jumps over the lazy dog', 20);

    // The quick brown fox...

還可以傳入第三個參數來取代預設會附加到結尾的「...」：

    $truncated = str_limit('The quick brown fox jumps over the lazy dog', 20, ' (...)');

    // The quick brown fox (...)

<a name="method-str-plural"></a>
#### `str_plural()` {#collection-method}

`str_plural` 函式可以將字串轉換成複數形式。目前這個函式僅支援英文單字：

    $plural = str_plural('car');

    // cars

    $plural = str_plural('child');

    // children

可以提供一個整數值作為該函式的第二個參數，這可被用來指定要回傳單數還是複數：

    $plural = str_plural('child', 2);

    // children

    $plural = str_plural('child', 1);

    // child

<a name="method-str-random"></a>
#### `str_random()` {#collection-method}

`str_random` 函式可以產生一個指定長度的隨機字串。這個函式主要使用 PHP 的 `random_bytes` 函式：

    $random = str_random(40);

<a name="method-str-replace-array"></a>
#### `str_replace_array()` {#collection-method}

`str_replace_array` 函式可以使用一組陣列的順序來替換字串中指定內容：

    $string = 'The event will take place between ? and ?';

    $replaced = str_replace_array('?', ['8:30', '9:00'], $string);

    // The event will take place between 8:30 and 9:00

<a name="method-str-replace-first"></a>
#### `str_replace_first()` {#collection-method}

`str_replace_first` 函式可以更換字串中第一個出現的指定內容：

    $replaced = str_replace_first('the', 'a', 'the quick brown fox jumps over the lazy dog');

    // a quick brown fox jumps over the lazy dog

<a name="method-str-replace-last"></a>
#### `str_replace_last()` {#collection-method}

`str_replace_last` 函式可以更換字串中最後一個出現的指定內容：

    $replaced = str_replace_last('the', 'a', 'the quick brown fox jumps over the lazy dog');

    // the quick brown fox jumps over a lazy dog

<a name="method-str-singular"></a>
#### `str_singular()` {#collection-method}

`str_singular` 函式可以將一個字串轉換成它的單數形式。目前這個函式僅支援英文單字。

    $singular = str_singular('cars');

    // car

    $singular = str_singular('children');

    // child

<a name="method-str-slug"></a>
#### `str_slug()` {#collection-method}

`str_slug` 函式根據指定的字串生成一個對 URL 友善的「slug」：

    $slug = str_slug('Laravel 5 Framework', '-');

    // laravel-5-framework

<a name="method-str-start"></a>
#### `str_start()` {#collection-method}

`str_start`  函式可以將指定的內容新增到字串的開頭，如果開頭已有指定內容則不變動：

    $adjusted = str_start('this/string', '/');

    // /this/string

    $adjusted = str_start('/this/string/', '/');

    // /this/string

<a name="method-studly-case"></a>
#### `studly_case()` {#collection-method}

`studly_case` 函式可以將給定字串轉換成 `StudlyCas`：

    $converted = studly_case('foo_bar');

    // FooBar

<a name="method-title-case"></a>
#### `title_case()` {#collection-method}

`title_case` 函式可以將給定字串轉換成 `Title Case`:

    $converted = title_case('a nice title uses the correct case');

    // A Nice Title Uses The Correct Case

<a name="method-trans"></a>
#### `trans()` {#collection-method}

`trans` 函式會使用你的[在地化檔案](/laravel_tw/docs/5.5/localization)來翻譯給定的語系鍵：

    echo trans('messages.welcome');

如果指定語系鍵不存在，那麼 `trans` 函式只會回傳給定鍵。所以，以上述來範例來說，`trans` 函式會因為語系鍵不存在的關係而直接回傳 `messages.welcome`。

<a name="method-trans-choice"></a>
#### `trans_choice()` {#collection-method}

`trans_choice` 函式根據數量來翻譯給定的語系鍵：

    echo trans_choice('messages.notifications', $unreadCount);

如果指定的語系鍵不存在，那麼 `trans_choice` 函式只會回傳給定鍵。所以，以上述範例來說，`trans_choice` 函式會因為語系鍵不存在的關係而直接會傳 `messages.notifications`。

<a name="urls"></a>
## URL

<a name="method-action"></a>
#### `action()` {#collection-method}

`action` 函式可以為給定的控制器行為產生一個 URL。你不需要傳入控制器的完整命名空間，而是傳入相對於 `App\Http\Controllers` 命名空間的控制器類別名稱：

    $url = action('HomeController@index');

如果該方法需要接受的路由參數，可以將一組陣列傳入該方法的第二個參數。

    $url = action('UserController@profile', ['id' => 1]);

<a name="method-asset"></a>
#### `asset()` {#collection-method}

`asset` 函式可以產生一組使用當前請求協定（HTTP 或 HTTPS）的資源檔網址：

    $url = asset('img/photo.jpg');

<a name="method-secure-asset"></a>
#### `secure_asset()` {#collection-method}

`secure_asset` 函式可以產生一組使用 HTTPS 的資源檔網址：

    $url = secure_asset('img/photo.jpg');

<a name="method-route"></a>
#### `route()` {#collection-method}

`route` 函式會使用指定路由名稱來產生一組網址：

    $url = route('routeName');

如果該路由需要接受參數，你可以將一組陣列傳入該方法：

    $url = route('routeName', ['id' => 1]);

預設的 `route` 函式會產生一組絕對路徑的路由。如果你想要產生一組相對路徑的網址，你可以將 `false` 傳入該方法的第三個參數：

    $url = route('routeName', ['id' => 1], false);

<a name="method-secure-url"></a>
#### `secure_url()` {#collection-method}

`secure_url` 函式可以為給定路徑產生一組完整的 HTTPS 網址：

    $url = secure_url('user/profile');

    $url = secure_url('user/profile', [1]);

<a name="method-url"></a>
#### `url()` {#collection-method}

`url` 函式可以為給定路徑產生一組完整的 HTTP 網址：

    $url = url('user/profile');

    $url = url('user/profile', [1]);

如果沒有提供路徑給方法，則會回傳 `Illuminate\Routing\UrlGenerator` 實例：

    $current = url()->current();

    $full = url()->full();

    $previous = url()->previous();

<a name="miscellaneous"></a>
## 其它

<a name="method-abort"></a>
#### `abort()` {#collection-method}

`abort` 函式會使用[例外處理器](/laravel_tw/docs/5.5/errors#the-exception-handler)來渲染並回應[一件 HTTP 例外](/laravel_tw/docs/5.5/errors#http-exceptions)：

    abort(403);

你還可以為例外的回應提供額外的文字內容與自訂標頭：

    abort(403, 'Unauthorized.', $headers);

<a name="method-abort-if"></a>
#### `abort_if()` {#collection-method}

`abort_if` 函式會在給定布林表達式的結果為 `ture` 時，回應一件 HTTP 例外：

    abort_if(! Auth::user()->isAdmin(), 403);

就像是使用 `abort` 方法，你還可以提供例外的回應文字內容作為第三個參數，並提供一組自訂回應標頭的陣列作為第四個參數。

<a name="method-abort-unless"></a>
#### `abort_unless()` {#collection-method}

`abort_unless` 函式會在給定布林表達式結果為 `false` 時，回應一件 HTTP 例外：

    abort_unless(Auth::user()->isAdmin(), 403);

就像是使用 `abort` 方法，你還可以提供例外的回應文字內容作為第三個參數，並提供一組自訂回應標頭的陣列作為第四個參數。

<a name="method-app"></a>
#### `app()` {#collection-method}

`app` 函式可以回傳[服務容器](/laravel_tw/docs/5.5/container)的實例：

    $container = app();

你可以傳入一個類別或介面名稱來使用服務容器解析功能：

    $api = app('HelpSpot\API');

<a name="method-auth"></a>
#### `auth()` {#collection-method}

`auth` 函式可以回傳[認證功能](/laravel_tw/docs/5.5/authentication)的實例。可以用來取代 `Auth` Facade 來更方便的使用：

    $user = auth()->user();

如果你有需要，可以指定想要存取的 Guard 實例：

    $user = auth('admin')->user();

<a name="method-back"></a>
#### `back()` {#collection-method}

`back` 函式可以產生一個[重導 HTTP 回應](/laravel_tw/docs/5.5/responses#redirects)來讓使用者回到前一個瀏覽的位置：

    return back($status = 302, $headers = [], $fallback = false);

    return back();

<a name="method-bcrypt"></a>
#### `bcrypt()` {#collection-method}

`bcrypt` 函式使用 Bcrypt 來[雜湊](/laravel_tw/docs/5.5/hashing)給定的值。你可以使用這個函式來取代 `Hash` Facade：

    $password = bcrypt('my-secret-password');

<a name="method-broadcast"></a>
#### `broadcast()` {#collection-method}

`broadcast` 函式會[廣播](/laravel_tw/docs/5.5/broadcasting)給定[事件](/laravel_tw/docs/5.5/events)給它的監聽器：

    broadcast(new UserRegistered($user));

<a name="method-blank"></a>
#### `blank()` {#collection-method}

`blank` 函式回傳給定值是否為「空值」：

    blank('');
    blank('   ');
    blank(null);
    blank(collect());

    // true

    blank(0);
    blank(true);
    blank(false);

    // false

`blank` 的另一個相反的方法為 [`filled`](#method-filled)。

<a name="method-cache"></a>
#### `cache()` {#collection-method}

`cache` 函式可以被用來從[快取](/laravel_tw/docs/5.5/cache)中取得值。如果在快取中沒有給定的鍵，就會回傳自訂的預設值：

    $value = cache('key');

    $value = cache('key', 'default');

你可以將一組鍵值對傳入該函式來把它新增到快取中。你還能傳入快取值的有效與持續時間：

    cache(['key' => 'value'], 5);

    cache(['key' => 'value'], Carbon::now()->addSeconds(10));

<a name="method-class-uses-recursive"></a>
#### `class_uses_recursive()` {#collection-method}

`class_uses_recursive` 函式會回傳一個類別使用的所有 trait，也包含任何子類別所用的 trait：

    $traits = class_uses_recursive(App\User::class);

<a name="method-collect"></a>
#### `collect()` {#collection-method}

`collect` 函式可以從給定值中建立一個 [collection](/laravel_tw/docs/5.5/collections) 實例：

    $collection = collect(['taylor', 'abigail']);

<a name="method-config"></a>
#### `config()` {#collection-method}

`config` 函式可以取得[系統設定](/laravel_tw/docs/5.5/configuration)的變數值。可以使用「點」語法來存取系統設定值，這還包括了檔案名稱和你想要存取的選項。如果設定選項不存在，則可以回傳一個指定的預設值：

    $value = config('app.timezone');

    $value = config('app.timezone', $default);

你可以在執行時傳入一組鍵值對來寫入系統設定變數：

    config(['app.debug' => true]);

<a name="method-cookie"></a>
#### `cookie()` {#collection-method}

`cookie` 函式可以建立一個新的 [cookie](/laravel_tw/docs/5.5/requests#cookies) 實例：

    $cookie = cookie('name', 'value', $minutes);

<a name="method-csrf-field"></a>
#### `csrf_field()` {#collection-method}

`csrf_field` 函式可以產生一組 `hidden` 輸入欄位，並包含了 CSRF token 的值。例如，使用 [Blade 語法](/laravel_tw/docs/5.5/blade)：

    {% raw %} {{ csrf_field() }} {% endraw %}

<a name="method-csrf-token"></a>
#### `csrf_token()` {#collection-method}

`csrf_token` 函式可以取得當前 CSRF token 的值：

    $token = csrf_token();

<a name="method-dd"></a>
#### `dd()` {#collection-method}

`dd` 函式會印出給定變數並結束腳本執行：

    dd($value);

    dd($value1, $value2, $value3, ...);

如果你不想中斷腳本的執行，請改使用 [`dump`](#method-dump) 函式。

<a name="method-decrypt"></a>
#### `decrypt()` {#collection-method}

`decrypt` 函式會使用 Laravel 的[解密器](/laravel_tw/docs/5.5/encryption)來解密給定的值：

    $decrypted = decrypt($encrypted_value);

<a name="method-dispatch"></a>
#### `dispatch()` {#collection-method}

`dispatch` 函式可以將給定的[任務](/laravel_tw/docs/5.5/queues#creating-jobs)推送到 Laravel [任務隊列](/laravel_tw/docs/5.5/queues)：

    dispatch(new App\Jobs\SendEmails);

<a name="method-dispatch-now"></a>
#### `dispatch_now()` {#collection-method}

`dispatch_now` 函式會馬上執行給定的[任務](/laravel_tw/docs/5.5/queues#creating-jobs)並從 `handle` 方法中回傳結果：

    $result = dispatch_now(new App\Jobs\SendEmails);

<a name="method-dump"></a>
#### `dump()` {#collection-method}

`dump` 函式可以印出給定的變數：

    dump($value);

    dump($value1, $value2, $value3, ...);

如果你想要在印出變數以後，馬上中斷腳本的執行。請改用 [`dd`](#method-dd) 函式。

<a name="method-encrypt"></a>
#### `encrypt()` {#collection-method}

`encrypt` 函式會使用 Laravel [加密器](/laravel_tw/docs/5.5/encryption)來加密給定的值：

    $encrypted = encrypt($unencrypted_value);

<a name="method-env"></a>
#### `env()` {#collection-method}

`env` 函式可以取得一個[環境變數](/laravel_tw/docs/5.5/configuration#environment-configuration)的值或回傳一個預設值：

    $env = env('APP_ENV');

    // 回傳 'production'，因為沒有設定 APP_ENV...
    $env = env('APP_ENV', 'production');

<a name="method-event"></a>
#### `event()` {#collection-method}

`event` 函式可以調度給定的[事件](/laravel_tw/docs/5.5/events)到它的監聽器：

    event(new UserRegistered($user));

<a name="method-factory"></a>
#### `factory()` {#collection-method}

`factory` 函式可以為給定類別、名稱和數量建立一個模型工廠建構器。它能被用在[測試](/laravel_tw/docs/5.5/database-testing#writing-factories)或[資料填充](/laravel_tw/docs/5.5/seeding#using-model-factories)：

    $user = factory(App\User::class)->make();

<a name="method-filled"></a>
#### `filled()` {#collection-method}

`filled` 函式可以回傳給定的值是否不是「空白」：

    filled(0);
    filled(true);
    filled(false);

    // true

    filled('');
    filled('   ');
    filled(null);
    filled(collect());

    // false

`filled` 的另一個相反的方法為 [`blank`](#method-blank)。

<a name="method-info"></a>
#### `info()` {#collection-method}

`info` 函式會寫入資訊到 [log](/laravel_tw/docs/5.5/errors#logging)：

    info('Some helpful information!');

還能將一組情境資料的陣列傳入該函式：

    info('User login attempt failed.', ['id' => $user->id]);

<a name="method-logger"></a>
#### `logger()` {#collection-method}

`logger` 函式能被用在寫入一個 `debug` 等級的訊息到 [log](/laravel_tw/docs/5.5/errors#logging):

    logger('Debug message');

還能將一組情境資料的陣列傳入該函式：

    logger('User has logged in.', ['id' => $user->id]);

如果沒有值可以傳入該函式，將會回傳 [logger](/laravel_tw/docs/5.5/errors#logging) 實例：

    logger()->error('You are not allowed here.');

<a name="method-method-field"></a>
#### `method_field()` {#collection-method}

`method_field`  函式產生擬造 HTTP 表單動作內容的 HTML 表單隱藏欄位。例如，請使用 [Blade 語法](/laravel_tw/docs/5.5/blade)：

    <form method="POST">
        {% raw %} {{ method_field('DELETE') }} {% endraw %}
    </form>

<a name="method-now"></a>
#### `now()` {#collection-method}

`now` 函式可以為當前時間建立一個 `Illuminate\Support\Carbon` 實例：

    $now = now();

<a name="method-old"></a>
#### `old()` {#collection-method}

`old` 函式可以[取得](/laravel_tw/docs/5.5/requests#retrieving-input)快閃到 session 的[舊有輸入](/laravel_tw/docs/5.5/requests#old-input)數值：

    $value = old('value');

    $value = old('value', 'default');

<a name="method-optional"></a>
#### `optional()` {#collection-method}

`optional` 函式接受任何參數，並可以讓你存取物件的屬性或呼叫的方法。如果給定的物件為 `null`，則屬性和方法也會只回傳 `null` 而不是回傳錯誤：

    return optional($user->address)->street;

    {!! old('name', optional($user)->name) !!}

<a name="method-policy"></a>
#### `policy()` {#collection-method}

`policy` 方法可以為給定類別取得一個 [policy](/laravel_tw/docs/5.5/authorization#creating-policies) 實例：

    $policy = policy(App\User::class);

<a name="method-redirect"></a>
#### `redirect()` {#collection-method}

`redirect` 函式可以回傳一個[重導 HTTP 回應](/laravel_tw/docs/5.5/responses#redirects)，或在沒有參數的情況下回傳重導器實例：

    return redirect($to = null, $status = 302, $headers = [], $secure = null);

    return redirect('/home');

    return redirect()->route('route.name');

<a name="method-report"></a>
#### `report()` {#collection-method}

`report` 函式會使用[例外處理器](/laravel_tw/docs/5.5/errors#the-exception-handler)的 `report` 方法來回報一個例外：

    report($e);

<a name="method-request"></a>
#### `request()` {#collection-method}

`request` 函式可以回傳當前[請求](/laravel_tw/docs/5.5/requests)的實例或取得一個輸入項目：

    $request = request();

    $value = request('key', $default = null);

<a name="method-rescue"></a>
#### `rescue()` {#collection-method}

`rescue` 函式會執行給定的閉包並捕捉執行過程中的任何例外。所有捕捉到的例外會被傳送到你的[例外處理器](/laravel_tw/docs/5.5/errors#the-exception-handler)的 `report` 方法。然而，會繼續處理該請求：

    return rescue(function () {
        return $this->method();
    });

你還可以傳入第二個參數到 `rescue` 函式。如果在執行閉包的期間發生例外，這個參數會做為預設值來回傳：

    return rescue(function () {
        return $this->method();
    }, false);

    return rescue(function () {
        return $this->method();
    }, function () {
        return $this->failure();
    });

<a name="method-resolve"></a>
#### `resolve()` {#collection-method}

`resolve` 函式使用[服務容器](/laravel_tw/docs/5.5/container)將給定的類別或介面名稱解析到它的實例：

    $api = resolve('HelpSpot\API');

<a name="method-response"></a>
#### `response()` {#collection-method}

`response` 函式可以建立一個[回應](/laravel_tw/docs/5.5/responses)實例或取得一個回應工廠的實例：

    return response('Hello World', 200, $headers);

    return response()->json(['foo' => 'bar'], 200, $headers);

<a name="method-retry"></a>
#### `retry()` {#collection-method}

`retry` 函式會嘗試執行給定的回呼，直到給定的最長嘗試次數為止。如果回呼沒有拋出例外，就會回傳它的回傳值。如果回呼拋出例外，就會自動重試。如果超過最大嘗試次數，就會拋出例外：

    return retry(5, function () {

        // 嘗試五次並間隔 100ms ...
    }, 100);

<a name="method-session"></a>
#### `session()` {#collection-method}

`session` 函式可被用在取得或設定 [session](/laravel_tw/docs/5.5/session) 的值：

    $value = session('key');

你可以使用一組鍵值對傳入該函式來設定值：

    session(['chairs' => 7, 'instruments' => 3]);

該函式在沒有內容傳遞時，將回傳所儲存的 seesion 內容：

    $value = session()->get('key');

    session()->put('key', $value);

<a name="method-tap"></a>
#### `tap()` {#collection-method}

`tap` 函式接受兩組參數：其一為任意的 `$value`，另一個為一個閉包。`$value` 會被傳入到閉包中，然後被 `tap` 函數所回傳。閉包不能將結果回傳給函式：

    $user = tap(User::first(), function ($user) {
        $user->name = 'taylor';

        $user->save();
    });

如果沒有閉包要被傳入 `tap` 函式，那麼可以呼叫在給定的 `$value` 上的任何方法。不論該方法實際回傳了什麼值，你呼叫的方法最終都會回傳 `$value`。例如，Eloquent 的 `update` 方法一般會回傳一個整數。然而，我們能強制該方法通過 `tap` 函式鏈結呼叫 `update` 方法來回傳模型身：

    $user = tap($user)->update([
        'name' => $name,
        'email' => $email,
    ]);

<a name="method-today"></a>
#### `today()` {#collection-method}

`today` 函式可以為當前日期建立一個新的 `Illuminate\Support\Carbon` 實例：

    $today = today();

<a name="method-throw-if"></a>
#### `throw_if()` {#collection-method}

`throw_if` 函式會在給定布林表達式為 `true` 時拋出給定例外：

    throw_if(! Auth::user()->isAdmin(), AuthorizationException::class);

    throw_if(
        ! Auth::user()->isAdmin(),
        AuthorizationException::class,
        'You are not allowed to access this page'
    );

<a name="method-throw-unless"></a>
#### `throw_unless()` {#collection-method}

`throw_unless` 函式會在給定布林表達式結果為 `false` 時拋出給定的例外：

    throw_unless(Auth::user()->isAdmin(), AuthorizationException::class);

    throw_unless(
        Auth::user()->isAdmin(),
        AuthorizationException::class,
        'You are not allowed to access this page'
    );

<a name="method-trait-uses-recursive"></a>
#### `trait_uses_recursive()` {#collection-method}

`trait_uses_recursive` 函式可以回傳 trait 所用到的所有 trait：

    $traits = trait_uses_recursive(\Illuminate\Notifications\Notifiable::class);

<a name="method-transform"></a>
#### `transform()` {#collection-method}

`transform` 函式會先判斷給定的值是否為[空白](#method-blank)，若不是空白則去執行`閉包`，並回傳該`閉包`的結果：

    $callback = function ($value) {
        return $value * 2;
    };

    $result = transform(5, $callback);

    // 10

也可以使用預設值或`閉包`作為傳入該方法的第三個參數。這個值會在第一個參數值為空白時回傳它：

    $result = transform(null, $callback, 'The value is blank');

    // The value is blank

<a name="method-validator"></a>
#### `validator()` {#collection-method}

`validator` 函式可以建立一個新的[驗證器](/laravel_tw/docs/5.5/validation)實例與給定的參數。你可以使用它來取代 `Validator` Facade：

    $validator = validator($data, $rules, $messages);

<a name="method-value"></a>
#### `value()` {#collection-method}

`value` 函式會回傳給定的值。然而，如果你是傳入一個`閉包`到該函式，則會執行`閉包`，並回傳它的結果：

    $result = value(true);

    // true

    $result = value(function () {
        return false;
    });

    // false

<a name="method-view"></a>
#### `view()` {#collection-method}

`view` 函式可以接受一個[視圖](/laravel_tw/docs/5.5/views)實例：

    return view('auth.login');

<a name="method-with"></a>
#### `with()` {#collection-method}

`with` 函式會回傳給定的值。若將`閉包`作為傳入該函式的第二個參數，則會執行`閉包`，並回傳它的結果：

    $callback = function ($value) {
        return (is_numeric($value)) ? $value * 2 : 0;
    };

    $result = with(5, $callback);

    // 10

    $result = with(null, $callback);

    // 0

    $result = with(5, null);

    // 5
