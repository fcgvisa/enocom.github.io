---
layout: post
title: "A Brief Intro to Destructuring in Clojure"
categories: clojure
---

While going through the [Clojure Koans](http://clojurekoans.com/) a second time, I was surprised by how simple destructuring is. To make the point clear, here is a short introduction to the DSL.

First, we will start with a basic example. Assuming we have an array of two elements, we can easily associate each element with its own binding.

``` clojure
(defn f [[a b]]
  {:a a
   :b b})

(f ["foo" "bar"]) ;; => {:a "foo", :b "bar"}
```

By declaring the arguments to the function `f` as a nested vector, we are telling Clojure to expect a two element vector as an argument, and to bind the first element to `a` and the second element to `b` within the scope of the function. By then invoking `f` with `["foo" "bar"]`, we see that the result does bind as we expect. Note that if we pass anything less than a two element vector, the respective bindings will simply be `nil`.

``` clojure
(f ["foo"]) ;; => {:a "foo" :b nil}
(f [])      ;; => {:a nil   :b nil}
```

In the case a multi-element vector, we can destructure as many individual elements as we wish all while capturing the rest in a seq.

``` clojure
(defn f [[a b & more]]
  {:a a
   :b b
   :more more})

(f ["foo" "bar" "baz" "quux"])
;; => {:a "foo", :b "bar", :more ("baz" "quux")}
```

As we can see from the output, `:a` and `:b` are just like the example above, but this time the remaining elements of the vector have been collected into a seq. The choice of `more` is entirely arbitrary; any name would be perfectly fine.

An alternate form similar to `& more` above is the use of the `:as` keyword which creates a binding to the original argument, in addition to whatever destructuring we wish to require.

``` clojure
(defn f [[a b :as original-arg]]
  {:a a
   :b b
   :original-arg original-arg})

(f ["foo" "bar"])
;; => {:a "foo", :b "bar", :original-arg ["foo" "bar"]}
```

So far we have covered destructuring of vectors. Lists work much the same. When we reach maps, though, we have a slightly different approach.

Given a map of key value pairs, we can create bindings for the values based on their keys.

``` clojure
(defn f [{a :foo b :bar}]
  {:a a
   :b b})

(f {:foo "my foo" :bar "my bar"})
;; => {:a "my foo", :b "my bar"}
```

By providing a map within the vector of arguments to the function `f`, we tell Clojure to expect a map as an argument, to look up the keys `:foo` and `:bar` in that map, and to bind the associated values of those keys to `a` and `b` respectively. For clarity, I have chosen a local binding which is distinct from the corresponding map key. In common use, though, we will often want to use a local binding which matches the key it is based upon. For example, rather than binding `:foo` to `a`, we might have instead bound `:foo` to `foo`.

Because binding a certain key to a symbol of the same name (e.g., `:foo` to `foo`) is such a common operation, Clojure provides a shorthand syntax.

``` clojure
(defn f [{:keys [foo bar]}]
  {:a foo
   :b bar})

(f {:foo "my foo", :bar "my bar"})
;; => {:a "my foo", :b "my bar"}
```

By using the `:keys` keyword with a vector of key names, Clojure creates the local bindings whose name matches that of the key.

Note that the `:as` keyword works just as we saw above with vectors:

``` clojure
(defn f [{:keys [foo bar] :as original-arg}]
  {:a foo
   :b bar
   :original-arg original-arg})

(f {:foo "my foo", :bar "my bar"})
;; => {:a "my foo", :b "my bar" :original-arg {:foo "my foo", :bar "my bar"}}
```

There are likely more details of destructuring that I have missed here. Nonetheless, the above examples represent the basics of destructuring.
