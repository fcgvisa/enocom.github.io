---
layout: post
title: "Counting Words in Clojure"
categories: clojure
---

One of the early Clojure exercises on [exercism.io](http://exercism.io/) is writing a word count algorithm. The exercise is easy enough in Ruby, but at first glance seems daunting for someone not accustomed to functional programming. In reality, though, with a proper introduction to the concept of reduce in Clojure, the exercise is quite enjoyable and not nearly so difficult.

To start, let's assume we have a string: `one fish two fish red fish blue fish`. How do we write an algorithm to get the following output?

``` clojure
{"one" 1, "fish" 4, "two" 1, "red" 1, "blue" 1}
```

In Ruby, we might split the string out into an array of words and iterate over them while keeping track of their frequency using a hash. Clojure gives us an easy way to do much the same:

``` clojure
(def words-to-count "one fish two fish red fish blue fish")

(frequencies (clojure.string/split words-to-count #"\s"))
; {"one" 1, "fish" 4, "two" 1, "red" 1, "blue" 1}
```

First, we split our string using a regular expression for whitespace (i.e., `#"\s"`), and then pass the resulting vector off to the `frequencies` function which does the rest of the work, returning the result we wanted.

So that wasn't so hard, but what if we didn't have the handy `frequencies` function? Enter `reduce`.

In a Clojure repl, we can type `(doc reduce)` to get some basic documentation on the function. According to the docs, `reduce` takes a function, an initial value, and a collection. For example, if we wanted to sum a a vector of numbers, we could use `reduce` as follows:

``` clojure
(reduce + 0 [1 2 3]) ; 6
```

The plus sign is our function, the zero our initial value, and the vector our collection. Understanding `reduce` in my mind is a fundamental requirement to understanding much of how functional programming works. And there's a [great post](http://www.learningclojure.com/2010/08/reduce-not-scary.html) on reduce which has helped me wrap my mind around the subtle function. The post deserves to be read in its entirety, but I will briefly present the points relevant to counting words here.

The core of `reduce` is in its use of looping and recursion. Let's go one step further with `reduce` before we return to our word count algorithm.

Assume we wanted to sum a vector without using `reduce`. How would that work? We would start by using `loop` and `recur`, Clojure's (and functional programming's) version of iteration.

``` clojure
(loop [acc 0 coll [1 2 3]]
  (if (empty? coll) acc
      (recur (+ acc (first coll)) (rest coll))))
```

We start by assigning some local values, `acc` for our accumulator which is set to zero, and `coll` for our initial collection. As with all recursive algorithms, we access the base case first, i.e., that our collection is empty. If it is in fact empty, we will return our accumulator. Otherwise, we use `recur` to pass new values for `acc` and `coll`: 1) the sum of our accumulator's value and the first value in the collection as `acc`, and 2) what's left of the collection without the first element, i.e., `(rest coll)`, as `coll`. This recursion will continue until we empty out `coll` and have added all values to the accumulator.

Whereas the loop above will only use addition, we could easily write our own version of reduce to which we could pass the function with which we wanted to reduce our collection:

``` clojure
(defn my-reduce [fn init collection]
  (loop [acc init coll collection]
    (if (empty? coll) acc
        (recur (fn acc (first coll)) (rest coll)))))
```

Although the function above is a naive implementation, it nonetheless demonstrates how `reduce` is built from `loop` and `recur`.

Now that we've covered the basics of `reduce`, let's return to our word count algorithm above. We know now that we can use `reduce` in the absence of `frequencies`. Our first attempt might look something like the following:

``` clojure
(def split-words (clojure.string/split words-to-count #"\s"))

(defn count-words [words-to-count]
  (reduce _ {} words-to-count))

(count-words split-words)
```

Once we've split our string into a vector of words, we can easily formulate the skeleton of our call to `reduce`. What function will take the place of the underscore above? Let's write it now.

We'll call the function `record-word-count`. It will take two arguments, a map to keep track of each word and its count, and a word to enter into a new copy of the map.

``` clojure
(defn record-word-count [memo word]
  (assoc memo word (inc (memo word 0))))
```

We'll start from the inside and move outwards. First, with `(memo word 0)`, we attempt to lookup the value stored in `memo` for our `word`. If we don't find it, we return 0. This is much like Ruby's `Hash#fetch` method which optionally takes a value for a missing key. Next, whatever value we return (zero or otherwise), we increment its value with `inc`. Finally, we use `assoc` to create a new map which will include our word and its updated value.

Our resulting code is as follows:

``` clojure
(def split-words (clojure.string/split words-to-count #"\s"))

(defn record-word-count [memo word]
  (assoc memo word (inc (memo word 0))))

(defn count-words [words-to-count]
  (reduce record-word-count {} words-to-count))

(count-words split-words)
; {"one" 1, "fish" 4, "two" 1, "red" 1, "blue" 1}
```
