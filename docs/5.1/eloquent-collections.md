---
layout: post
title: eloquent-collections
tag: 5.1
---
# Eloquent：集合

- [簡介](#introduction)
- [可用的方法](#available-methods)
- [自訂集合](#custom-collections)

<a name="introduction"></a>
## 簡介

由 Eloquent 回傳的所有多種結果的集合都是一個 `Illuminate\Database\Eloquent\Collection` 物件的實例，包含經由 `get` 方法或是經由存取一個關聯來取得的結果。Eloquent 集合物件繼承了 Laravel [基底集合](/laravel_tw/docs/5.1/collections)，所以它自然繼承了許多可用於流暢地與 Eloquent 模型的底層陣列合作的方法。

當然，所有集合也可以作為迭代器，讓你可以遍歷集合像是一個簡單的 PHP 陣列：

    $users = App\User::where('active', 1)->get();

    foreach ($users as $user) {
        echo $user->name;
    }

然而，集合比陣列更強大的地方是使用各種 map / reduce 直觀的操作。例如，讓我們移除所有閒置的模型和收集其餘每個使用者的名字：

    $users = App\User::where('active', 1)->get();

    $names = $users->reject(function ($user) {
        return $user->active === false;
    })
    ->map(function ($user) {
        return $user->name;
    });

<a name="available-methods"></a>
## 可用的方法

### 基底的集合

所有 Eloquent 集合繼承了基底 [Laravel 集合](/laravel_tw/docs/5.1/collections)物件；因此，他們繼承所有基底集合類別所提供的強大的方法：

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
[all](/laravel_tw/docs/5.1/collections#method-all)
[chunk](/laravel_tw/docs/5.1/collections#method-chunk)
[collapse](/laravel_tw/docs/5.1/collections#method-collapse)
[contains](/laravel_tw/docs/5.1/collections#method-contains)
[count](/laravel_tw/docs/5.1/collections#method-count)
[diff](/laravel_tw/docs/5.1/collections#method-diff)
[each](/laravel_tw/docs/5.1/collections#method-each)
[every](/laravel_tw/docs/5.1/collections#method-every)
[filter](/laravel_tw/docs/5.1/collections#method-filter)
[first](/laravel_tw/docs/5.1/collections#method-first)
[flatten](/laravel_tw/docs/5.1/collections#method-flatten)
[flip](/laravel_tw/docs/5.1/collections#method-flip)
[forget](/laravel_tw/docs/5.1/collections#method-forget)
[forPage](/laravel_tw/docs/5.1/collections#method-forpage)
[get](/laravel_tw/docs/5.1/collections#method-get)
[groupBy](/laravel_tw/docs/5.1/collections#method-groupby)
[has](/laravel_tw/docs/5.1/collections#method-has)
[implode](/laravel_tw/docs/5.1/collections#method-implode)
[intersect](/laravel_tw/docs/5.1/collections#method-intersect)
[isEmpty](/laravel_tw/docs/5.1/collections#method-isempty)
[keyBy](/laravel_tw/docs/5.1/collections#method-keyby)
[keys](/laravel_tw/docs/5.1/collections#method-keys)
[last](/laravel_tw/docs/5.1/collections#method-last)
[map](/laravel_tw/docs/5.1/collections#method-map)
[merge](/laravel_tw/docs/5.1/collections#method-merge)
[pluck](/laravel_tw/docs/5.1/collections#method-pluck)
[pop](/laravel_tw/docs/5.1/collections#method-pop)
[prepend](/laravel_tw/docs/5.1/collections#method-prepend)
[pull](/laravel_tw/docs/5.1/collections#method-pull)
[push](/laravel_tw/docs/5.1/collections#method-push)
[put](/laravel_tw/docs/5.1/collections#method-put)
[random](/laravel_tw/docs/5.1/collections#method-random)
[reduce](/laravel_tw/docs/5.1/collections#method-reduce)
[reject](/laravel_tw/docs/5.1/collections#method-reject)
[reverse](/laravel_tw/docs/5.1/collections#method-reverse)
[search](/laravel_tw/docs/5.1/collections#method-search)
[shift](/laravel_tw/docs/5.1/collections#method-shift)
[shuffle](/laravel_tw/docs/5.1/collections#method-shuffle)
[slice](/laravel_tw/docs/5.1/collections#method-slice)
[sort](/laravel_tw/docs/5.1/collections#method-sort)
[sortBy](/laravel_tw/docs/5.1/collections#method-sortby)
[sortByDesc](/laravel_tw/docs/5.1/collections#method-sortbydesc)
[splice](/laravel_tw/docs/5.1/collections#method-splice)
[sum](/laravel_tw/docs/5.1/collections#method-sum)
[take](/laravel_tw/docs/5.1/collections#method-take)
[toArray](/laravel_tw/docs/5.1/collections#method-toarray)
[toJson](/laravel_tw/docs/5.1/collections#method-tojson)
[transform](/laravel_tw/docs/5.1/collections#method-transform)
[unique](/laravel_tw/docs/5.1/collections#method-unique)
[values](/laravel_tw/docs/5.1/collections#method-values)
[where](/laravel_tw/docs/5.1/collections#method-where)
[whereLoose](/laravel_tw/docs/5.1/collections#method-whereloose)
[zip](/laravel_tw/docs/5.1/collections#method-zip)
</div>

<a name="custom-collections"></a>
## 自訂集合

如果你需要使用一個自訂的 `Collection` 物件與你自己的擴充方法，你可以在模型中覆寫 `newCollection` 方法：

    <?php

    namespace App;

    use App\CustomCollection;
    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * Create a new Eloquent Collection instance.
         *
         * @param  array  $models
         * @return \Illuminate\Database\Eloquent\Collection
         */
        public function newCollection(array $models = [])
        {
            return new CustomCollection($models);
        }
    }

一旦你定義了 `newCollection` 方法，在任何 Eloquent 回傳該模型的 `Collection` 實例的時候，你將會接收到一個你的自訂集合的實例。如果你想要在你的應用程式每個模型中使用自訂的集合，你應該在所有的模型繼承的基底模型中覆寫 `newCollection` 方法。
