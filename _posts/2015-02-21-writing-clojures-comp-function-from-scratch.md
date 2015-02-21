---
layout: post
title: "Writing Clojure's comp Function from Scratch"
categories: clojure
---

One intriguing problem on [4clojure](http://www.4clojure.com) is writing `comp` from scratch. Before we discuss how to do that, let's start by understanding how Clojure's `comp` or "compose" works.

If we wanted to write a function to reverse a list, and then take only the rest of that reversed list, we could write a function to do just that:

``` clojure
(defn reverse-and-take-rest [numbers]
  (rest (reverse numbers)))

(reverse-and-take-rest [1 2 3 4]) ;; => (3 2 1)
```

Using the `comp` function, we have another and more concise way to achieve the same result:

``` clojure
(def reverse-and-take-rest (comp rest reverse))

(reverse-and-take-rest [1 2 3 4]) ;; => (3 2 1)
```

Note that `comp` will work right to left applying each function. In other words, `reverse` is called first and `rest` is called second.

One other important detail is how `comp` behaves when it receives no functions:

``` clojure
((comp) [1 2 3 4]) ;; => [1 2 3 4]
```

In the simplest case with no functions, `comp` will use the `identity` function, handing back whatever it received. We will return to this detail in a moment.

Now that we understand how `comp` works, how would we write it from scratch? Having worked through [The Little Schemer](http://www.ccs.neu.edu/home/matthias/BTLS/), my first inclination is to use recursion with Clojure's `loop` and `recur`. Although idiomatic Clojure code rarely uses `loop`, it's a good place to start.

First, we know how our function should behave:

``` clojure
((my-comp rest reverse) [1 2 3 4]) ;; => (3 2 1)
```

Even though the example above shows two functions, in usage, it will have to support any number of functions. So we'll start by declaring a function which takes a variable number of arguments:

``` clojure
(defn my-comp [& all-fns]
  ;; body here
  )
```

Then, we can put the recursive structure in place using `loop` and `recur`. With each recursive step we will build up a new function while working through the list of functions passed to `my-comp`. I have adopted a convention used in _The Little Schemer_ and refer to the "accumulated function" as `acc-fn`. The usual convention is to have an accumulator which as its name suggests accumulates the results of each recursive call.

``` clojure
(defn my-comp [& all-fns]
  (loop [fns    all-fns
         acc-fn identity]

    ;; if base case, return the accumulated function
    ;; else keep recursing

    (recur)))
```

Recall that the base case of `comp` is to return the `identity` function. Here we have initialized our `acc-fn` to `identity` for the case when `my-comp` is called with no arguments.

With the basic skeleton in place, we can now establish the base case and start thinking about how each recursive step should work.

``` clojure
(defn my-comp [& all-fns]
  (loop [fns    all-fns
         acc-fn identity]

    (if (empty? fns)
      acc-fn

      (recur (butlast fns)
            ;; create an accumulated function here
        ))
```

Once our collection of functions is empty, we know that we're done and can return the resultant `acc-fn`. Because we'll apply the functions from right to left, we will use `butlast` to recurse with all but the last function in the list. The `butlast` function does exactly what its name implies:

``` clojure
(butlast '(rest reverse)) ;; => (rest)
(butlast '(rest))         ;; => nil
```

Note that calling `butlast` on a one-member sequence will return `nil`. "But what about calling `(empty? nil)`," you might ask, as will happen once we have recursed through all our functions and have finally reached the bottom of `my-comp`. Fortunately, Clojure has us covered:

``` clojure
(empty? nil) ;; => true
```

So now that we have established our basic recursive structure with a base case and with a recur step to whittle down our list of functions, how we do build up an accumulated (and composed) function?

First, we know that we need to return a function that takes an argument. Second, we want to call our accumulated function with that argument. And third, we want to pass the result of the function call to our current function. Here's what that looks like:

``` clojure
(defn my-comp [& all-fns]
  (loop [fns    all-fns
         acc-fn identity]

    (if (empty? fns)
      acc-fn

      (recur (butlast fns)
             (fn [a]
               ((last fns) (acc-fn a)))))))
```

And with that, we have a simple but working version of `comp`. The hardest part is thinking about how to build up the accumulated function. It helps to write out all the recursive steps to figure out just how to build up the accumulated function. As we've written `my-comp`, our accumulated function will be as follows:

``` clojure
;; First frame
(fn [a]
  (reverse (identity a)))

;; Second frame
(fn [b]
  (rest
    ((fn [a]
       (reverse (identity a))) b))
```

At this point, it's fairly easy to refactor our code to use `reduce` instead of `loop` and `recur`.

``` clojure
(defn my-comp [& all-fns]
  (reduce
    (fn [acc-fn curr-fn]
      (fn [a]
        (acc-fn (curr-fn a)))
    identity
    all-fns))
```

The `reduce` function takes three arguments: 1) a reducing function, 2) an initial argument passed to that reducing function as the first argument, and 3) a collection to reduce. The reducing function takes two arguments: the accumulated function passed from one step to the next, and an element from the collection. Within the reducing function, we have to do almost exactly what we did in the recursive function above: return a function composed of the accumulated function and the current function. Note that the order of invoking the accumulated function and the current function is reversed because `reduce` works from left to right through a collection.

This is a good first step, but there is one flaw to `my-comp`: it returns a composed function which only takes one argument. In Clojure, though, `comp` returns a composed function which accepts a variable number of arguments. How can we fix our function to have the same behavior?

The key issue is the arity of the returned function within our reducing function:

``` clojure
(fn [a]
  (acc-fn (curr-fn a)))
```

If we instead change the signature to accept a variable number of arguments and apply the current function to those arguments, we have a solution. The entire `my-comp` is now:

``` clojure
(defn my-comp [& all-fns]
  (reduce
    (fn [acc-fn curr-fn]
      (fn [& args]
        (acc-fn (apply curr-fn args)))
    identity
    all-fns))
```

While building up a more involved function like `my-comp`, I find it helpful to start with a very simply recursive solution first to develop an understanding of the various details involved with a working solution. With a good understanding in place, it's fairly simple to refactor the code, both to support variable arguments and to be idiomatic.
