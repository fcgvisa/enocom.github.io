---
layout: post
title: "Implementing Map in Clojure"
categories: clojure
---

Of all the problems on [4Clojure](http://www.4clojure.com), I continue to revisit the challenge of implementing `map`. It's such a basic function, yet writing it from scratch has some subtle problems. Let's look at the process in detail.

To begin, here are the assertions that our custom `map` function needs to handle:

``` clojure
;; assertion 1
(= [3 4 5 6 7]
  (my-map inc [2 3 4 5 6]))

;; assertion 2
(= (repeat 10 nil)
  (my-map (fn [_] nil) (range 10)))

;; assertion 3
(= [1000000 1000001]
  (->> (my-map inc (range))
       (drop (dec 1000000))
       (take 2))
```

For now, we will focus on the first assertion. A naive solution might make use of `for`.

``` clojure
(defn my-map [f col]
  (for [x col]
    (f x)))
```

In fact, the implementation above passes all three assertions. To make the problem more interesting, though, 4Clojure disallows the use of `for` in a solution. So the question is, how do we implement map without using `for`?

Again, let's concern ourselves with just the first assertion. For anyone who has worked through [The Little Schemer](http://www.ccs.neu.edu/home/matthias/BTLS/), the following recursive solution might come to mind:

``` clojure
(defn my-map [f col]
  (loop [acc []
         c coll]
    (if (empty? c)
      acc
      (recur (conj acc (f (first c)))
             (rest c)))))
```

Using an accumulator (i.e., `acc`), we recursively work our way through a collection, accumulating the values which have been calculated using the mapping function (i.e., `f`). This solves the constraint of avoiding `for`, but when we run this implementation as part of the third assertion, we have a performance disaster.

Our implementation of `map` will never finish incrementing an infinite range, and so, the only way to remove the performance problem is to start being lazy.

``` clojure
(defn my-map [f col]
  (if (seq col)
    (lazy-seq
      (cons (f (first col))
            (my-map f (rest col))))))
```

By wrapping our recursive call to the map function with a `lazy-seq`, we ensure that only the necessary values will be calculated and nothing else. And with the solution here, we have a working implementation of `map` which accepts a mapping function and a single collection as an argument.

