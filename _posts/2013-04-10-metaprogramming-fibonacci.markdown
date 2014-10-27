---
layout: post
title: "Metaprogramming Fibonacci"
categories: fibonacci metaprogramming
---

In a recent pairing session, a Ruby veteran showed me how to use metaprogramming to abstract a memoizing functionality out of a recursive Fibonacci method. Let's look at how that works.

First, we will start with a basic recursive Fibonacci method whose use of memoization depends upon a class variable.

``` ruby
class Fib
  @@memo = { 0 => 0, 1 => 1 }

  def self.fib_rec(n)
    @@memo[n] ||= fib_rec(n - 1) + fib_rec(n - 2)
  end
end
```

The code above is concise, fast, and clear. In my mind, it demonstrates both the power and beauty of Ruby. Abstracting the memo using metaprogramming serves only to emphasive the expressiveness of the language.

Let's start by removing the memo from our current method and converting the method itself into an instance method. The reason for switching from a class method will become clear in a moment. At the same time, we will mixin a `Memo` module, which will contain the `memoize` function.

``` ruby
class Fib
  extend Memo

  def fib_rec(n)
    return n if n < 2
    return fib_rec(n - 1) + fib_rec(n - 2)
  end

  memoize :fib_rec
end
```

Now we need to create the `Memo` module and define `memoize`. The method will need to do two things. It will redefine whatever method it receives so as to use memoization. And it will create an alias so that the unmemoized method is still callable. One implementation following such requirements is as follows:

``` ruby
module Memo
  @@memo = { 0 => 0, 1 => 1 }

  def memoize(method)
    alias_method "old_#{method}".to_sym, method
    
    define_method(method) do |*args|
      @@memo[args] ||= self.send("old_#{method}".to_sym, *args)
    end
  end
end
```

Although the `memoize` function looks complicated, it is quite simple. First, `alias_method` simply prefixes "old" before whatever method is passed in so that we can still call the old method. Second, with `define_method` we dynamically define a new method with memoization built in. Our new method still calls the old method whenever our memo does not hold a value for a particular key. Note that `define_method` will define an instance method in the receiver. Hence the need to switch from a class method above.

In all likelihood, `memoize` could use refactoring. Nonetheless, it gets the job done and demonstrates a basic use of metaprogramming with dynamic method assignments.

**Update 13 April 2013**


Charlie Somerville has proposed an improvement for the `memoize` method on [Twitter](https://twitter.com/charliesome/status/322698820297322497).

``` ruby
module Memo
  def memoize(method)
    old_method = instance_method(method)
    memo = {}

    define_method(method) do |*args|
      memo[args] ||= old_method.bind(self).call(*args)
    end
  end
end
```

There are three key changes. First, instead of using `alias_method`, we have `instance_method`. Presumably, `alias_method` does not protect against a user altering the alias. By using `instance_method` instead, we have an unbreakable hold on the old method.

Second, the class variable `memo` has been removed in favor of a local hash. Why does this work? Because the `memo` object is referenced inside a block which behaves much like a closure, `memo` will persist throughout the lifetime of the program and will remain available for lookup.

Finally, we have to bind our instance method to a class. By passing `self` to `bind`, we are attaching our new method to the class of whatever method was passed in. The final step is to simply call the method with the appropriate arguments.

The result is a much tighter implementation of `memoize`.