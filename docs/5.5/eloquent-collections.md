---
layout: post
title: eloquent-collections
---
# Eloquent: 集合

- [介紹](#introduction)
- [可用的方法](#available-methods)
- [自訂集合](#custom-collections)

<a name="introduction"></a>
## 介紹

Eloquent 回傳的所有多結果集都會是 `Illuminate\Database\Eloquent\Collection` 物件的實例，包括透過 `get` 方法取得或透過關聯存取的結果。Eloquent 集合物件繼承 Laravel [基礎集合](/docs/{{version}}/collections)，所以它也自然的繼承了幾十種用於與底層 Eloquent 模型陣列的優雅方法。

當然，所有的集合也都可以作為迭代器，可以讓你迭代他們就像是簡單的 PHP 陣列：

    $users = App\User::where('active', 1)->get();

    foreach ($users as $user) {
        echo $user->name;
    }

然而，集合比陣列強大許多，並提供 map 或 reduce 等各種鏈結操作的直觀介面。例如，讓我們移除所有不活躍的模型，並收集其餘使用者的名字：

    $users = App\User::where('active', 1)->get();

    $names = $users->reject(function ($user) {
        return $user->active === false;
    })
    ->map(function ($user) {
        return $user->name;
    });

> {note} 雖然大多數 Eloquent 集合會回傳新的實例，不過只有 `pluck`、`keys`、`zip`、`collapse`、`flatten` 和 `flip` 方法會回傳[基本的集合](/docs/{{version}}/collections)實例。同樣的，如果 `map` 操作會傳一個不具有任何 Eloquent 模型的集合，它會自動轉換成基本的集合。

<a name="available-methods"></a>
## 可用的方法

### 基本的集合

所有 Eloquent 集合都繼承了基本的 [Laravel 集合](/docs/{{version}}/collections)物件。因此，它們繼承了基本集合類別提供的所有強大方法：

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

[all](/docs/{{version}}/collections#method-all)
[average](/docs/{{version}}/collections#method-average)
[avg](/docs/{{version}}/collections#method-avg)
[chunk](/docs/{{version}}/collections#method-chunk)
[collapse](/docs/{{version}}/collections#method-collapse)
[combine](/docs/{{version}}/collections#method-combine)
[concat](/docs/{{version}}/collections#method-concat)
[contains](/docs/{{version}}/collections#method-contains)
[containsStrict](/docs/{{version}}/collections#method-containsstrict)
[count](/docs/{{version}}/collections#method-count)
[crossJoin](/docs/{{version}}/collections#method-crossjoin)
[dd](/docs/{{version}}/collections#method-dd)
[diff](/docs/{{version}}/collections#method-diff)
[diffKeys](/docs/{{version}}/collections#method-diffkeys)
[dump](/docs/{{version}}/collections#method-dump)
[each](/docs/{{version}}/collections#method-each)
[eachSpread](/docs/{{version}}/collections#method-eachspread)
[every](/docs/{{version}}/collections#method-every)
[except](/docs/{{version}}/collections#method-except)
[filter](/docs/{{version}}/collections#method-filter)
[first](/docs/{{version}}/collections#method-first)
[flatMap](/docs/{{version}}/collections#method-flatmap)
[flatten](/docs/{{version}}/collections#method-flatten)
[flip](/docs/{{version}}/collections#method-flip)
[forget](/docs/{{version}}/collections#method-forget)
[forPage](/docs/{{version}}/collections#method-forpage)
[get](/docs/{{version}}/collections#method-get)
[groupBy](/docs/{{version}}/collections#method-groupby)
[has](/docs/{{version}}/collections#method-has)
[implode](/docs/{{version}}/collections#method-implode)
[intersect](/docs/{{version}}/collections#method-intersect)
[isEmpty](/docs/{{version}}/collections#method-isempty)
[isNotEmpty](/docs/{{version}}/collections#method-isnotempty)
[keyBy](/docs/{{version}}/collections#method-keyby)
[keys](/docs/{{version}}/collections#method-keys)
[last](/docs/{{version}}/collections#method-last)
[map](/docs/{{version}}/collections#method-map)
[mapInto](/docs/{{version}}/collections#method-mapinto)
[mapSpread](/docs/{{version}}/collections#method-mapspread)
[mapToGroups](/docs/{{version}}/collections#method-maptogroups)
[mapWithKeys](/docs/{{version}}/collections#method-mapwithkeys)
[max](/docs/{{version}}/collections#method-max)
[median](/docs/{{version}}/collections#method-median)
[merge](/docs/{{version}}/collections#method-merge)
[min](/docs/{{version}}/collections#method-min)
[mode](/docs/{{version}}/collections#method-mode)
[nth](/docs/{{version}}/collections#method-nth)
[only](/docs/{{version}}/collections#method-only)
[pad](/docs/{{version}}/collections#method-pad)
[partition](/docs/{{version}}/collections#method-partition)
[pipe](/docs/{{version}}/collections#method-pipe)
[pluck](/docs/{{version}}/collections#method-pluck)
[pop](/docs/{{version}}/collections#method-pop)
[prepend](/docs/{{version}}/collections#method-prepend)
[pull](/docs/{{version}}/collections#method-pull)
[push](/docs/{{version}}/collections#method-push)
[put](/docs/{{version}}/collections#method-put)
[random](/docs/{{version}}/collections#method-random)
[reduce](/docs/{{version}}/collections#method-reduce)
[reject](/docs/{{version}}/collections#method-reject)
[reverse](/docs/{{version}}/collections#method-reverse)
[search](/docs/{{version}}/collections#method-search)
[shift](/docs/{{version}}/collections#method-shift)
[shuffle](/docs/{{version}}/collections#method-shuffle)
[slice](/docs/{{version}}/collections#method-slice)
[sort](/docs/{{version}}/collections#method-sort)
[sortBy](/docs/{{version}}/collections#method-sortby)
[sortByDesc](/docs/{{version}}/collections#method-sortbydesc)
[splice](/docs/{{version}}/collections#method-splice)
[split](/docs/{{version}}/collections#method-split)
[sum](/docs/{{version}}/collections#method-sum)
[take](/docs/{{version}}/collections#method-take)
[tap](/docs/{{version}}/collections#method-tap)
[toArray](/docs/{{version}}/collections#method-toarray)
[toJson](/docs/{{version}}/collections#method-tojson)
[transform](/docs/{{version}}/collections#method-transform)
[union](/docs/{{version}}/collections#method-union)
[unique](/docs/{{version}}/collections#method-unique)
[uniqueStrict](/docs/{{version}}/collections#method-uniquestrict)
[unless](/docs/{{version}}/collections#method-unless)
[values](/docs/{{version}}/collections#method-values)
[when](/docs/{{version}}/collections#method-when)
[where](/docs/{{version}}/collections#method-where)
[whereStrict](/docs/{{version}}/collections#method-wherestrict)
[whereIn](/docs/{{version}}/collections#method-wherein)
[whereInStrict](/docs/{{version}}/collections#method-whereinstrict)
[whereNotIn](/docs/{{version}}/collections#method-wherenotin)
[whereNotInStrict](/docs/{{version}}/collections#method-wherenotinstrict)
[zip](/docs/{{version}}/collections#method-zip)

</div>

<a name="custom-collections"></a>
## 自訂集合

如果你需要使用自己擴充的方法來自訂 `Collection` 物件，可以在你的模型上覆寫 `newCollection` 方法：

    <?php

    namespace App;

    use App\CustomCollection;
    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * 建立一個新的 Eloquent 集合實例。
         *
         * @param  array  $models
         * @return \Illuminate\Database\Eloquent\Collection
         */
        public function newCollection(array $models = [])
        {
            return new CustomCollection($models);
        }
    }

你一旦定義了 `newCollection` 方法，在任何 Eloquent 回傳該模型的 `Collection` 實例的時候，你會接收到一個你的自訂集合的實例。如果你想要在應用程式中為每個模型使用自訂的集合，你應該在所有模型繼承的基本模型類別上覆寫 `newCollection` 方法。
