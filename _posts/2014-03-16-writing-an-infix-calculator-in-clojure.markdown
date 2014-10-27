---
layout: post
title: "Writing an Infix Calculator in Clojure"
categories: clojure

---

A common problem in the Lisp world involves demonstrating how in spite of the language's use of prefix notation, it is so robust as to allow for infix notation at the same time.

In other words, normally, we might put together a series of mathematical calculations like this:

``` clojure
(/ (- (+ 38 48) 2) 2)
```

Alternatively, we could also write a function to support the following, as well:

``` clojure
(infix 38 + 48 - 2 / 2)
```

Let's look at how the `infix` function might be written. A common Lisp style would be to write the function using recursion. We would treat the arguments to `infix` as a stack, pulling them off one at a time and operating on an accumulator.


``` clojure
(defn infix [val & others]
  (loop [acc   val
         stack others]
    (if (empty? stack)
      acc
      (let [op  (first stack)
            b   (second stack)
            rem (rest (rest stack))]
        (recur (op acc b) rem)))))
```

Above, we treat the first argument to `infix` as the accumulator `acc` which will hold the result of the various operations. The `stack` binding will hold all the remaining operations and numbers. Our base case occurs when the `stack` binding is empty, at which point we hand back our accumulator and have a result. Otherwise, using `let` for clarity, we bind an operator `op`, the first element from the `stack` list, and a number `b`, the second element. What remains will be everything from the third element onward, what is slightly clumsily done here with two calls to `rest`. With the bindings in place, we make a recursive call to `infix`, this time passing the result of the operation on the accumulator and our next number, as well as the remaining operations or numbers.

What makes the problem interesting is its highlighting how easy it is to store functions like `+`, `-`, `*`, and `/` in a list and then later call them. In particular, in my mind, it's the line `(op acc b)` that shows how cool a Lisp can be.

Now the solution above is rather verbose. The use of `loop` and `recur` comes from my having learned Lisp with [The Little Schemer](http://mitpress.mit.edu/books/little-schemer). In writing Clojure code, I find it helpful to start with a recursive algorithm using `loop` and then refactor towards a tighter implementation when possible.

A cleaner implementation of `infix` in Clojure might look something like the following. (N.B. The code below is not my own and is taken from the collection of solutions to [problem 135 on 4clojure](http://www.4clojure.com/problem/135).)

``` clojure
(defn infix
  ([x op y]
      (op x y))
  ([x op y & xs]
     (apply infix (cons (infix x op y) xs))))
```
By defining two method signatures with distinct arities, we can set up our list of numbers and operations, and then call `apply` with our simple method signature to get a result.

Taking the example above, i.e., `(infix 38 + 48 - 2 / 2)`, our initial call to `infix` takes the first three elements from the passed arguments, e.g., `38`. `+`, and `48`, and calls `infix` again, this time using the simple `(op x y)` code. From there, we call `cons` with `86` and `(- 2 / 2)`, to get the list `(86 - 2 / 2)`. From there, it's a simple matter of invoking our `infix` method on the list, which will produce a solution.
