---
layout: post
title: collections
tag: 5.5
---
# 集合

- [簡介](#introduction)
    - [建立集合](#creating-collections)
    - [擴充集合](#extending-collections)
- [可用的方法](#available-methods)
- [高級訊息傳遞](#higher-order-messages)

<a name="introduction"></a>
## 簡介

`Illuminate\Support\Collection` 類別提供一個流暢、便利的封裝來操控陣列資料。舉個例子，查看下列的程式碼。我們將用 `collect` 輔助方法從陣列建立一個新的集合實例，對每一個元素執行 `strtoupper` 函式，然後移除所有的空元素：

    $collection = collect(['taylor', 'abigail', null])->map(function ($name) {
        return strtoupper($name);
    })
    ->reject(function ($name) {
        return empty($name);
    });

如你所見，`Collection` 類別允許你鏈結它的方法以對底層的陣列流暢地進行映射與刪減。一般來說，每一個 `Collection` 方法會回傳一個全新的 `Collection` 實例。

<a name="creating-collections"></a>
### 建立集合

如上所述，`collect` 輔助方法會用傳入的陣列回傳一個新的 `Illuminate\Support\Collection` 實例。所以要建立一個集合就這麼簡單：

    $collection = collect([1, 2, 3]);

> {tip} [Eloquent](/laravel_tw/docs/5.5/eloquent) 查詢結果，每次都會回傳`Collection`的實例。

<a name="extending-collections"></a>
### 擴充集合

每個集合都是可"巨集化"的，這允許你在程式運作期間增加額外的方法到集合內。以下程式碼示範了如何將`toUpper`加進`Collection`裡：

    use Illuminate\Support\Str;

    Collection::macro('toUpper', function () {
        return $this->map(function ($value) {
            return Str::upper($value);
        });
    });
    
    $collection = collect(['first', 'second']);
    
    $upper = $collection->toUpper();
    
    // ['FIRST', 'SECOND']

一般來說, 你可以將巨集定義在 [service provider](/laravel_tw/docs/5.5/providers) 裡.

<a name="available-methods"></a>
## 可用的方法

在這份文件剩餘的部份，我們將會探討每一個 `Collection` 類別上可用的方法。要先知道的是，以下所有的方法，都可以用串鏈式的方式處理陣列。另外，大部分的方法都會回傳一個新的 `Collection` 實例，讓你在必要時，還有原始的數組可使用。

<style>
    #collection-method-list > p {
        column-count: 3; -moz-column-count: 3; -webkit-column-count: 3;
        column-gap: 2em; -moz-column-gap: 2em; -webkit-column-gap: 2em;
    }
    
    #collection-method-list a {
        display: block;
    }
</style>

<div id="collection-method-list" markdown="1">

[all](#method-all)
[average](#method-average)
[avg](#method-avg)
[chunk](#method-chunk)
[collapse](#method-collapse)
[combine](#method-combine)
[concat](#method-concat)
[contains](#method-contains)
[containsStrict](#method-containsstrict)
[count](#method-count)
[crossJoin](#method-crossjoin)
[dd](#method-dd)
[diff](#method-diff)
[diffAssoc](#method-diffassoc)
[diffKeys](#method-diffkeys)
[dump](#method-dump)
[each](#method-each)
[eachSpread](#method-eachspread)
[every](#method-every)
[except](#method-except)
[filter](#method-filter)
[first](#method-first)
[flatMap](#method-flatmap)
[flatten](#method-flatten)
[flip](#method-flip)
[forget](#method-forget)
[forPage](#method-forpage)
[get](#method-get)
[groupBy](#method-groupby)
[has](#method-has)
[implode](#method-implode)
[intersect](#method-intersect)
[intersectByKeys](#method-intersectbykeys)
[isEmpty](#method-isempty)
[isNotEmpty](#method-isnotempty)
[keyBy](#method-keyby)
[keys](#method-keys)
[last](#method-last)
[macro](#method-macro)
[make](#method-make)
[map](#method-map)
[mapInto](#method-mapinto)
[mapSpread](#method-mapspread)
[mapToGroups](#method-maptogroups)
[mapWithKeys](#method-mapwithkeys)
[max](#method-max)
[median](#method-median)
[merge](#method-merge)
[min](#method-min)
[mode](#method-mode)
[nth](#method-nth)
[only](#method-only)
[pad](#method-pad)
[partition](#method-partition)
[pipe](#method-pipe)
[pluck](#method-pluck)
[pop](#method-pop)
[prepend](#method-prepend)
[pull](#method-pull)
[push](#method-push)
[put](#method-put)
[random](#method-random)
[reduce](#method-reduce)
[reject](#method-reject)
[reverse](#method-reverse)
[search](#method-search)
[shift](#method-shift)
[shuffle](#method-shuffle)
[slice](#method-slice)
[sort](#method-sort)
[sortBy](#method-sortby)
[sortByDesc](#method-sortbydesc)
[splice](#method-splice)
[split](#method-split)
[sum](#method-sum)
[take](#method-take)
[tap](#method-tap)
[times](#method-times)
[toArray](#method-toarray)
[toJson](#method-tojson)
[transform](#method-transform)
[union](#method-union)
[unique](#method-unique)
[uniqueStrict](#method-uniquestrict)
[unless](#method-unless)
[unwrap](#method-unwrap)
[values](#method-values)
[when](#method-when)
[where](#method-where)
[whereStrict](#method-wherestrict)
[whereIn](#method-wherein)
[whereInStrict](#method-whereinstrict)
[whereNotIn](#method-wherenotin)
[whereNotInStrict](#method-wherenotinstrict)
[wrap](#method-wrap)
[zip](#method-zip)

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

<a name="method-all"></a>
#### `all()` {#collection-method .first-collection-method}

`all` 方法回傳該集合所代表的底層陣列：

    collect([1, 2, 3])->all();

    // [1, 2, 3]

<a name="method-average"></a>
#### `average()` {#collection-method}

此為 [`avg`](#method-avg) 方法的別名。

<a name="method-avg"></a>
#### `avg()` {#collection-method}

`avg` 方法會回傳指定 key 所對應數值的[平均值](https://en.wikipedia.org/wiki/Average)：

    $average = collect([['foo' => 10], ['foo' => 10], ['foo' => 20], ['foo' => 40]])->avg('foo');

    // 20

    $average = collect([1, 1, 2, 4])->avg();

    // 2

<a name="method-chunk"></a>
#### `chunk()` {#collection-method}

`chunk` 方法將集合拆成多個給定大小的較小集合：

    $collection = collect([1, 2, 3, 4, 5, 6, 7]);

    $chunks = $collection->chunk(4);

    $chunks->toArray();

    // [[1, 2, 3, 4], [5, 6, 7]]

當你在 [views](/laravel_tw/docs/5.5/views) 裡有使用像 [Bootstrap](https://getbootstrap.com/css/#grid) 的網格系統的時候，這方法會特別有用。想像一下你要在網格系統裡展示出 [Eloquent](/laravel_tw/docs/5.5/eloquent) 的數據集時：

    @foreach ($products->chunk(3) as $chunk)
        <div class="row">
            @foreach ($chunk as $product)
                <div class="col-xs-4">{% raw %} {{ $product->name }} {% endraw %}</div>
            @endforeach
        </div>
    @endforeach

<a name="method-collapse"></a>
#### `collapse()` {#collection-method}

`collapse` 方法將多個陣列組成的集合折成單一陣列集合：

    $collection = collect([[1, 2, 3], [4, 5, 6], [7, 8, 9]]);

    $collapsed = $collection->collapse();

    $collapsed->all();

    // [1, 2, 3, 4, 5, 6, 7, 8, 9]

<a name="method-combine"></a>
#### `combine()` {#collection-method}

`combine` 方法將此一集合的值當成鍵值，與另一個陣列或集合的值結合成新的集合。

    $collection = collect(['name', 'age']);

    $combined = $collection->combine(['George', 29]);

    $combined->all();

    // ['name' => 'George', 'age' => 29]

<a name="method-concat"></a>
#### `concat()` {#collection-method}

`concat` 方法會將給定的`陣列`或集合的值附加在此一集合之後。

    $collection = collect(['John Doe']);

    $concatenated = $collection->concat(['Jane Doe'])->concat(['name' => 'Johnny Doe']);

    $concatenated->all();

    // ['John Doe', 'Jane Doe', 'Johnny Doe']

<a name="method-contains"></a>
#### `contains()` {#collection-method}

`contains` 方法用來判斷該集合是否含有指定的項目：

    $collection = collect(['name' => 'Desk', 'price' => 100]);

    $collection->contains('Desk');

    // true

    $collection->contains('New York');

    // false

你可以將一對鍵/值傳入 `contains` 方法，用來判斷該組合是否存在於集合內：

    $collection = collect([
        ['product' => 'Desk', 'price' => 200],
        ['product' => 'Chair', 'price' => 100],
    ]);
    
    $collection->contains('product', 'Bookcase');
    
    // false

最後，你也可以傳入一個回呼函式到 `contains` 方法內執行你自己的判斷式：

    $collection = collect([1, 2, 3, 4, 5]);

    $collection->contains(function ($value, $key) {
        return $value > 5;
    });
    
    // false

`contains` 方法使用`弱型別比較`，所以當字串的內容是數字時，比較的結果會等於型態是 integer 的數字。可使用 [`containsStrict`](#method-containsstrict) 方法改成強型別比較。

<a name="method-containsstrict"></a>
#### `containsStrict()` {#collection-method}

這個方法的用法與 [`contains`](#method-contains) 相同，但所有數值都會用「嚴格模式」來比較。

<a name="method-count"></a>
#### `count()` {#collection-method}

`count` 方法回傳該集合內的項目總數：

    $collection = collect([1, 2, 3, 4]);

    $collection->count();

    // 4

<a name="method-crossjoin"></a>
#### `crossJoin()` {#collection-method}

`crossJoin` 方法會將此一集合的值與給定的陣列或集合做交叉結合，並返回所有排列組合的笛卡兒積。

    $collection = collect([1, 2]);

    $matrix = $collection->crossJoin(['a', 'b']);

    $matrix->all();

    /*
        [
            [1, 'a'],
            [1, 'b'],
            [2, 'a'],
            [2, 'b'],
        ]
    */
    
    $collection = collect([1, 2]);
    
    $matrix = $collection->crossJoin(['a', 'b'], ['I', 'II']);
    
    $matrix->all();
    
    /*
        [
            [1, 'a', 'I'],
            [1, 'a', 'II'],
            [1, 'b', 'I'],
            [1, 'b', 'II'],
            [2, 'a', 'I'],
            [2, 'a', 'II'],
            [2, 'b', 'I'],
            [2, 'b', 'II'],
        ]
    */

<a name="method-dd"></a>
#### `dd()` {#collection-method}

`dd` 方法會傾倒出該集合內的所有元素，並結束執行此程式。

    $collection = collect(['John Doe', 'Jane Doe']);

    $collection->dd();

    /*
        array:2 [
            0 => "John Doe"
            1 => "Jane Doe"
        ]
    */

如果你不想停止執行，可以使用 [`dump`](#method-dump) 方法替代。

<a name="method-diff"></a>
#### `diff()` {#collection-method}

`diff` 方法會比較其他集合或純 PHP `array` 裡的值。最後返回有在原始集合裡，但沒有在給定集合內的值。

    $collection = collect([1, 2, 3, 4, 5]);

    $diff = $collection->diff([2, 4, 6, 8]);

    $diff->all();

    // [1, 3, 5]

<a name="method-diffassoc"></a>
#### `diffAssoc()` {#collection-method}

`diffAssoc` 方法會比較其他集合或純 PHP `array` 的鍵與值。最後返回有在原始集合裡，但沒有在給定集合內的鍵與值：

    $collection = collect([
        'color' => 'orange',
        'type' => 'fruit',
        'remain' => 6
    ]);
    
    $diff = $collection->diffAssoc([
        'color' => 'yellow',
        'type' => 'fruit',
        'remain' => 3,
        'used' => 6
    ]);
    
    $diff->all();
    
    // ['color' => 'orange', 'remain' => 6]

<a name="method-diffkeys"></a>
#### `diffKeys()` {#collection-method}

`diffKeys` 方法會比較其他集合或純 PHP `array` 的鍵與值。最後返回有在原始集合裡，但沒有在給定集合內的鍵與其對應的值：

    $collection = collect([
        'one' => 10,
        'two' => 20,
        'three' => 30,
        'four' => 40,
        'five' => 50,
    ]);
    
    $diff = $collection->diffKeys([
        'two' => 2,
        'four' => 4,
        'six' => 6,
        'eight' => 8,
    ]);
    
    $diff->all();
    
    // ['one' => 10, 'three' => 30, 'five' => 50]

<a name="method-dump"></a>
#### `dump()` {#collection-method}

`dump` 方法會傾印出此集合的所有元素：

    $collection = collect(['John Doe', 'Jane Doe']);

    $collection->dump();

    /*
        Collection {
            #items: array:2 [
                0 => "John Doe"
                1 => "Jane Doe"
            ]
        }
    */

如果你想在傾印後停止執行程式，可使用 [`dd`](#method-dd) 方法代替。

<a name="method-each"></a>
#### `each()` {#collection-method}

`each` 方法遍歷集合中的元素，並將之傳入給定的回呼函式：

    $collection = $collection->each(function ($item, $key) {
        //
    });

如果你想在某個元素中斷迴圈，可以在回呼函式返回 `false`

    $collection = $collection->each(function ($item, $key) {
        if (/* some condition */) {
            return false;
        }
    });

<a name="method-eachspread"></a>
#### `eachSpread()` {#collection-method}

`eachSpread` 方法遍歷集合中的元素，並將每個巢狀元素都傳給回呼函式: 

    $collection = collect([['John Doe', 35], ['Jane Doe', 33]]);

    $collection->eachSpread(function ($name, $age) {
        //
    });

如果你想在某個元素中斷迴圈，可以在回呼函式返回 `false`:

    $collection->eachSpread(function ($name, $age) {
        return false;
    });

<a name="method-every"></a>
#### `every()` {#collection-method}

`every` 方法用於檢驗所有元素是否通過所給予的測試：

    collect([1, 2, 3, 4])->every(function ($value, $key) {
        return $value > 2;
    });
    
    // false

<a name="method-except"></a>
#### `except()` {#collection-method}

`except` 方法回傳集合中排除指定鍵的所有項目：

    $collection = collect(['product_id' => 1, 'price' => 100, 'discount' => false]);

    $filtered = $collection->except(['price', 'discount']);

    $filtered->all();

    // ['product_id' => 1]

與 `except` 相反的方法請查看 [only](#method-only)。

<a name="method-filter"></a>
#### `filter()` {#collection-method}

`filter` 方法以給定的回呼函式篩選集合，只留下那些通過判斷測試的項目：

    $collection = collect([1, 2, 3, 4]);

    $filtered = $collection->filter(function ($value, $key) {
        return $value > 2;
    });
    
    $filtered->all();
    
    // [3, 4]

如果沒有提供回呼函式，就會移除所有等於 `false` 的元素：

    $collection = collect([1, 2, 3, null, false, '', 0, []]);

    $collection->filter()->all();

    // [1, 2, 3]

與 `filter` 相對的方法可以檢視 [reject](#method-reject)。

<a name="method-first"></a>
#### `first()` {#collection-method}

`first` 方法回傳集合中，第一個通過給定測試的元素：

    collect([1, 2, 3, 4])->first(function ($value, $key) {
        return $value > 2;
    });
    
    // 3

你也可以不傳入參數使用 `first` 方法以取得集合中第一個元素。如果集合是空的，則會回傳 `null`：

    collect([1, 2, 3, 4])->first();

    // 1

<a name="method-flatmap"></a>
#### `flatMap()` {#collection-method}

`flatMap` 方法將集合內元素傳入回呼函式中，此回呼函式可對每個元素任意修改並回傳，從而形成一個修改過的新項目集合。該陣列會被扁平化成同一個層級：

    $collection = collect([
        ['name' => 'Sally'],
        ['school' => 'Arkansas'],
        ['age' => 28]
    ]);
    
    $flattened = $collection->flatMap(function ($values) {
        return array_map('strtoupper', $values);
    });
    
    $flattened->all();
    
    // ['name' => 'SALLY', 'school' => 'ARKANSAS', 'age' => '28'];

<a name="method-flatten"></a>
#### `flatten()` {#collection-method}

`flatten` 方法將多維集合轉為一維集合：

    $collection = collect(['name' => 'taylor', 'languages' => ['php', 'javascript']]);

    $flattened = $collection->flatten();

    $flattened->all();

    // ['taylor', 'php', 'javascript'];

你還可以帶入「深度」參數

    $collection = collect([
        'Apple' => [
            ['name' => 'iPhone 6S', 'brand' => 'Apple'],
        ],
        'Samsung' => [
            ['name' => 'Galaxy S7', 'brand' => 'Samsung']
        ],
    ]);
    
    $products = $collection->flatten(1);
    
    $products->values()->all();
    
    /*
        [
            ['name' => 'iPhone 6S', 'brand' => 'Apple'],
            ['name' => 'Galaxy S7', 'brand' => 'Samsung'],
        ]
    */

在這例子中，若沒有傳入「深度」參數，將會得到`['iPhone 6S', 'Apple', 'Galaxy S7', 'Samsung']`的結果。提供深度可指定從哪一階層開始。

<a name="method-flip"></a>
#### `flip()` {#collection-method}

`flip` 方法將集合中的鍵和對應的數值進行互換：

    $collection = collect(['name' => 'taylor', 'framework' => 'laravel']);

    $flipped = $collection->flip();

    $flipped->all();

    // ['taylor' => 'name', 'laravel' => 'framework']

<a name="method-forget"></a>
#### `forget()` {#collection-method}

`forget` 方法以鍵自集合移除掉一個項目：

    $collection = collect(['name' => 'taylor', 'framework' => 'laravel']);

    $collection->forget('name');

    $collection->all();

    // ['framework' => 'laravel']

> {note} 與大多數的集合的方法不同，`forget` 不會回傳修改過後的新集合，而是會直接修改原先呼叫的集合。

<a name="method-forpage"></a>
#### `forPage()` {#collection-method}

`forPage` 方法回傳含有可以用來在給定頁碼顯示的項目的新集合。第一個參數是頁數，第二個則是每頁要顯示的項目筆數。

    $collection = collect([1, 2, 3, 4, 5, 6, 7, 8, 9]);

    $chunk = $collection->forPage(2, 3);

    $chunk->all();

    // [4, 5, 6]

<a name="method-get"></a>
#### `get()` {#collection-method}

`get` 方法回傳給定鍵的項目。如果該鍵不存在，則回傳 `null`：

    $collection = collect(['name' => 'taylor', 'framework' => 'laravel']);

    $value = $collection->get('name');

    // taylor

你可以選擇性地傳入一個預設值為第二個參數：

    $collection = collect(['name' => 'taylor', 'framework' => 'laravel']);

    $value = $collection->get('foo', 'default-value');

    // default-value

你甚至可以傳入回呼函式當預設值。如果指定的鍵不存在，就會回傳回呼函式的執行結果：

    $collection->get('email', function () {
        return 'default-value';
    });
    
    // default-value

<a name="method-groupby"></a>
#### `groupBy()` {#collection-method}

`groupBy` 方法根據給定的鍵替集合內的項目分組：

    $collection = collect([
        ['account_id' => 'account-x10', 'product' => 'Chair'],
        ['account_id' => 'account-x10', 'product' => 'Bookcase'],
        ['account_id' => 'account-x11', 'product' => 'Desk'],
    ]);
    
    $grouped = $collection->groupBy('account_id');
    
    $grouped->toArray();
    
    /*
        [
            'account-x10' => [
                ['account_id' => 'account-x10', 'product' => 'Chair'],
                ['account_id' => 'account-x10', 'product' => 'Bookcase'],
            ],
            'account-x11' => [
                ['account_id' => 'account-x11', 'product' => 'Desk'],
            ],
        ]
    */

除了傳入字串的`鍵`之外，你也可以傳入回呼函式。該函式應該回傳你希望用來分組的鍵的值。

    $grouped = $collection->groupBy(function ($item, $key) {
        return substr($item['account_id'], -3);
    });
    
    $grouped->toArray();
    
    /*
        [
            'x10' => [
                ['account_id' => 'account-x10', 'product' => 'Chair'],
                ['account_id' => 'account-x10', 'product' => 'Bookcase'],
            ],
            'x11' => [
                ['account_id' => 'account-x11', 'product' => 'Desk'],
            ],
        ]
    */

<a name="method-has"></a>
#### `has()` {#collection-method}

`has` 方法用來確認集合中是否含有給定的鍵：

    $collection = collect(['account_id' => 1, 'product' => 'Desk']);

    $collection->has('product');

    // true

<a name="method-implode"></a>
#### `implode()` {#collection-method}

`implode` 方法連接集合中的項目。它的參數依集合中的項目類型而定。假如集合含有陣列或物件，你應該傳入你希望連接的屬性的鍵，以及你希望放在數值之間的「黏著」字串：

    $collection = collect([
        ['account_id' => 1, 'product' => 'Desk'],
        ['account_id' => 2, 'product' => 'Chair'],
    ]);
    
    $collection->implode('product', ', ');
    
    // Desk, Chair

假如集合只含有簡單的字串或數字，就只要傳入黏著字串作該方法唯一的參數：

    collect([1, 2, 3, 4, 5])->implode('-');

    // '1-2-3-4-5'

<a name="method-intersect"></a>
#### `intersect()` {#collection-method}

`intersect` 方法移除任何給定`陣列`或集合內所沒有的數值。

    $collection = collect(['Desk', 'Sofa', 'Chair']);

    $intersect = $collection->intersect(['Desk', 'Chair', 'Bookcase']);

    $intersect->all();

    // [0 => 'Desk', 2 => 'Chair']

<a name="method-intersectbykeys"></a>
#### `intersectByKeys()` {#collection-method}

`intersectByKeys` 方法從原始集合中移除任何給定`陣列`或集合內所沒有的鍵值。

    $collection = collect([
        'serial' => 'UX301', 'type' => 'screen', 'year' => 2009
    ]);
    
    $intersect = $collection->intersectByKeys([
        'reference' => 'UX404', 'type' => 'tab', 'year' => 2011
    ]);
    
    $intersect->all();
    
    // ['type' => 'screen', 'year' => 2009]

<a name="method-isempty"></a>
#### `isEmpty()` {#collection-method}

假如集合是空的，`isEmpty` 方法會回傳 `true`，否則回傳 `false`：

    collect([])->isEmpty();

    // true

<a name="method-isnotempty"></a>
#### `isNotEmpty()` {#collection-method}

假如集合不是空的，`isNotEmpty` 方法會回傳 `true`，否則回傳 `false`：

    collect([])->isNotEmpty();

    // false

<a name="method-keyby"></a>
#### `keyBy()` {#collection-method}

`keyBy`方法以給定鍵的值作為集合項目的鍵。如果有多個元素有相同的鍵，則會把最後的值當成新集合的鍵：

    $collection = collect([
        ['product_id' => 'prod-100', 'name' => 'Desk'],
        ['product_id' => 'prod-200', 'name' => 'Chair'],
    ]);
    
    $keyed = $collection->keyBy('product_id');
    
    $keyed->all();
    
    /*
        [
            'prod-100' => ['product_id' => 'prod-100', 'name' => 'Desk'],
            'prod-200' => ['product_id' => 'prod-200', 'name' => 'Chair'],
        ]
    */

你也可以傳入自己的回呼函式，該函式應該回傳集合的鍵的值：

    $keyed = $collection->keyBy(function ($item) {
        return strtoupper($item['product_id']);
    });
    
    $keyed->all();
    
    /*
        [
            'PROD-100' => ['product_id' => 'prod-100', 'name' => 'Desk'],
            'PROD-200' => ['product_id' => 'prod-200', 'name' => 'Chair'],
        ]
    */

<a name="method-keys"></a>
#### `keys()` {#collection-method}

`keys` 方法回傳該集合所有的鍵：

    $collection = collect([
        'prod-100' => ['product_id' => 'prod-100', 'name' => 'Desk'],
        'prod-200' => ['product_id' => 'prod-200', 'name' => 'Chair'],
    ]);
    
    $keys = $collection->keys();
    
    $keys->all();
    
    // ['prod-100', 'prod-200']

<a name="method-last"></a>
#### `last()` {#collection-method}

`last` 方法回傳集合中，最後一個通過給定測試的元素：

    collect([1, 2, 3, 4])->last(function ($value, $key) {
        return $value < 3;
    });
    
    // 2

你也可以不傳入參數使用 `last` 方法以取得集合中最後一個元素。如果集合是空的，則會回傳 `null`：

    collect([1, 2, 3, 4])->last();

    // 4

<a name="method-macro"></a>
#### `macro()` {#collection-method}

靜態的 `macro` 方法，允許你在執行期間動態加入 `Collection` 類別內。可參考 [extending collections](#extending-collections) 章節獲得更多資訊。

<a name="method-make"></a>
#### `make()` {#collection-method}

靜態的 `make` 方法，可產生新的集合，見 [建立集合](#creating-collections) 章節。

<a name="method-map"></a>
#### `map()` {#collection-method}

`map` 方法遍歷整個集合並將每一個數值傳入給定的回呼函式。回呼函式可以任意修改並回傳項目，從而形成一個修改過的新項目集合：

    $collection = collect([1, 2, 3, 4, 5]);

    $multiplied = $collection->map(function ($item, $key) {
        return $item * 2;
    });
    
    $multiplied->all();
    
    // [2, 4, 6, 8, 10]

> {note} 如同大多數的集合方法一樣，`map` 回傳一個新集合實例。它並沒有修改到原本的集合。假如你想改變原始的集合，得使用 [`transform`](#method-transform) 方法。

<a name="method-mapinto"></a>
#### `mapInto()` {#collection-method}

`mapInto()` 方法遍歷整個集合，並將每個值傳入指定類別實體的建構子中：

    class Currency
    {
        /**
         * Create a new currency instance.
         *
         * @param  string  $code
         * @return void
         */
        function __construct(string $code)
        {
            $this->code = $code;
        }
    }
    
    $collection = collect(['USD', 'EUR', 'GBP']);
    
    $currencies = $collection->mapInto(Currency::class);
    
    $currencies->all();
    
    // [Currency('USD'), Currency('EUR'), Currency('GBP')]

<a name="method-mapspread"></a>
#### `mapSpread()` {#collection-method}

`mapSpread` 方法會去遍歷整個集合，並將下一層的巢狀陣列傳入指定的回呼函式，回呼函式可以任意修改並回傳項目，從而形成一個修改過的新項目集合：

    $collection = collect([0, 1, 2, 3, 4, 5, 6, 7, 8, 9]);

    $chunks = $collection->chunk(2);

    $sequence = $chunks->mapSpread(function ($odd, $even) {
        return $odd + $even;
    });
    
    $sequence->all();
    
    // [1, 5, 9, 13, 17]

<a name="method-maptogroups"></a>
#### `mapToGroups()` {#collection-method}

`mapToGroups` 方法依據回呼函式將各項目分組。該回呼函式會回傳有關聯的鍵值對陣列，最後組成新的集合。

    $collection = collect([
        [
            'name' => 'John Doe',
            'department' => 'Sales',
        ],
        [
            'name' => 'Jane Doe',
            'department' => 'Sales',
        ],
        [
            'name' => 'Johnny Doe',
            'department' => 'Marketing',
        ]
    ]);
    
    $grouped = $collection->mapToGroups(function ($item, $key) {
        return [$item['department'] => $item['name']];
    });
    
    $grouped->toArray();
    
    /*
        [
            'Sales' => ['John Doe', 'Jane Doe'],
            'Marketing' => ['Johhny Doe'],
        ]
    */
    
    $grouped->get('Sales')->all();
    
    // ['John Doe', 'Jane Doe']

<a name="method-mapwithkeys"></a>
#### `mapWithKeys()` {#collection-method}

`mapWithKeys` 方法遍歷整個集合並將每個數值（包含鍵）傳進回呼函式。該回呼函式會回傳只包含一對鍵與值的組合：

    $collection = collect([
        [
            'name' => 'John',
            'department' => 'Sales',
            'email' => 'john@example.com'
        ],
        [
            'name' => 'Jane',
            'department' => 'Marketing',
            'email' => 'jane@example.com'
        ]
    ]);
    
    $keyed = $collection->mapWithKeys(function ($item) {
        return [$item['email'] => $item['name']];
    });
    
    $keyed->all();
    
    /*
        [
            'john@example.com' => 'John',
            'jane@example.com' => 'Jane',
        ]
    */

<a name="method-max"></a>
#### `max()` {#collection-method}

`max` 方法會回傳給定鍵的最大值：

    $max = collect([['foo' => 10], ['foo' => 20]])->max('foo');

    // 20

    $max = collect([1, 2, 3, 4, 5])->max();

    // 5

<a name="method-median"></a>
#### `median()` {#collection-method}

`median` 方法會回傳給定鍵的[中間值](https://en.wikipedia.org/wiki/Median)：

    $median = collect([['foo' => 10], ['foo' => 10], ['foo' => 20], ['foo' => 40]])->median('foo');

    // 15

    $median = collect([1, 1, 2, 4])->median();

    // 1.5

<a name="method-merge"></a>
#### `merge()` {#collection-method}

`merge` 方法將給定的陣列合併進集合。陣列字串鍵與集合字串鍵相同的將會覆蓋掉原始集合的數值：

    $collection = collect(['product_id' => 1, 'price' => 100]);

    $merged = $collection->merge(['price' => 200, 'discount' => false]);

    $merged->all();

    // ['product_id' => 1, 'price' => 200, 'discount' => false]

如果給定陣列的鍵為數字，則數值將會附加在集合的後面：

    $collection = collect(['Desk', 'Chair']);

    $merged = $collection->merge(['Bookcase', 'Door']);

    $merged->all();

    // ['Desk', 'Chair', 'Bookcase', 'Door']

<a name="method-min"></a>
#### `min()` {#collection-method}

`min` 方法會回傳給定鍵的最小值：

    $min = collect([['foo' => 10], ['foo' => 20]])->min('foo');

    // 10

    $min = collect([1, 2, 3, 4, 5])->min();

    // 1

<a name="method-mode"></a>
#### `mode()` {#collection-method}

`mode` 方法會回傳給定鍵中[最常出現的數值(mode value)](https://en.wikipedia.org/wiki/Mode_(statistics))：

    $mode = collect([['foo' => 10], ['foo' => 10], ['foo' => 20], ['foo' => 40]])->mode('foo');

    // [10]

    $mode = collect([1, 1, 2, 4])->mode();

    // [1]

<a name="method-nth"></a>
#### `nth()` {#collection-method}

`nth` 方法會產生一個包含第n個倍數的值的新集合。

    $collection = collect(['a', 'b', 'c', 'd', 'e', 'f']);

    $collection->nth(4);

    // ['a', 'e']

你可以選擇性的傳入第二個參數做偏移：

    $collection->nth(4, 1);

    // ['b', 'f']

<a name="method-only"></a>
#### `only()` {#collection-method}

`only` 方法回傳集合中指定鍵的所有項目：

    $collection = collect(['product_id' => 1, 'name' => 'Desk', 'price' => 100, 'discount' => false]);

    $filtered = $collection->only(['product_id', 'name']);

    $filtered->all();

    // ['product_id' => 1, 'name' => 'Desk']

與 `only` 相反的方法請查看 [except](#method-only)。

<a name="method-pad"></a>
#### `pad()` {#collection-method}

`pad` 方法會用給定的值補滿至給定的大小。這方法類似於 PHP 的函式 [array_pad](https://secure.php.net/manual/en/function.array-pad.php)。

如果要往左邊填滿，可以傳入一個負數大小。若給定的大小小於等於目前陣列的大小，則不會有任何填充：

    $collection = collect(['A', 'B', 'C']);

    $filtered = $collection->pad(5, 0);

    $filtered->all();

    // ['A', 'B', 'C', 0, 0]

    $filtered = $collection->pad(-5, 0);

    $filtered->all();

    // [0, 0, 'A', 'B', 'C']

<a name="method-partition"></a>
#### `partition()` {#collection-method}

`partition` 方法將通過和不通過回呼函式測試的項目分開，並可以跟 PHP 函式 `list` 結合使用。

    $collection = collect([1, 2, 3, 4, 5, 6]);

    list($underThree, $aboveThree) = $collection->partition(function ($i) {
        return $i < 3;
    });

<a name="method-pipe"></a>
#### `pipe()` {#collection-method}

`pipe` 方法將此集合傳入回呼函式中，並回傳其結果：

    $collection = collect([1, 2, 3]);

    $piped = $collection->pipe(function ($collection) {
        return $collection->sum();
    });
    
    // 6

<a name="method-pluck"></a>
#### `pluck()` {#collection-method}

`pluck` 方法取得所有集合中給定鍵的值：

    $collection = collect([
        ['product_id' => 'prod-100', 'name' => 'Desk'],
        ['product_id' => 'prod-200', 'name' => 'Chair'],
    ]);
    
    $plucked = $collection->pluck('name');
    
    $plucked->all();
    
    // ['Desk', 'Chair']

你也可以指定要怎麼給最後出來的集合分配鍵：

    $plucked = $collection->pluck('name', 'product_id');

    $plucked->all();

    // ['prod-100' => 'Desk', 'prod-200' => 'Chair']

<a name="method-pop"></a>
#### `pop()` {#collection-method}

`pop` 方法移除並回傳集合最後一個項目：

    $collection = collect([1, 2, 3, 4, 5]);

    $collection->pop();

    // 5

    $collection->all();

    // [1, 2, 3, 4]

<a name="method-prepend"></a>
#### `prepend()` {#collection-method}

`prepend` 方法在集合前面增加一個項目：

    $collection = collect([1, 2, 3, 4, 5]);

    $collection->prepend(0);

    $collection->all();

    // [0, 1, 2, 3, 4, 5]

你可以傳遞選擇性的第二個參數來設置前置項目的鍵：

    $collection = collect(['one' => 1, 'two' => 2]);

    $collection->prepend(0, 'zero');

    $collection->all();

    // ['zero' => 0, 'one' => 1, 'two' => 2]

<a name="method-pull"></a>
#### `pull()` {#collection-method}

`pull` 方法以鍵從集合中移除並回傳一個項目：

    $collection = collect(['product_id' => 'prod-100', 'name' => 'Desk']);

    $collection->pull('name');

    // 'Desk'

    $collection->all();

    // ['product_id' => 'prod-100']

<a name="method-push"></a>
#### `push()` {#collection-method}

`push` 方法附加一個項目到集合後面：

    $collection = collect([1, 2, 3, 4]);

    $collection->push(5);

    $collection->all();

    // [1, 2, 3, 4, 5]

<a name="method-put"></a>
#### `put()` {#collection-method}

`put` 在集合內設定一個給定鍵和數值：

    $collection = collect(['product_id' => 1, 'name' => 'Desk']);

    $collection->put('price', 100);

    $collection->all();

    // ['product_id' => 1, 'name' => 'Desk', 'price' => 100]

<a name="method-random"></a>
#### `random()` {#collection-method}

`random` 方法從集合中隨機回傳一個項目：

    $collection = collect([1, 2, 3, 4, 5]);

    $collection->random();

    // 4 - (retrieved randomly)

你可以選擇性的傳入一個整數，以指定要回傳幾個隨機元素，並會以集合的型態回傳。

    $random = $collection->random(3);

    $random->all();

    // [2, 4, 5] - (retrieved randomly)

<a name="method-reduce"></a>
#### `reduce()` {#collection-method}

`reduce` 方法將集合縮減到單一數值，該方法會將每次迭代的結果傳入到下一次迭代：

    $collection = collect([1, 2, 3]);

    $total = $collection->reduce(function ($carry, $item) {
        return $carry + $item;
    });
    
    // 6

第一次迭代時 `$carry` 的數值為 `null`；然而你也可以傳入第二個參數進 `reduce` 以指定它的初始值：

    $collection->reduce(function ($carry, $item) {
        return $carry + $item;
    }, 4);
    
    // 10

<a name="method-reject"></a>
#### `reject()` {#collection-method}

`reject` 方法以給定的回呼函式篩選集合。該回呼函式應該對希望從最終集合移除掉的項目回傳 `true`：

    $collection = collect([1, 2, 3, 4]);

    $filtered = $collection->reject(function ($value, $key) {
        return $value > 2;
    });
    
    $filtered->all();
    
    // [1, 2]

與 `reject` 相對的方法可以檢視 [`filter`](#method-filter) 方法。

<a name="method-reverse"></a>
#### `reverse()` {#collection-method}

`reverse` 方法倒轉集合內項目的順序：

    $collection = collect([1, 2, 3, 4, 5]);

    $reversed = $collection->reverse();

    $reversed->all();

    // [5, 4, 3, 2, 1]

<a name="method-search"></a>
#### `search()` {#collection-method}

`search` 方法在集合內搜尋給定的數值並回傳找到的鍵。假如找不到項目，則回傳 `false`：

    $collection = collect([2, 4, 6, 8]);

    $collection->search(4);

    // 1

搜尋是用「寬鬆」比對來進行。這意味著字串型態的數字會等於有相同數值的數字。要使用嚴格比對，就傳入 `true` 為該方法的第二個參數：

    $collection->search('4', true);

    // false

另外，你可以傳入你自己的回呼函式來搜尋第一個通過你判斷測試的項目：

    $collection->search(function ($item, $key) {
        return $item > 5;
    });
    
    // 2

<a name="method-shift"></a>
#### `shift()` {#collection-method}

`shift` 方法移除並回傳集合的第一個項目：

    $collection = collect([1, 2, 3, 4, 5]);

    $collection->shift();

    // 1

    $collection->all();

    // [2, 3, 4, 5]

<a name="method-shuffle"></a>
#### `shuffle()` {#collection-method}

`shuffle` 方法隨機排序集合的項目：

    $collection = collect([1, 2, 3, 4, 5]);

    $shuffled = $collection->shuffle();

    $shuffled->all();

    // [3, 2, 5, 1, 4] - (generated randomly)

<a name="method-slice"></a>
#### `slice()` {#collection-method}

`slice` 方法回傳集合從給定索引開始的一部分切片：

    $collection = collect([1, 2, 3, 4, 5, 6, 7, 8, 9, 10]);

    $slice = $collection->slice(4);

    $slice->all();

    // [5, 6, 7, 8, 9, 10]

如果你想限制回傳切片的大小，就傳入想要的大小為方法的第二個參數：

    $slice = $collection->slice(4, 2);

    $slice->all();

    // [5, 6]

回傳的切片預設將會保留原有的鍵。如果你不想保留，可使用 [`values`](#method-values) 方法重設為連續的數字索引。

<a name="method-sort"></a>
#### `sort()` {#collection-method}

`sort` 方法對集合排序。排序過的集合保有原來的陣列鍵。在這個例子中我們用了 [`values`](#method-values) 方法重設鍵為連續的數字索引。

    $collection = collect([5, 3, 1, 2, 4]);

    W$sorted = $collection->sort();

    $sorted->values()->all();

    // [1, 2, 3, 4, 5]

假如你需要更進階的排序，你可以傳入回呼函式以你自己的演算法進行`排序`。參考 PHP 文件的 [`usort`](http://php.net/manual/en/function.usort.php#refsect1-function.usort-parameters)，這是集合的 `sort` 方法在背後所使用的函式。

> {tip} 要排序內含陣列或物件的集合，見 [`sortBy`](#method-sortby) 和 [`sortByDesc`](#method-sortbydesc) 方法。


<a name="method-sortby"></a>
#### `sortBy()` {#collection-method}

`sortBy` 方法以給定的鍵排序集合。排序過的集合保有原來的陣列鍵。在這個例子中我們用了 [`values`](#method-values) 方法重設鍵為連續的數字索引：

    $collection = collect([
        ['name' => 'Desk', 'price' => 200],
        ['name' => 'Chair', 'price' => 100],
        ['name' => 'Bookcase', 'price' => 150],
    ]);
    
    $sorted = $collection->sortBy('price');
    
    $sorted->values()->all();
    
    /*
        [
            ['name' => 'Chair', 'price' => 100],
            ['name' => 'Bookcase', 'price' => 150],
            ['name' => 'Desk', 'price' => 200],
        ]
    */

你也可以傳入自己的回呼函式以決定如何排序集合數值：

    $collection = collect([
        ['name' => 'Desk', 'colors' => ['Black', 'Mahogany']],
        ['name' => 'Chair', 'colors' => ['Black']],
        ['name' => 'Bookcase', 'colors' => ['Red', 'Beige', 'Brown']],
    ]);
    
    $sorted = $collection->sortBy(function ($product, $key) {
        return count($product['colors']);
    });
    
    $sorted->values()->all();
    
    /*
        [
            ['name' => 'Chair', 'colors' => ['Black']],
            ['name' => 'Desk', 'colors' => ['Black', 'Mahogany']],
            ['name' => 'Bookcase', 'colors' => ['Red', 'Beige', 'Brown']],
        ]
    */

<a name="method-sortbydesc"></a>
#### `sortByDesc()` {#collection-method}

這個方法與 [`sortBy`](#method-sortby) 有著一樣的形式，但是會以相反的順序來排序集合。

<a name="method-splice"></a>
#### `splice()` {#collection-method}

`splice` 方法移除並回傳從指定索引開始的所有項目：

    $collection = collect([1, 2, 3, 4, 5]);

    $chunk = $collection->splice(2);

    $chunk->all();

    // [3, 4, 5]

    $collection->all();

    // [1, 2]

你也可以傳入第二個參數以限制大小：

    $collection = collect([1, 2, 3, 4, 5]);

    $chunk = $collection->splice(2, 1);

    $chunk->all();

    // [3]

    $collection->all();

    // [1, 2, 4, 5]

此外，你可以傳入新項目在第三個參數以取代集合中被移除的項目：

    $collection = collect([1, 2, 3, 4, 5]);

    $chunk = $collection->splice(2, 1, [10, 11]);

    $chunk->all();

    // [3]

    $collection->all();

    // [1, 2, 10, 11, 4, 5]

<a name="method-split"></a>
#### `split()` {#collection-method}

`split` 方法將集合分散為給定數個的集合：

    $collection = collect([1, 2, 3, 4, 5]);

    $groups = $collection->split(3);

    $groups->toArray();

    // [[1, 2], [3, 4], [5]]

<a name="method-sum"></a>
#### `sum()` {#collection-method}

`sum` 方法回傳集合內所有項目的總和：

    collect([1, 2, 3, 4, 5])->sum();

    // 15

如果集合包含陣列或物件，你應該傳入一個鍵來確認要用哪些數值來計算總合：

    $collection = collect([
        ['name' => 'JavaScript: The Good Parts', 'pages' => 176],
        ['name' => 'JavaScript: The Definitive Guide', 'pages' => 1096],
    ]);
    
    $collection->sum('pages');
    
    // 1272

此外，你可以傳入自己的回呼函式來決定要用哪些數值來計算總合：

    $collection = collect([
        ['name' => 'Chair', 'colors' => ['Black']],
        ['name' => 'Desk', 'colors' => ['Black', 'Mahogany']],
        ['name' => 'Bookcase', 'colors' => ['Red', 'Beige', 'Brown']],
    ]);
    
    $collection->sum(function ($product) {
        return count($product['colors']);
    });
    
    // 6

<a name="method-take"></a>
#### `take()` {#collection-method}

`take` 方法回傳有著指定數量項目的集合：

    $collection = collect([0, 1, 2, 3, 4, 5]);

    $chunk = $collection->take(3);

    $chunk->all();

    // [0, 1, 2]

你也可以傳入負整數以取得從集合後面來算指定數量的項目：

    $collection = collect([0, 1, 2, 3, 4, 5]);

    $chunk = $collection->take(-2);

    $chunk->all();

    // [4, 5]

<a name="method-tap"></a>
#### `tap()` {#collection-method}

`tap` 方法將集合傳入給定的回呼函式，允許你在適當的時機"引出"裡面的元素，且做任何事都不會影響到原有的集合：

    collect([2, 4, 3, 1, 5])
        ->sort()
        ->tap(function ($collection) {
            Log::debug('Values after sorting', $collection->values()->toArray());
        })
        ->shift();
    
    // 1

<a name="method-times"></a>
#### `times()` {#collection-method}

`times` 這個靜態函式，會以一個參數為次數的回呼函式，創建一個新的集合：

    $collection = Collection::times(10, function ($number) {
        return $number * 9;
    });
    
    $collection->all();
    
    // [9, 18, 27, 36, 45, 54, 63, 72, 81, 90]

這個方法與產生 [Eloquent](/laravel_tw/docs/5.5/eloquent) 模型的工廠結合會非常好用。

    $categories = Collection::times(3, function ($number) {
        return factory(Category::class)->create(['name' => 'Category #'.$number]);
    });
    
    $categories->all();
    
    /*
        [
            ['id' => 1, 'name' => 'Category #1'],
            ['id' => 2, 'name' => 'Category #2'],
            ['id' => 3, 'name' => 'Category #3'],
        ]
    */

<a name="method-toarray"></a>
#### `toArray()` {#collection-method}

`toArray` 方法將集合轉換成純 PHP `陣列`。假如集合的數值是 [Eloquent](/laravel_tw/docs/5.5/eloquent) 模型，也會被轉換成陣列：

    $collection = collect(['name' => 'Desk', 'price' => 200]);

    $collection->toArray();

    /*
        [
            ['name' => 'Desk', 'price' => 200],
        ]
    */

> {note} `toArray` 也會轉換所有內嵌的物件為陣列。假如你希望取得原本的底層陣列，改用 [`all`](#method-all) 方法。

<a name="method-tojson"></a>
#### `toJson()` {#collection-method}

`toJson` 方法將集合轉換成 JSON：

    $collection = collect(['name' => 'Desk', 'price' => 200]);

    $collection->toJson();

    // '{"name":"Desk", "price":200}'

<a name="method-transform"></a>
#### `transform()` {#collection-method}

`transform` 方法遍歷集合並對集合內每一個項目呼叫給定的回呼函式。集合的項目將會被回呼函式回傳的數值取代掉：

    $collection = collect([1, 2, 3, 4, 5]);

    $collection->transform(function ($item, $key) {
        return $item * 2;
    });
    
    $collection->all();
    
    // [2, 4, 6, 8, 10]

> {note} 與大多數其他集合的方法不同，`transform` 會修改集合本身。如果你希望建立新集合，就改用 [`map`](#method-map) 方法。

<a name="method-union"></a>
#### `union()` {#collection-method}

`union` 方法會將傳入的陣列加進集合中。如果傳入陣列索引跟原始集合重複，則會優先使用原始集合的值：

    $collection = collect([1 => ['a'], 2 => ['b']]);

    $union = $collection->union([3 => ['c'], 1 => ['b']]);

    $union->all();

    // [1 => ['a'], 2 => ['b'], 3 => ['c']]

<a name="method-unique"></a>
#### `unique()` {#collection-method}

`unique` 方法回傳集合中所有獨特的項目。排序過的集合保有原來的陣列鍵，所以在這個例子中我們用了 [`values`](#method-values) 方法重設鍵為連續的數字索引：

    $collection = collect([1, 1, 2, 2, 3, 4, 2]);

    $unique = $collection->unique();

    $unique->values()->all();

    // [1, 2, 3, 4]

當處理內嵌的陣列或物件，你可以指定用來決定獨特性的鍵：

    $collection = collect([
        ['name' => 'iPhone 6', 'brand' => 'Apple', 'type' => 'phone'],
        ['name' => 'iPhone 5', 'brand' => 'Apple', 'type' => 'phone'],
        ['name' => 'Apple Watch', 'brand' => 'Apple', 'type' => 'watch'],
        ['name' => 'Galaxy S6', 'brand' => 'Samsung', 'type' => 'phone'],
        ['name' => 'Galaxy Gear', 'brand' => 'Samsung', 'type' => 'watch'],
    ]);
    
    $unique = $collection->unique('brand');
    
    $unique->values()->all();
    
    /*
        [
            ['name' => 'iPhone 6', 'brand' => 'Apple', 'type' => 'phone'],
            ['name' => 'Galaxy S6', 'brand' => 'Samsung', 'type' => 'phone'],
        ]
    */

你可以傳入自己的回呼函式來決定項目的獨特性：

    $unique = $collection->unique(function ($item) {
        return $item['brand'].$item['type'];
    });
    
    $unique->values()->all();
    
    /*
        [
            ['name' => 'iPhone 6', 'brand' => 'Apple', 'type' => 'phone'],
            ['name' => 'Apple Watch', 'brand' => 'Apple', 'type' => 'watch'],
            ['name' => 'Galaxy S6', 'brand' => 'Samsung', 'type' => 'phone'],
            ['name' => 'Galaxy Gear', 'brand' => 'Samsung', 'type' => 'watch'],
        ]
    */

`unique` 方法是使用「寬鬆」比對來進行，這意味著字串型態的數字會等於有相同數值的數字。可使用 [`uniqueStrict`](#method-uniquestrict) 方法做嚴格比對。

<a name="method-uniquestrict"></a>
#### `uniqueStrict()` {#collection-method}

此方法的特徵都與 [`unique`](#method-unique) 方法相同，但所有數值將會以嚴格模式來比較。

<a name="method-unless"></a>
#### `unless()` {#collection-method}

`unless` 方法會在第一個參數為 `true` 時執行回呼函式：

    $collection = collect([1, 2, 3]);

    $collection->unless(true, function ($collection) {
        return $collection->push(4);
    });
    
    $collection->unless(false, function ($collection) {
        return $collection->push(5);
    });
    
    $collection->all();
    
    // [1, 2, 3, 5]

與 `unless` 相對的方法可檢視[`when`](#method-when)。

<a name="method-unwrap"></a>
#### `unwrap()` {#collection-method}

`unwrap` 靜態方法會回以最適當的值回傳其最根本的元素。

    Collection::unwrap(collect('John Doe'));

    // ['John Doe']

    Collection::unwrap(['John Doe']);

    // ['John Doe']

    Collection::unwrap('John Doe');

    // 'John Doe'

<a name="method-values"></a>
#### `values()` {#collection-method}

`values` 方法回傳鍵重設為連續整數的的新集合：

    $collection = collect([
        10 => ['product' => 'Desk', 'price' => 200],
        11 => ['product' => 'Desk', 'price' => 200]
    ]);
    
    $values = $collection->values();
    
    $values->all();
    
    /*
        [
            0 => ['product' => 'Desk', 'price' => 200],
            1 => ['product' => 'Desk', 'price' => 200],
        ]
    */

<a name="method-when"></a>
#### `when()` {#collection-method}

`when` 會在第一個參數為 `true` 時執行回呼函式：

    $collection = collect([1, 2, 3]);

    $collection->when(true, function ($collection) {
        return $collection->push(4);
    });
    
    $collection->when(false, function ($collection) {
        return $collection->push(5);
    });
    
    $collection->all();
    
    // [1, 2, 3, 4]

與 `when` 相對的方法可檢視 [`unless`](#method-unless)。

<a name="method-where"></a>
#### `where()` {#collection-method}

`where` 方法以一對給定的鍵／數值篩選集合：

    $collection = collect([
        ['product' => 'Desk', 'price' => 200],
        ['product' => 'Chair', 'price' => 100],
        ['product' => 'Bookcase', 'price' => 150],
        ['product' => 'Door', 'price' => 100],
    ]);
    
    $filtered = $collection->where('price', 100);
    
    $filtered->all();
    
    /*
        [
            ['product' => 'Chair', 'price' => 100],
            ['product' => 'Door', 'price' => 100],
        ]
    */

`where` 方法是用「寬鬆」比對來進行。這意味著字串型態的數字會等於有相同數值的數字。可使用 [`whereStrict`](#method-wherestrict)方法做嚴格比對。

<a name="method-wherestrict"></a>
#### `whereStrict()` {#collection-method}

這方法與 [`where`](#method-where) 方法有相同的行為，但都是以嚴格模式比對所有值。

<a name="method-wherein"></a>
#### `whereIn()` {#collection-method}

`whereIn` 方法以一對給定的鍵／多個數值篩選集合：

    $collection = collect([
        ['product' => 'Desk', 'price' => 200],
        ['product' => 'Chair', 'price' => 100],
        ['product' => 'Bookcase', 'price' => 150],
        ['product' => 'Door', 'price' => 100],
    ]);
    
    $filtered = $collection->whereIn('price', [150, 200]);
    
    $filtered->all();
    
    /*
        [
            ['product' => 'Bookcase', 'price' => 150],
            ['product' => 'Desk', 'price' => 200],
        ]
    */

`whereIn` 方法是用「寬鬆」比對來進行。這意味著字串型態的數字會等於有相同數值的數字。可使用 [`whereInStrict`](#method-whereinstrict)方法做嚴格比對。

<a name="method-whereinstrict"></a>
#### `whereInStrict()` {#collection-method}

這方法與 [`whereIn`](#method-wherein) 方法有相同的行為，但都是以嚴格模式比對所有值。

<a name="method-wherenotin"></a>
#### `whereNotIn()` {#collection-method}

`whereIn` 方法以給定的鍵／多個數值篩選不存在在集合內的值：

    $collection = collect([
        ['product' => 'Desk', 'price' => 200],
        ['product' => 'Chair', 'price' => 100],
        ['product' => 'Bookcase', 'price' => 150],
        ['product' => 'Door', 'price' => 100],
    ]);
    
    $filtered = $collection->whereNotIn('price', [150, 200]);
    
    $filtered->all();
    
    /*
        [
            ['product' => 'Chair', 'price' => 100],
            ['product' => 'Door', 'price' => 100],
        ]
    */

`whereNotIn` 方法是用「寬鬆」比對來進行。這意味著字串型態的數字會等於有相同數值的數字。可使用 [`whereNotInStrict`](#method-wherenotinstrict)方法做嚴格比對。

<a name="method-wherenotinstrict"></a>
#### `whereNotInStrict()` {#collection-method}

這方法與 [`whereNotIn`](#method-wherenotin) 方法有相同的行為，但都是以嚴格模式比對所有值。

<a name="method-wrap"></a>
#### `wrap()` {#collection-method}

`wrap` 靜態方法將給定的值包進集合內：

    $collection = Collection::wrap('John Doe');

    $collection->all();

    // ['John Doe']

    $collection = Collection::wrap(['John Doe']);

    $collection->all();

    // ['John Doe']

    $collection = Collection::wrap(collect('John Doe'));

    $collection->all();

    // ['John Doe']

<a name="method-zip"></a>
#### `zip()` {#collection-method}

`zip` 方法將集合與給定陣列同樣索引的值合併在一起：

    $collection = collect(['Chair', 'Desk']);

    $zipped = $collection->zip([100, 200]);

    $zipped->all();

    // [['Chair', 100], ['Desk', 200]]

<a name="higher-order-messages"></a>
## 高階訊息傳遞

集合也支援「高階訊息傳遞」，這對於集合的一般方法來說是捷徑。集合裡提供高階訊息傳遞的方法有：`average`，`avg`，`contains`，`each`，`every`，`filter`，`first`，`flatMap`，`map`，`partition`，`reject`，`sortBy`，`sortByDesc` 和 `sum`。

每個高階訊息傳遞像是存取集合的動態屬性一樣。例如，我們用高階訊息傳遞的 `each` 來讓集合內的每個元素都呼叫一個方法：

    $users = User::where('votes', '>', 500)->get();

    $users->each->markAsVip();

同樣的，我們可以用`sum`高階訊息傳遞去收集集合內使用者的「votes」總數：

    $users = User::where('group', 'Development')->get();

    return $users->sum->votes;
