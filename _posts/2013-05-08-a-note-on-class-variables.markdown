---
layout: post
title: "A Note on Class Variables"
categories: ruby
---

If you have been using Ruby for a little while, you might have encountered class variables which are indicated within the top level of a class definition by prepending a variable name with two at-signs, e.g., `@@foo`.

At first glance, class variables are pretty cool. Just as instances have instance variables, classes have class variables. And all is well, right?

Wrong. In [Metaprogramming Ruby](http://pragprog.com/book/ppmetr/metaprogramming-ruby) -- a phenomenal book worth reading immediately if you have not read it already, by the way -- the following example is used to warn a Rubyist away from class variables.

``` ruby
@@foo = 1

class Bar
  @@foo = 2
end

@@foo # => What will the value be?
```

Now it would seem that we have two separate scopes in the code here. In reality, when we ask for `@@foo` again, guess what its value will be? If you guessed two, the tricky bit with class variables is perhaps already clear.

Since a class variable is inherited by all subclasses (`Object` being the superclass and `Bar` being the subclass here), we have access to that single variable throughout the class hierarchy and can all too easily overwrite its value.

The solution is to use instead class instance variables. That may sound like an oxymoron. In fact, because Ruby classes are really just objects, we can define instance variables that are properly encapsulated within the class. Observe.

``` ruby
@foo = 1

class Bar
  @foo = 2

  # adding a getter for our class instance variable
  def self.foo
    @foo
  end
end

Bar.foo   # => 2
@foo      # => 1
```

And, voilà! We have a class instance variable unaffected by what occurs outside the class.
