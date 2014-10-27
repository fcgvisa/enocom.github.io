---
layout: post
title: "Factorials in Clojure"
categories: clojure
---

Recently when time allows, I have been learning some Clojure. As a dialect  of Lisp and as a functional language, Clojure challenges its user to think in ways dramatically different from that of imperative or object-oriented styles of programming. And that's largely the fun of working with a Lisp. It's a whole new way of thinking about programming.

To start, going through [The Little Schemer](http://www.ccs.neu.edu/home/matthias/BTLS/) is a great way to acclimate to the world of Lisp. Incidentally, using the Racket IDE (found [here](http://racket-lang.org/)) is recommended for working through the book.

But once one has an understanding of some of the basics of functional programming in Lisp, there remains the substantial learning curve of simply figuring out how to write idiomatic Clojure. Like Ruby, there is a REPL for interacting with the language -- and one should use [Leiningen](https://github.com/technomancy/leiningen) for easy access to the REPL.

With the Ruby and Clojure REPL opened side by side, I have found it useful to translate from one language to another in learning how to do basic things in Clojure.

A good exercise is writing a factorial function in both Ruby and Clojure. Because Lisp dialects so commonly make use of recursion, let's write our factorial function using recursion to ease the translation between languages.

To start, here's the Ruby implementation.

``` ruby
def factorial(n)
  return n if n == 1
  n * factorial(n - 1)
end
```

We start by defining the base case: returning `n` when it reaches one. Otherwise, we multiple `n` by a recursive call to `factorial` with `n` decremented.

Aside from details in syntax, the implementation in Clojure is identical.

``` clojure
(defn factorial [n]
  (if (= n 1)
    n
    (* n (factorial (dec n)))
  )
)
```

If our number equals one, we return it, otherwise we multiply `n` by a recursive call to a decremented (hence `dec`) `n`.

For small values, both the Ruby and Clojure implementations above do fine. However, when the passed-in value becomes sufficiently large, our recursive functions will throw stack overflow errors.

Perhaps the easiest way to fix this in Ruby is to cease to use a recursive solution.

``` ruby
def factorial(n)
  (1..n).reduce(:*)
end
```

However, Clojure offers a way to still use a recursive solution without the danger of stack overflows. The following code (borrowed from [Clojure Programming](http://en.wikibooks.org/wiki/Clojure_Programming/By_Example#Immutable_data)) is an excellent example of Clojure's expressive power.

``` clojure
(defn factorial
  ([n]                    ; when only one argument is passed in
    (factorial n 1))
  ([n acc]                ; when two arguments are passed in
    (if  (= n 0)  acc
    (recur (dec n) (* acc n)))))
```

In effect, we have two function definitions wrapped into one. In Ruby land, a function has a set arity. However as in the Clojure function above, we can designate different behaviors depending on different arities.

First, there is the case of when `factorial` is called with only one argument. In those cases, we will make a recursive call passing the argument `n` and the number one.

In the case where were we have two arguments, the interesting part of our `factorial` function does its work. If our `n` argument is zero, we simply return the accumulator (i.e., `acc`). Otherwise, we make a recursive call with `recur` passing in two arguments: 1) a decremented `n`, and 2) the memo: `n` times `acc`. It is almost as if we made a call to `factorial` directly with one important difference. Unlike plain recursion, `recur` discards stack frames and simply passes the values through, leaving the recursive nature in place without the high computational cost. In other words, with `recur` we run no danger of stack overflows.

Naturally, this is a fairly straight-forward example. Nonetheless, it highlights what I find particularly fascinating about Clojure.

*Edit*

As has been [pointed out](https://twitter.com/jfarmer/status/374645184111206400) to me on Twitter, we can use an accumulator in Ruby as well:

``` ruby
def factorial(n, acc = 1)
  return acc if n == 0
  factorial(n - 1, n * acc)
end
```
