---
layout: blag
title: Sometimes You Feel Like a Map
tags: clojure map jvm
---
# Sometimes You Feel Like a Map

Creating your own data type that behaves like a map is pretty straight forward. The tricky part is figuring out
which interfaces you need to implement to enable the different parts of Clojure\'s map API. The following is an
overview of what is necessary to make your custom type behave like a map on the JVM.

## `clojure.lang.Associative`

The `Associative` interface gives us some building blocks that we\'ll be able to reuse later on.

{% highlight clojure %}
  Associative
  (containsKey [this key])
  (entryAt [this key])
  (assoc [this key value])
{% endhighlight %}

### Watch Out
  - `containsKey` must return `true` or `false` or something that can be cast to `java.lang.Boolean`, otherwise you\'ll get a `ClassCastException`
  - `entryAt` should return an `IMapEntry` if the key exists, `nil` otherwise. Your best bet is to return a `clojure.lang.MapEntry`.

### API Calls
  - `assoc`
  - `contains?`
  - `find`

## `clojure.lang.ILookup`

Again, some building blocks for later, this is also a great place to reuse the `containsKey` from the `Associative` interface.

{% highlight clojure %}
  ILookup
  (valAt [this k])
  (valAt [this k not-found])
{% endhighlight %}

### API Calls
  - `(:kwd foo)`
  - `(:kwd foo 'default')`
  - `get`
  - `get-in`

## `clojure.lang.IFn`

The `IFn` interface is what allows your map to be called as a function (See API Calls below).

{% highlight clojure %}
  IFn
  (invoke [this k])
  (invoke [this k not-found])
{% endhighlight %}

### API Calls
  - `(your-map :kwd)`
  - `(your-map :kwd 'default')`


By defining `invoke` for one and two parameters, you can use instances of your map as a function, eg `(foo :kwd)`.

## `clojure.lang.IPersistentCollection`

{% highlight clojure %}
  IPersistentCollection
  (cons [this o])
  (count [this])
  (empty [this])
  (equiv [this o])
{% endhighlight %}

### Watch Out
  - `empty` _is not_ the same thing as `empty?`. It should return whatever an "empty" representation of your map is.
  - `equiv` will only apply to comparing other things to your map, but not the other way. Eg. `(= your-map thing)` will hit your implementation of `equiv`, but `(= thing your-map)` will not.

### API Calls
  - `conj`
  - `cons`
  - `count`
  - `empty`
  - `=`

## `clojure.lang.IPersistentMap`

{% highlight clojure %}
  IPersistentMap
  (assoc [this k v])
  (assocEx [this k v])
  (without [this k])
{% endhighlight %}

Having already implemented `Associative`, we get a free pass on this `assoc`. `assocEx` is pretty much the same except that it throws an exception if the key already exists. And without is basically `dissoc`.

### API Calls
  - `assoc`
  - `dissoc`

## `clojure.lang.Seqable`

{% highlight clojure %}
  Seqable
  (seq [this o])
{% endhighlight %}

This one is pretty simple, but it unlocks so much. And remember, `seq` should return `nil` if your map is empty.

### Watch Out
  - Remember, `seq` should return `nil` if your map is empty.

### API Calls
  - `seq`
  - `keys`
  - `vals`
  - `empty?`
  - `map`
  - `some`
  - `filter`
  - `select-keys`
  - etc.

## `java.lang.Iterable`

{% highlight clojure %}
  Iterable
  (iterator [this])
{% endhighlight %}

Seeing as you\'ve already implemented `seq`, this one is a no brainer. You can simply wrap the result of calling `seq` into a `clojure.lang.SeqIterator` and call it a day.

### API Calls
  - `reduce`
  - `reduce-kv`
