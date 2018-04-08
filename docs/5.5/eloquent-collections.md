---
layout: post
title: eloquent-collections
tag: 5.5
---
# Eloquent: 集合

- [介紹](#introduction)
- [可用的方法](#available-methods)
- [自訂集合](#custom-collections)

<a name="introduction"></a>
## 介紹

Eloquent 回傳的所有多結果集都會是 `Illuminate\Database\Eloquent\Collection` 物件的實例，包括透過 `get` 方法取得或透過關聯存取的結果。Eloquent 集合物件繼承 Laravel [基礎集合](/laravel_tw/docs/5.5/collections)，所以它也自然的繼承了幾十種用於與底層 Eloquent 模型陣列的優雅方法。

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

> {note} 雖然大多數 Eloquent 集合會回傳新的實例，不過只有 `pluck`、`keys`、`zip`、`collapse`、`flatten` 和 `flip` 方法會回傳[基本的集合](/laravel_tw/docs/5.5/collections)實例。同樣的，如果 `map` 操作會傳一個不具有任何 Eloquent 模型的集合，它會自動轉換成基本的集合。

<a name="available-methods"></a>
## 可用的方法

### 基本的集合

所有 Eloquent 集合都繼承了基本的 [Laravel 集合](/laravel_tw/docs/5.5/collections)物件。因此，它們繼承了基本集合類別提供的所有強大方法：

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

[all](/laravel_tw/docs/5.5/collections#method-all)
[average](/laravel_tw/docs/5.5/collections#method-average)
[avg](/laravel_tw/docs/5.5/collections#method-avg)
[chunk](/laravel_tw/docs/5.5/collections#method-chunk)
[collapse](/laravel_tw/docs/5.5/collections#method-collapse)
[combine](/laravel_tw/docs/5.5/collections#method-combine)
[concat](/laravel_tw/docs/5.5/collections#method-concat)
[contains](/laravel_tw/docs/5.5/collections#method-contains)
[containsStrict](/laravel_tw/docs/5.5/collections#method-containsstrict)
[count](/laravel_tw/docs/5.5/collections#method-count)
[crossJoin](/laravel_tw/docs/5.5/collections#method-crossjoin)
[dd](/laravel_tw/docs/5.5/collections#method-dd)
[diff](/laravel_tw/docs/5.5/collections#method-diff)
[diffKeys](/laravel_tw/docs/5.5/collections#method-diffkeys)
[dump](/laravel_tw/docs/5.5/collections#method-dump)
[each](/laravel_tw/docs/5.5/collections#method-each)
[eachSpread](/laravel_tw/docs/5.5/collections#method-eachspread)
[every](/laravel_tw/docs/5.5/collections#method-every)
[except](/laravel_tw/docs/5.5/collections#method-except)
[filter](/laravel_tw/docs/5.5/collections#method-filter)
[first](/laravel_tw/docs/5.5/collections#method-first)
[flatMap](/laravel_tw/docs/5.5/collections#method-flatmap)
[flatten](/laravel_tw/docs/5.5/collections#method-flatten)
[flip](/laravel_tw/docs/5.5/collections#method-flip)
[forget](/laravel_tw/docs/5.5/collections#method-forget)
[forPage](/laravel_tw/docs/5.5/collections#method-forpage)
[get](/laravel_tw/docs/5.5/collections#method-get)
[groupBy](/laravel_tw/docs/5.5/collections#method-groupby)
[has](/laravel_tw/docs/5.5/collections#method-has)
[implode](/laravel_tw/docs/5.5/collections#method-implode)
[intersect](/laravel_tw/docs/5.5/collections#method-intersect)
[isEmpty](/laravel_tw/docs/5.5/collections#method-isempty)
[isNotEmpty](/laravel_tw/docs/5.5/collections#method-isnotempty)
[keyBy](/laravel_tw/docs/5.5/collections#method-keyby)
[keys](/laravel_tw/docs/5.5/collections#method-keys)
[last](/laravel_tw/docs/5.5/collections#method-last)
[map](/laravel_tw/docs/5.5/collections#method-map)
[mapInto](/laravel_tw/docs/5.5/collections#method-mapinto)
[mapSpread](/laravel_tw/docs/5.5/collections#method-mapspread)
[mapToGroups](/laravel_tw/docs/5.5/collections#method-maptogroups)
[mapWithKeys](/laravel_tw/docs/5.5/collections#method-mapwithkeys)
[max](/laravel_tw/docs/5.5/collections#method-max)
[median](/laravel_tw/docs/5.5/collections#method-median)
[merge](/laravel_tw/docs/5.5/collections#method-merge)
[min](/laravel_tw/docs/5.5/collections#method-min)
[mode](/laravel_tw/docs/5.5/collections#method-mode)
[nth](/laravel_tw/docs/5.5/collections#method-nth)
[only](/laravel_tw/docs/5.5/collections#method-only)
[pad](/laravel_tw/docs/5.5/collections#method-pad)
[partition](/laravel_tw/docs/5.5/collections#method-partition)
[pipe](/laravel_tw/docs/5.5/collections#method-pipe)
[pluck](/laravel_tw/docs/5.5/collections#method-pluck)
[pop](/laravel_tw/docs/5.5/collections#method-pop)
[prepend](/laravel_tw/docs/5.5/collections#method-prepend)
[pull](/laravel_tw/docs/5.5/collections#method-pull)
[push](/laravel_tw/docs/5.5/collections#method-push)
[put](/laravel_tw/docs/5.5/collections#method-put)
[random](/laravel_tw/docs/5.5/collections#method-random)
[reduce](/laravel_tw/docs/5.5/collections#method-reduce)
[reject](/laravel_tw/docs/5.5/collections#method-reject)
[reverse](/laravel_tw/docs/5.5/collections#method-reverse)
[search](/laravel_tw/docs/5.5/collections#method-search)
[shift](/laravel_tw/docs/5.5/collections#method-shift)
[shuffle](/laravel_tw/docs/5.5/collections#method-shuffle)
[slice](/laravel_tw/docs/5.5/collections#method-slice)
[sort](/laravel_tw/docs/5.5/collections#method-sort)
[sortBy](/laravel_tw/docs/5.5/collections#method-sortby)
[sortByDesc](/laravel_tw/docs/5.5/collections#method-sortbydesc)
[splice](/laravel_tw/docs/5.5/collections#method-splice)
[split](/laravel_tw/docs/5.5/collections#method-split)
[sum](/laravel_tw/docs/5.5/collections#method-sum)
[take](/laravel_tw/docs/5.5/collections#method-take)
[tap](/laravel_tw/docs/5.5/collections#method-tap)
[toArray](/laravel_tw/docs/5.5/collections#method-toarray)
[toJson](/laravel_tw/docs/5.5/collections#method-tojson)
[transform](/laravel_tw/docs/5.5/collections#method-transform)
[union](/laravel_tw/docs/5.5/collections#method-union)
[unique](/laravel_tw/docs/5.5/collections#method-unique)
[uniqueStrict](/laravel_tw/docs/5.5/collections#method-uniquestrict)
[unless](/laravel_tw/docs/5.5/collections#method-unless)
[values](/laravel_tw/docs/5.5/collections#method-values)
[when](/laravel_tw/docs/5.5/collections#method-when)
[where](/laravel_tw/docs/5.5/collections#method-where)
[whereStrict](/laravel_tw/docs/5.5/collections#method-wherestrict)
[whereIn](/laravel_tw/docs/5.5/collections#method-wherein)
[whereInStrict](/laravel_tw/docs/5.5/collections#method-whereinstrict)
[whereNotIn](/laravel_tw/docs/5.5/collections#method-wherenotin)
[whereNotInStrict](/laravel_tw/docs/5.5/collections#method-wherenotinstrict)
[zip](/laravel_tw/docs/5.5/collections#method-zip)

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
