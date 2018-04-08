---
layout: post
title: eloquent-collections
tag: 5.3
---
# Eloquent: Collections

- [Introduction](#introduction)
- [Available Methods](#available-methods)
- [Custom Collections](#custom-collections)

<a name="introduction"></a>
## Introduction

All multi-result sets returned by Eloquent are instances of the `Illuminate\Database\Eloquent\Collection` object, including results retrieved via the `get` method or accessed via a relationship. The Eloquent collection object extends the Laravel [base collection](/laravel_tw/docs/5.3/collections), so it naturally inherits dozens of methods used to fluently work with the underlying array of Eloquent models.

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

> {note} While most Eloquent collection methods return a new instance of an Eloquent collection, the `pluck`, `keys`, `zip`, `collapse`, `flatten` and `flip` methods return a [base collection](/laravel_tw/docs/5.3/collections) instance. Likewise, if a `map` operation returns a collection that does not contain any Eloquent models, it will be automatically cast to a base collection.

<a name="available-methods"></a>
## Available Methods

### The Base Collection

All Eloquent collections extend the base [Laravel collection](/laravel_tw/docs/5.3/collections) object; therefore, they inherit all of the powerful methods provided by the base collection class:

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

[all](/laravel_tw/docs/5.3/collections#method-all)
[avg](/laravel_tw/docs/5.3/collections#method-avg)
[chunk](/laravel_tw/docs/5.3/collections#method-chunk)
[collapse](/laravel_tw/docs/5.3/collections#method-collapse)
[combine](/laravel_tw/docs/5.3/collections#method-combine)
[contains](/laravel_tw/docs/5.3/collections#method-contains)
[count](/laravel_tw/docs/5.3/collections#method-count)
[diff](/laravel_tw/docs/5.3/collections#method-diff)
[diffKeys](/laravel_tw/docs/5.3/collections#method-diffkeys)
[each](/laravel_tw/docs/5.3/collections#method-each)
[every](/laravel_tw/docs/5.3/collections#method-every)
[except](/laravel_tw/docs/5.3/collections#method-except)
[filter](/laravel_tw/docs/5.3/collections#method-filter)
[first](/laravel_tw/docs/5.3/collections#method-first)
[flatMap](/laravel_tw/docs/5.3/collections#method-flatmap)
[flatten](/laravel_tw/docs/5.3/collections#method-flatten)
[flip](/laravel_tw/docs/5.3/collections#method-flip)
[forget](/laravel_tw/docs/5.3/collections#method-forget)
[forPage](/laravel_tw/docs/5.3/collections#method-forpage)
[get](/laravel_tw/docs/5.3/collections#method-get)
[groupBy](/laravel_tw/docs/5.3/collections#method-groupby)
[has](/laravel_tw/docs/5.3/collections#method-has)
[implode](/laravel_tw/docs/5.3/collections#method-implode)
[intersect](/laravel_tw/docs/5.3/collections#method-intersect)
[isEmpty](/laravel_tw/docs/5.3/collections#method-isempty)
[keyBy](/laravel_tw/docs/5.3/collections#method-keyby)
[keys](/laravel_tw/docs/5.3/collections#method-keys)
[last](/laravel_tw/docs/5.3/collections#method-last)
[map](/laravel_tw/docs/5.3/collections#method-map)
[max](/laravel_tw/docs/5.3/collections#method-max)
[merge](/laravel_tw/docs/5.3/collections#method-merge)
[min](/laravel_tw/docs/5.3/collections#method-min)
[only](/laravel_tw/docs/5.3/collections#method-only)
[pluck](/laravel_tw/docs/5.3/collections#method-pluck)
[pop](/laravel_tw/docs/5.3/collections#method-pop)
[prepend](/laravel_tw/docs/5.3/collections#method-prepend)
[pull](/laravel_tw/docs/5.3/collections#method-pull)
[push](/laravel_tw/docs/5.3/collections#method-push)
[put](/laravel_tw/docs/5.3/collections#method-put)
[random](/laravel_tw/docs/5.3/collections#method-random)
[reduce](/laravel_tw/docs/5.3/collections#method-reduce)
[reject](/laravel_tw/docs/5.3/collections#method-reject)
[reverse](/laravel_tw/docs/5.3/collections#method-reverse)
[search](/laravel_tw/docs/5.3/collections#method-search)
[shift](/laravel_tw/docs/5.3/collections#method-shift)
[shuffle](/laravel_tw/docs/5.3/collections#method-shuffle)
[slice](/laravel_tw/docs/5.3/collections#method-slice)
[sort](/laravel_tw/docs/5.3/collections#method-sort)
[sortBy](/laravel_tw/docs/5.3/collections#method-sortby)
[sortByDesc](/laravel_tw/docs/5.3/collections#method-sortbydesc)
[splice](/laravel_tw/docs/5.3/collections#method-splice)
[sum](/laravel_tw/docs/5.3/collections#method-sum)
[take](/laravel_tw/docs/5.3/collections#method-take)
[toArray](/laravel_tw/docs/5.3/collections#method-toarray)
[toJson](/laravel_tw/docs/5.3/collections#method-tojson)
[transform](/laravel_tw/docs/5.3/collections#method-transform)
[union](/laravel_tw/docs/5.3/collections#method-union)
[unique](/laravel_tw/docs/5.3/collections#method-unique)
[values](/laravel_tw/docs/5.3/collections#method-values)
[where](/laravel_tw/docs/5.3/collections#method-where)
[whereStrict](/laravel_tw/docs/5.3/collections#method-wherestrict)
[whereIn](/laravel_tw/docs/5.3/collections#method-wherein)
[whereInLoose](/laravel_tw/docs/5.3/collections#method-whereinloose)
[zip](/laravel_tw/docs/5.3/collections#method-zip)

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
