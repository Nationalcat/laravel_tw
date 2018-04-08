---
layout: post
title: eloquent-collections
tag: 5.4
---
# Eloquent: Collections

- [Introduction](#introduction)
- [Available Methods](#available-methods)
- [Custom Collections](#custom-collections)

<a name="introduction"></a>
## Introduction

All multi-result sets returned by Eloquent are instances of the `Illuminate\Database\Eloquent\Collection` object, including results retrieved via the `get` method or accessed via a relationship. The Eloquent collection object extends the Laravel [base collection](/laravel_tw/docs/5.4/collections), so it naturally inherits dozens of methods used to fluently work with the underlying array of Eloquent models.

Of course, all collections also serve as iterators, allowing you to loop over them as if they were simple PHP arrays:

    $users = App\User::where('active', 1)->get();

    foreach ($users as $user) {
        echo $user->name;
    }

However, collections are much more powerful than arrays and expose a variety of map / reduce operations that may be chained using an intuitive interface. For example, let's remove all inactive models and gather the first name for each remaining user:

    $users = App\User::where('active', 1)->get();

    $names = $users->reject(function ($user) {
        return $user->active === false;
    })
    ->map(function ($user) {
        return $user->name;
    });

> {note} While most Eloquent collection methods return a new instance of an Eloquent collection, the `pluck`, `keys`, `zip`, `collapse`, `flatten` and `flip` methods return a [base collection](/laravel_tw/docs/5.4/collections) instance. Likewise, if a `map` operation returns a collection that does not contain any Eloquent models, it will be automatically cast to a base collection.

<a name="available-methods"></a>
## Available Methods

### The Base Collection

All Eloquent collections extend the base [Laravel collection](/laravel_tw/docs/5.4/collections) object; therefore, they inherit all of the powerful methods provided by the base collection class:

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

[all](/laravel_tw/docs/5.4/collections#method-all)
[average](/laravel_tw/docs/5.4/collections#method-average)
[avg](/laravel_tw/docs/5.4/collections#method-avg)
[chunk](/laravel_tw/docs/5.4/collections#method-chunk)
[collapse](/laravel_tw/docs/5.4/collections#method-collapse)
[combine](/laravel_tw/docs/5.4/collections#method-combine)
[contains](/laravel_tw/docs/5.4/collections#method-contains)
[containsStrict](/laravel_tw/docs/5.4/collections#method-containsstrict)
[count](/laravel_tw/docs/5.4/collections#method-count)
[diff](/laravel_tw/docs/5.4/collections#method-diff)
[diffKeys](/laravel_tw/docs/5.4/collections#method-diffkeys)
[each](/laravel_tw/docs/5.4/collections#method-each)
[every](/laravel_tw/docs/5.4/collections#method-every)
[except](/laravel_tw/docs/5.4/collections#method-except)
[filter](/laravel_tw/docs/5.4/collections#method-filter)
[first](/laravel_tw/docs/5.4/collections#method-first)
[flatMap](/laravel_tw/docs/5.4/collections#method-flatmap)
[flatten](/laravel_tw/docs/5.4/collections#method-flatten)
[flip](/laravel_tw/docs/5.4/collections#method-flip)
[forget](/laravel_tw/docs/5.4/collections#method-forget)
[forPage](/laravel_tw/docs/5.4/collections#method-forpage)
[get](/laravel_tw/docs/5.4/collections#method-get)
[groupBy](/laravel_tw/docs/5.4/collections#method-groupby)
[has](/laravel_tw/docs/5.4/collections#method-has)
[implode](/laravel_tw/docs/5.4/collections#method-implode)
[intersect](/laravel_tw/docs/5.4/collections#method-intersect)
[isEmpty](/laravel_tw/docs/5.4/collections#method-isempty)
[isNotEmpty](/laravel_tw/docs/5.4/collections#method-isnotempty)
[keyBy](/laravel_tw/docs/5.4/collections#method-keyby)
[keys](/laravel_tw/docs/5.4/collections#method-keys)
[last](/laravel_tw/docs/5.4/collections#method-last)
[map](/laravel_tw/docs/5.4/collections#method-map)
[mapWithKeys](/laravel_tw/docs/5.4/collections#method-mapwithkeys)
[max](/laravel_tw/docs/5.4/collections#method-max)
[median](/laravel_tw/docs/5.4/collections#method-median)
[merge](/laravel_tw/docs/5.4/collections#method-merge)
[min](/laravel_tw/docs/5.4/collections#method-min)
[mode](/laravel_tw/docs/5.4/collections#method-mode)
[nth](/laravel_tw/docs/5.4/collections#method-nth)
[only](/laravel_tw/docs/5.4/collections#method-only)
[partition](/laravel_tw/docs/5.4/collections#method-partition)
[pipe](/laravel_tw/docs/5.4/collections#method-pipe)
[pluck](/laravel_tw/docs/5.4/collections#method-pluck)
[pop](/laravel_tw/docs/5.4/collections#method-pop)
[prepend](/laravel_tw/docs/5.4/collections#method-prepend)
[pull](/laravel_tw/docs/5.4/collections#method-pull)
[push](/laravel_tw/docs/5.4/collections#method-push)
[put](/laravel_tw/docs/5.4/collections#method-put)
[random](/laravel_tw/docs/5.4/collections#method-random)
[reduce](/laravel_tw/docs/5.4/collections#method-reduce)
[reject](/laravel_tw/docs/5.4/collections#method-reject)
[reverse](/laravel_tw/docs/5.4/collections#method-reverse)
[search](/laravel_tw/docs/5.4/collections#method-search)
[shift](/laravel_tw/docs/5.4/collections#method-shift)
[shuffle](/laravel_tw/docs/5.4/collections#method-shuffle)
[slice](/laravel_tw/docs/5.4/collections#method-slice)
[sort](/laravel_tw/docs/5.4/collections#method-sort)
[sortBy](/laravel_tw/docs/5.4/collections#method-sortby)
[sortByDesc](/laravel_tw/docs/5.4/collections#method-sortbydesc)
[splice](/laravel_tw/docs/5.4/collections#method-splice)
[split](/laravel_tw/docs/5.4/collections#method-split)
[sum](/laravel_tw/docs/5.4/collections#method-sum)
[take](/laravel_tw/docs/5.4/collections#method-take)
[tap](/laravel_tw/docs/5.4/collections#method-tap)
[toArray](/laravel_tw/docs/5.4/collections#method-toarray)
[toJson](/laravel_tw/docs/5.4/collections#method-tojson)
[transform](/laravel_tw/docs/5.4/collections#method-transform)
[union](/laravel_tw/docs/5.4/collections#method-union)
[unique](/laravel_tw/docs/5.4/collections#method-unique)
[uniqueStrict](/laravel_tw/docs/5.4/collections#method-uniquestrict)
[values](/laravel_tw/docs/5.4/collections#method-values)
[when](/laravel_tw/docs/5.4/collections#method-when)
[where](/laravel_tw/docs/5.4/collections#method-where)
[whereStrict](/laravel_tw/docs/5.4/collections#method-wherestrict)
[whereIn](/laravel_tw/docs/5.4/collections#method-wherein)
[whereInStrict](/laravel_tw/docs/5.4/collections#method-whereinstrict)
[whereNotIn](/laravel_tw/docs/5.4/collections#method-wherenotin)
[whereNotInStrict](/laravel_tw/docs/5.4/collections#method-wherenotinstrict)
[zip](/laravel_tw/docs/5.4/collections#method-zip)

</div>

<a name="custom-collections"></a>
## Custom Collections

If you need to use a custom `Collection` object with your own extension methods, you may override the `newCollection` method on your model:

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

Once you have defined a `newCollection` method, you will receive an instance of your custom collection anytime Eloquent returns a `Collection` instance of that model. If you would like to use a custom collection for every model in your application, you should override the `newCollection` method on a base model class that is extended by all of your models.
