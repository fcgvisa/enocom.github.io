---
layout: post
title: "Testing for Balanced Brackets in Clojure"
categories: clojure
---

While working through [4clojure](http://www.4clojure.com/), I recently came across a problem on balanced brackets. I have written about a solution in Ruby to the problem in a [previous post](http://commandercoriander.net/blog/2013/04/18/how-to-validate-matching-parentheses/). How to go about solving the same problem in a functional style wasn't immediately obvious, though, and so it seemed like an amusing challenge.

We'll write a function which takes a string and returns either `true` or `false` depending on whether the various brackets and parentheses are balanced.

``` clojure
(defn balanced? [s]
    ;; do something here
    )
```

Our input will be a string of various characters, including parentheses, square brackets, and curly brackets. For instance, we might have something like the following Hello World program in string form:

``` clojure
"class HelloWorld {
    public static void main(String[] args) {
        System.out.println(\"Hello world.\");
    }
}"
```

Alternatively, we might have something as simple as `"())"`. Our function needs to handle both.

The first obvious step is to remove all non-bracket characters from the string. We can start by filtering the passed-in string to include only the characters of interest.

``` clojure
(defn balanced? [s]
    (filter (set "()[]{}") s)
    ;; still need something here
    )
```

Here we're using a set of bracket characters as a function. If a bracket character is passed to the set, we'll get that character back, otherwise we'll get `nil`.

``` clojure
((set "{}[]()") \{) ;; => \{
((set "{}[]()") \z) ;; => nil
```

By passing the set of bracket characters as a function to `filter`, we'll end up with just the relevant characters. As it stands now, our `balanced?` function is a simple filter.

``` clojure
(balanced? "[{foo(bar)}]") ;; => (\[ \{ \( \) \} \])
(balanced? "foobar")       ;; => ()
```

The next step needs a little more thought.

On a fundamental level, we need to take a collection and reduce it to a single value, either `true` or `false`. Reduce is the key word in the previous sentence and the case here is a perfect use for the `reduce` function. If you still feel a little uncomfortable with `reduce`, the [Programming Course in Clojure](http://iloveponies.github.io/120-hour-epic-sax-marathon/) has an [excellent chapter](http://iloveponies.github.io/120-hour-epic-sax-marathon/one-function-to-rule-them-all.html) on `reduce` alone.

Assuming we start with a filtered string, our use of `reduce` will look something like the following:

``` clojure
(reduce 
    (fn [stack item] ;; the reducing function
        ;; do something here
        )
    []       ;; the initial value of `stack` passed to the function
    "{}()[]" ;; our filtered input string goes here
    )
```

As we work our way through the filtered string, calling our reducing function for each value, we will use a stack to keep track of the matching parens. The initial value of the stack will be an empty vector.

From there, the body of our reducing function needs to account for three cases.

First, whenever we encounter an opening bracket, we need to push it onto the stack so that later on when we encounter a closing bracket we can check the top of the stack to ensure a match.

Second, whenever we encounter a closing bracket, if that bracket closes whatever is sitting on the top of the stack, we have a match set of brackets and we need to pop that opening bracket off the stack.

And finally, if the first two cases don't apply, we have encountered an unmatched bracket. We'll simply push it onto the stack for future reference.

Let's look at how this reducing function might be written in code. Since we have three conditions, we'll use `cond` to address them. Recall that `cond` consists of a boolean test and a corresponding bit of code to run when that test passes.

``` clojure
(fn [stack item]
    (cond
        ;; if we're looking at an opening bracket, add it to the stack
        ((set "({[") item) (conj stack item)

        ;; otherwise, we're looking at a closing bracket

        ;; if the last element of the stack is an opening bracket
        ;; and the current item closes the last item in the stack
        ;; we have a match, so remove the opening bracket from the stack
        (and ((set "({[") (last stack))
             (= ({ \) \(, \} \{, \] \[ } item) (last stack)))
        (pop stack)

        ;; we have a mismatch, add it to the stack
        :else (conj stack item)))
```

The first condition to `cond` uses the same trick using `set` above. If the current item is an opening bracket, we'll `conj` it onto the stack.

The second condition is a little more involved, but still rather simple. Since we know at this point we're dealing with a closing bracket, we need to see if it closes the last element on the stack. We first check that the last element on the stack is, in fact, an opening bracket. If that's true, we now need to check if the current closing bracket closes that opening bracket, so as to avoid something like "{)". To do this, we look up our current item in a map between opening and closing brackets and compare it to the last item on the stack. The map lookup follows the same idea with the set above, using a map as a function, with a particular bracket as an argument.

For example:

``` clojure
;; right curly bracket returns left curly bracket
({ \) \(, \} \{, \] \[ } \}) ;; => \{

;; right parent returns left paren
({ \) \(, \} \{, \] \[ } \)  ;; => \(
```

If we have a match between opening and closing brackets, we pop the corresponding opening bracket off the stack.

If we reach the final condition, we know that we have an unbalanced bracket. To keep track of this fact, we simply add it to the stack.

At this point, our reducing function will return an empty vector if all brackets are balanced. If the returned vector has any members, we know that we have unbalanced brackets.

In order for the `balanced?` function to return either `true` or `false`, we simply need to call `empty?` on the resulting vector.

In sum, our `balanced?` function will first filter the input string to remove non-bracket characters. It will then reduce the resulting characters to an alternatively empty or non-empty vector. And then finally, it will return the result of calling `empty?` on that vector.

Using the `->>` macro which takes an initial argument and threads it through any number of forms as the last argument, we can write our `balanced?` function just as described above. The result is as follows:

``` clojure
(defn balanced? [s]
    (->> s
        ;; remove non-bracket characters
        (filter #{\[ \] \( \) \{ \}})
        
        ;; reduce down to an empty or non-empty vector
        (reduce 
            (fn [stack item]
                (cond
                    (#{ \( \{ \[ } item) (conj stack item)
                    
                    (and (#{ \( \{ \[ } (last stack))
                         (= ({ \) \(, \} \{, \] \[ } item) (last stack)))
                    (pop stack)
                    
                    :else (conj stack item)))
        [])

        ;; return whether we have any unbalanced brackets
        empty?))
```

And there it is: a fairly simply way to check a string for unbalanced brackets in a function style.