---
layout: post
title: queries
---
# 資料庫：查詢建構器

- [介紹](#introduction)
- [取得結果](#retrieving-results)
    - [分塊結果](#chunking-results)
    - [Aggregates](#aggregates)
- [Selects](#selects)
- [原生表達式](#raw-expressions)
- [Joins](#joins)
- [Unions](#unions)
- [Where 子句](#where-clauses)
    - [參數分組](#parameter-grouping)
    - [Where Exists 子句](#where-exists-clauses)
    - [JSON Where 子句](#json-where-clauses)
- [Ordering、Grouping、Limit 和 Offset](#ordering-grouping-limit-and-offset)
- [Conditional 子句](#conditional-clauses)
- [Inserts](#inserts)
- [Updates](#updates)
    - [更新 JSON 欄位](#updating-json-columns)
    - [遞增與遞減](#increment-and-decrement)
- [Deletes](#deletes)
- [悲觀鎖定](#pessimistic-locking)

<a name="introduction"></a>
## 介紹

Laravel 的資料庫查詢建構器提供一個既方便又優雅的介面來建立和執行資料庫查詢。它能被用於在應用程式中執行大多數資料庫操作，並支援所有資料庫系統。

Laravel 查詢建構器使用 PDO 參數綁定到來保護你的應用程式避免受到 SQL 注入攻擊。不需要在處理要被作為綁定而傳入的字串。

<a name="retrieving-results"></a>
## 取得結果

#### 從資料表中取得所有的資料列

你可以在 `DB` facade 上使用 `table` 方法來開始查詢。`table` 方法為給定的資料表回傳一個優雅的查詢建構器實例，這可以讓你將更多的搜尋條件鏈結到查詢上，並在最後使用 `get` 方法取得結果：

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Support\Facades\DB;
    use App\Http\Controllers\Controller;

    class UserController extends Controller
    {
        /**
         * 顯示應用程式的所有使用者列表。
         *
         * @return Response
         */
        public function index()
        {
            $users = DB::table('users')->get();

            return view('user.index', ['users' => $users]);
        }
    }

`get` 方法回傳一個包含結果的 `Illuminate\Support\Collection` 實例，每個都是 PHP `StdClass` 物件實例的結果。你可以把存取的欄位當作物件的一個屬性來存取每個欄位的值：

    foreach ($users as $user) {
        echo $user->name;
    }

#### 從資料表中存取一筆完整記錄或欄位

如果你只需要從資料庫中取得一筆完整記錄，你可以使用 `first` 方法。這個方法會回傳一個 `StdClass` 物件：

    $user = DB::table('users')->where('name', 'John')->first();

    echo $user->name;

如果你甚至不需要一筆完整記錄，你可以使用 `value` 方法從一筆記錄中抽出一個欄位的值。這個方法會直接回傳該欄位的值。

    $email = DB::table('users')->where('name', 'John')->value('email');

#### 取得欄位值的列表

如果你想要取得一個包含單一欄位的集合，你可以使用 `pluck` 方法。在這個範例中，我們將會取得一個 role 的標題集合：

    $titles = DB::table('roles')->pluck('title');

    foreach ($titles as $title) {
        echo $title;
    }

 你也可以為回傳的集合指定一個自訂鍵的欄位：

    $roles = DB::table('roles')->pluck('title', 'name');

    foreach ($roles as $name => $title) {
        echo $title;
    }

<a name="chunking-results"></a>
### 分塊結果

如果你需要處理上千筆資料庫記錄，請考慮使用 `chunk` 方法。這個方法會在每次取得被分塊的結果的時候，將每個分塊送至 `Closure` 中處理。這個方法對於撰寫處理上千筆記錄的 [Artisan 指令](/laravel_tw/docs/5.5/artisan)來說是相當有用的。例如，讓我們每次以一百記錄為分塊單位的方式來處理整個 `user` 資料表：

    DB::table('users')->orderBy('id')->chunk(100, function ($users) {
        foreach ($users as $user) {
            //
        }
    });

你可以從 `Closure` 中回傳 `false` 來阻止更多的分塊被處理：

    DB::table('users')->orderBy('id')->chunk(100, function ($users) {
        // 處理記錄...

        return false;
    });

<a name="aggregates"></a>
### Aggregates

查詢建構器也提供各種 aggregate 方法，像是 `count`、`max`、`min`、`avg` 和 `sum`。你可以在建構你的查詢後呼叫這些任何方法：

    $users = DB::table('users')->count();

    $price = DB::table('orders')->max('price');

當然，你可以將這些方法與其他查詢子句做組合：

    $price = DB::table('orders')
                    ->where('finalized', 1)
                    ->avg('price');

<a name="selects"></a>
## Selects

#### 指定 Select 子句

當然，你不可能總是想要從資料表選擇所有欄位。使用 `select` 方法，你可以為查詢指定一個自訂的 `select` 子句：

    $users = DB::table('users')->select('name', 'email as user_email')->get();

`distinct` 方法可以讓你的查詢強制回傳不同的結果：

    $users = DB::table('users')->distinct()->get();

如果你已經有一個查詢建構器實例，以及你希望新增一個欄位到現有的 select 子句，請使用 `addSelect` 方法：

    $query = DB::table('users')->select('name');

    $users = $query->addSelect('age')->get();

<a name="raw-expressions"></a>
## 原生表達式

有時你可能需要在查詢中使用原生表達式。要建立一個原生表達式，你可以使用 `DB::raw` 方法：

    $users = DB::table('users')
                         ->select(DB::raw('count(*) as user_count, status'))
                         ->where('status', '<>', 1)
                         ->groupBy('status')
                         ->get();

> {note} 原生陳述句會被作為字串注入到查詢，所以你應該要非常小心，別造成了 SQL 注入漏洞。

<a name="raw-methods"></a>
### 原生方法

除了使用 `DB::raw`，你也可以使用以下的方法來插入原生表達式查詢的各個部分。

#### `selectRaw`

`selectRaw` 方法能被用在 `select(DB::raw(...))` 地方。這個方法接受一組可選的綁定陣列作為它的第二個參數：

    $orders = DB::table('orders')
                    ->selectRaw('price * ? as price_with_tax'), [1.0825])
                    ->get();

#### `whereRaw / orWhereRaw`

`whereRaw` 和 `orWhereRaw` 方法能被用在注入一個原生 `where` 子句到你的查詢。這些方法接受一組可選的綁定陣列作為它們的第二個參數：

    $orders = DB::table('orders')
                    ->whereRaw('price > IF(state = "TX", ?, 100)', [200])
                    ->get();

#### `havingRaw / orHavingRaw`

`havingRaw` 和 `orHavingRaw` 方法可以被用在設定原生字串作為 `having` 子句的值：

    $orders = DB::table('orders')
                    ->select('department', DB::raw('SUM(price) as total_sales'))
                    ->groupBy('department')
                    ->havingRaw('SUM(price) > 2500')
                    ->get();

#### `orderByRaw`

`orderByRaw` 方法可被用於設定一個原生字串作為 `order by` 子句的值：

    $orders = DB::table('orders')
                    ->orderByRaw('updated_at - created_at DESC')
                    ->get();

<a name="joins"></a>
## Joins

#### Inner Join 子句

查詢建構器也可以被用於撰寫 join 陳述句。要處理基本的「inner join」，你可以在查詢建構器實例上使用 `join` 方法。傳入 `join` 方法的第一個參數是你需要連接的資料表名稱，其他參數則指定用以連接的欄位約束。當然，如你所見，你可以在一個查詢中連接多個資料表：

    $users = DB::table('users')
                ->join('contacts', 'users.id', '=', 'contacts.user_id')
                ->join('orders', 'users.id', '=', 'orders.user_id')
                ->select('users.*', 'contacts.phone', 'orders.price')
                ->get();

#### Left Join 子句

如果你想要處理一個「left join」而不是「inner join」，請使用 `leftJoin` 方法。`leftJoin` 方法與 `join` 方法有相同的使用方式：

    $users = DB::table('users')
                ->leftJoin('posts', 'users.id', '=', 'posts.user_id')
                ->get();

#### Cross Join 子句

要處理一個「cross join」，請使用「crossJoin」方法以及要交叉連接的資料表名稱。交叉連接會在第一個資料表與要連接的資料表之間產生一個笛卡爾積（Cartesian Product）：

    $users = DB::table('sizes')
                ->crossJoin('colours')
                ->get();

#### 進階 Join 子句

你可能想要指定更進階的 join 子句，傳送一個 `Closure` 作為傳入 `join` 方法的第二個參數。`Closure` 將會接收一個 `JoinClause` 物件，讓你指定在 `join` 子句上的搜尋條件：

    DB::table('users')
            ->join('contacts', function ($join) {
                $join->on('users.id', '=', 'contacts.user_id')->orOn(...);
            })
            ->get();

如果你想要在你的 join 使用一個「where」風格的子句，你可以在一個 join 上使用 `where` 和 `orWhere` 方法。這些方法不是比較兩個欄位，而是將欄位與一個值進行比較：

    DB::table('users')
            ->join('contacts', function ($join) {
                $join->on('users.id', '=', 'contacts.user_id')
                     ->where('contacts.user_id', '>', 5);
            })
            ->get();

<a name="unions"></a>
## Unions

查詢產生器也提供一個快速的方法來「合併」兩個查詢。例如，你可以建立一個初始查詢，然後使用 `union` 方法將它與第二個查詢合併：

    $first = DB::table('users')
                ->whereNull('first_name');

    $users = DB::table('users')
                ->whereNull('last_name')
                ->union($first)
                ->get();

> {tip} 也可使用 `unionAll` 方法，它和 `union` 有相同的方法署名。

<a name="where-clauses"></a>
## Where 子句

#### 簡易的 Where 子句

你可以在查詢建構器實例上使用 `where` 方法來新增 `where` 子句查詢。最基本的呼叫 `where` 需要三個參數。第一個參數是欄位名稱，第二個參數是一個被資料庫所支援的任何運算子。最後，第三個參數是要對欄位評估的值。

例如，這裡的查詢是在驗證「votes」欄位值是否等於一百：

    $users = DB::table('users')->where('votes', '=', 100)->get();

為了方便起見，若你單純只想驗證某欄位等於一個給定值，你可以直接將這個值作為第二個參數傳入 `where` 方法：

    $users = DB::table('users')->where('votes', 100)->get();

當然，在編寫 `where` 子句時，你可以使用各式其他的運算子：

    $users = DB::table('users')
                    ->where('votes', '>=', 100)
                    ->get();

    $users = DB::table('users')
                    ->where('votes', '<>', 100)
                    ->get();

    $users = DB::table('users')
                    ->where('name', 'like', 'T%')
                    ->get();

你也可以將一組條件陣列傳入 `where` 函式：

    $users = DB::table('users')->where([
        ['status', '=', '1'],
        ['subscribed', '<>', '1'],
    ])->get();

#### Or 陳述句

你可以在查詢中新增 `or` 子句來將 `where` 的搜尋條件鏈結在一起。`orWhere` 方法和 `where` 方法接受相同的參數：

    $users = DB::table('users')
                        ->where('votes', '>', 100)
                        ->orWhere('name', 'John')
                        ->get();

#### 額外的 Where 子句

**whereBetween**

`whereBetween` 方法驗證一個欄位的值介於兩個值之間：

    $users = DB::table('users')
                        ->whereBetween('votes', [1, 100])->get();

**whereNotBetween**

`whereNotBetween` 方法驗證一個欄位的值落在兩個值之外：

    $users = DB::table('users')
                        ->whereNotBetween('votes', [1, 100])
                        ->get();

**whereIn / whereNotIn**

`whereIn` 方法驗證給定欄位的值包含在給定的陣列之內：

    $users = DB::table('users')
                        ->whereIn('id', [1, 2, 3])
                        ->get();

`whereNotIn` 方法驗證給定欄位的值**不**包含在給定的陣列之內：

    $users = DB::table('users')
                        ->whereNotIn('id', [1, 2, 3])
                        ->get();

**whereNull / whereNotNull**

`whereNull` 方法驗證給定㯗位的值為 `NULL`：

    $users = DB::table('users')
                        ->whereNull('updated_at')
                        ->get();

`whereNotNull` 方法驗證該欄位的值不為 `NULL`：

    $users = DB::table('users')
                        ->whereNotNull('updated_at')
                        ->get();

**whereDate / whereMonth / whereDay / whereYear / whereTime**

`whereDate` 方法可被用於針對日期比較欄位的值：

    $users = DB::table('users')
                    ->whereDate('created_at', '2016-12-31')
                    ->get();

`whereMonth` 方法可以被用於針對指定月份比較欄位的值：

    $users = DB::table('users')
                    ->whereMonth('created_at', '12')
                    ->get();

`whereDay` 方法可以被用於針對指定日子比較欄位的值：

    $users = DB::table('users')
                    ->whereDay('created_at', '31')
                    ->get();

`whereYear` 方法可以被用於針對指定年份比較欄位的值：

    $users = DB::table('users')
                    ->whereYear('created_at', '2016')
                    ->get();

`whereTime` 方法可被用於針對指定時間比較欄位的值：

    $users = DB::table('users')
                    ->whereTime('created_at', '=', '11:20')
                    ->get();

**whereColumn**

`whereColumn` 方法可被用於驗證兩個欄位是否相等：

    $users = DB::table('users')
                    ->whereColumn('first_name', 'last_name')
                    ->get();

你也可以傳入比較運算子到該方法：

    $users = DB::table('users')
                    ->whereColumn('updated_at', '>', 'created_at')
                    ->get();

`whereColumn` 方法也能傳入一個多個條件的陣列。這些條件將會使用 `and` 運算子被連接：

    $users = DB::table('users')
                    ->whereColumn([
                        ['first_name', '=', 'last_name'],
                        ['updated_at', '>', 'created_at']
                    ])->get();

<a name="parameter-grouping"></a>
### 參數分組

有時你可能需要建立更進階的 where 子句，像是「where exists」子句或巢狀參數分組。Laravel 的查詢產生器也可處理這些。讓我們看一個在括號中將約束分組的例子：

    DB::table('users')
                ->where('name', '=', 'John')
                ->orWhere(function ($query) {
                    $query->where('votes', '>', 100)
                          ->where('title', '<>', 'Admin');
                })
                ->get();

如你所見，將一個 `Closure` 傳入 `orWhere` 方法，告訴查詢產生器開始一個約束分組。此 `Closure` 接收一個查詢產生器的實例，你可以用它來設定應包含在括號分組中的約束。上面的例子會產生以下的 SQL：

    select * from users where name = 'John' or (votes > 100 and title <> 'Admin')

<a name="where-exists-clauses"></a>
### Where Exists 子句

`whereExists` 方法可以讓你撰寫 `where exists` SQL 子句。`whereExists` 方法接受一個 `Closure` 參數，它會接收查詢產生器實例，讓你可以定義應放在「exists」SQL 子句中的查詢：

    DB::table('users')
                ->whereExists(function ($query) {
                    $query->select(DB::raw(1))
                          ->from('orders')
                          ->whereRaw('orders.user_id = users.id');
                })
                ->get();

上述的查詢會產生以下的 SQL：

    select * from users
    where exists (
        select 1 from orders where orders.user_id = users.id
    )

<a name="json-where-clauses"></a>
### JSON Where 子句

Laravel 也支援在資料庫上查詢 JSON 欄位類型，並提供支援 JSON 欄位類型。目前，這能用在 MySQL 5.7 和 PostgreSQL。要查詢一個 JSON 欄位，請使用 `->` 運算子：

    $users = DB::table('users')
                    ->where('options->language', 'en')
                    ->get();

    $users = DB::table('users')
                    ->where('preferences->dining->meal', 'salad')
                    ->get();

<a name="ordering-grouping-limit-and-offset"></a>
## Ordering、Grouping、Limit 和 Offset

#### orderBy

`orderBy` 方法允許你針對給定的欄位，將查詢結果排序。`orderBy` 的第一個參數應為你要用來排序的欄位，第二個參數則控制排序的方向，可以是 `asc` 或 `desc`：

    $users = DB::table('users')
                    ->orderBy('name', 'desc')
                    ->get();

#### latest / oldest

`latest` and `oldest` 方法可以讓你輕易的依照日期排列結果。預設的結果會依照 `created_at` 欄位來排列。或者，你可以傳入你想要排序方式的欄位名稱：

    $user = DB::table('users')
                    ->latest()
                    ->first();

#### inRandomOrder

`inRandomOrder` 方法可被用於將查詢結果隨機排序。例如，你可以使用這個方法隨機取得一個使用者：

    $randomUser = DB::table('users')
                    ->inRandomOrder()
                    ->first();

#### groupBy / having

`groupBy` 和 `having` 方法可以用來將查詢結果分組。`having` 方法的署名和 `where` 方法的類似：

    $users = DB::table('users')
                    ->groupBy('account_id')
                    ->having('account_id', '>', 100)
                    ->get();

更多進階 `having` 陳述句，請查閱 [`havingRaw`](#raw-methods) 方法。

#### skip / take

要限制查詢所回傳的結果數量，或略過給定數量的查詢結果，你可以使用 `skip` 和 `take` 方法：

    $users = DB::table('users')->skip(10)->take(5)->get();

或者，你可以使用 `limit` 和 `offset` 方法：

    $users = DB::table('users')
                    ->offset(10)
                    ->limit(5)
                    ->get();

<a name="conditional-clauses"></a>
## 條件子句

有時你可能想要只有 else 為 true 的時候使用子句來查詢。例如，如果傳入的請求中出現給定的輸入值，你可能就會想要使用 `where` 陳述句。你可以 `when` 方法來達成：

    $role = $request->input('role');

    $users = DB::table('users')
                    ->when($role, function ($query) use ($role) {
                        return $query->where('role_id', $role);
                    })
                    ->get();


`when` 方法只有第一個參數為 `true` 時才執行給定的 Closure 。如果第一個參數為 `false`，該 Closure 將不會被執行。

你可以將另一個 Closure 作為第三個參數傳入 `when` 方法。這個 Closure 會在第一個參數結果為 `false` 時執行。為了說明如何使用此功能，我們使用它來設定預設的查詢排序：

    $sortBy = null;

    $users = DB::table('users')
                    ->when($sortBy, function ($query) use ($sortBy) {
                        return $query->orderBy($sortBy);
                    }, function ($query) {
                        return $query->orderBy('name');
                    })
                    ->get();


<a name="inserts"></a>
## Inserts

查詢產生器也提供了 `insert` 方法，用來將記錄插入資料表。`insert` 方法接受一組欄位名稱和值得陣列：

    DB::table('users')->insert(
        ['email' => 'john@example.com', 'votes' => 0]
    );

你甚至可以在一次的 `insert` 呼叫中，傳入一個包含陣列的陣列，來插入數筆記錄到資料表裡。每個陣列代表要插入資料表中的一列記錄：

    DB::table('users')->insert([
        ['email' => 'taylor@example.com', 'votes' => 0],
        ['email' => 'dayle@example.com', 'votes' => 0]
    ]);

#### 自動遞增 ID

如果資料表有一個自動遞增的 ID，使用 `insertGetId` 方法來插入一筆記錄，然後取得該 ID：

    $id = DB::table('users')->insertGetId(
        ['email' => 'john@example.com', 'votes' => 0]
    );

> {note} 當使用 PostgreSQL 時，insertGetId 方法預期自動遞增欄位的名稱為 `id`。 若你要從不同的「次序」取得 ID，你可以將欄位名稱作為第二個參數傳入 `insertGetId` 方法。

<a name="updates"></a>
## Updates

當然，除了在資料庫中插入記錄，也可使用 `update` 方法讓查詢產生器更新已存在的記錄。`update` 方法和 `insert` 方法一樣，接受含一對欄位及值的陣列，其中包含要被更新的欄位。你可以使用 `where` 子句來約束 `update` 查詢：

    DB::table('users')
                ->where('id', 1)
                ->update(['votes' => 1]);

<a name="updating-json-columns"></a>
### 更新 JSON 欄位

當你在更新 JSON 欄位時，應該使用 `->` 句法來存取在 JSON 物件中適合的鍵。這個操作只支援有支援 JSON 欄位的資料庫：

    DB::table('users')
                ->where('id', 1)
                ->update(['options->enabled' => true]);

<a name="increment-and-decrement"></a>
### 遞增與遞減

查詢建構器也為遞增與遞減給定欄位的值提供一個方便方法。這個優雅的寫法，提供一個比手動撰寫 `update` 陳述式更具直觀且優雅的介面。

這兩種方法至少都接受一個參數：第一個參數是要修改的欄位。可以選擇傳入第二個參數來控制參數的遞增或遞減數量：

    DB::table('users')->increment('votes');

    DB::table('users')->increment('votes', 5);

    DB::table('users')->decrement('votes');

    DB::table('users')->decrement('votes', 5);

你也可以在操作中指定額外要更新的欄位：

    DB::table('users')->increment('votes', 1, ['name' => 'John']);

<a name="deletes"></a>
## Deletes

透過 `delete` 方法，查詢產生器也可用來將記錄從資料表中刪除：你可以在呼叫 `delete` 方法之前新增 `where` 子句來限制 `delete` 陳述式：

    DB::table('users')->delete();

    DB::table('users')->where('votes', '>', 100)->delete();

若你希望截去整個資料表來移除所有資料列，並將自動遞增 ID 重設為零，你可以使用 `truncate` 方法：

    DB::table('users')->truncate();

<a name="pessimistic-locking"></a>
## 悲觀鎖定

產詢產生器也包含一些函式，用以協助你在 `select` 語法上作「悲觀鎖定」。要以「共享鎖」來執行述句，你可以在查詢上使用 `sharedLock` 方法。共享鎖可避免選擇的資料列被更改，直到你的交易提交為止：

    DB::table('users')->where('votes', '>', 100)->sharedLock()->get();

此外，你可以使用 `lockForUpdate` 方法。「用以更新」鎖可避免資料列被其他共享鎖修改或選取：

    DB::table('users')->where('votes', '>', 100)->lockForUpdate()->get();
