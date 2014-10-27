---
layout: post
title: "Four Types of Equality in Ruby"
categories: ruby equality
---

Ruby provides four methods to test for various types of equality. The methods are namely `equal?`, `==`, `===`, and `eql?`. Distinguishing them is a [perennial question](http://stackoverflow.com/questions/7156955/whats-the-difference-between-equal-eql-and) for newcomers to Ruby.

As one of numerous details worth knowing about Ruby, the distinction between each method has taken me some time to digest. Recently, though, I have been reading through [Eloquent Ruby](http://eloquentruby.com/), my second favorite Ruby book after [The Well-Grounded Rubyist](http://www.manning.com/black2/), and finally reached a point where the differences are clear.

First comes the most important method of all, `equal?`. Whereas the other methods occasionally require overriding, `equal?` is sacrosanct. The other methods of equality are built on top of `equal?` in one way or another. The [documentation](http://ruby-doc.org/core-2.0/BasicObject.html#method-i-equal-3F) is worth consulting to see how this works in C. In effect, `equal?` is comparing pointers to objects, which amounts to comparing the `object_id` of two objects in Ruby. If two variables point to the same object, (i.e., their `object_id` is the same), then `equal?` will return `true`.

``` ruby
"string".equal? "string" # => false, because each string is a separate object
```

After `equal?` comes `==`. Unlike `equal?`, the double-equals method is often overridden to provide more meaningful object comparison. Consider the following:

``` ruby
class Car
  attr_reader :make
  def initialize(make)
    @make = make
  end

  def ==(other)
    make == other.make
  end
end

foo_car = Car.new("Toyota")
bar_car = Car.new("Toyota")

foo_car.equal? bar_car # => false, they are two distinct objects
foo_car == bar_car     # => true,  on account of their common make
```

Without overriding `==`, a comparison between two cars would test their `object_id`, when we perhaps consider two cars to be "equal" when they both have the same make.

Next is `===` (threequals) which calls `==` (double-equals) by default. The `Regexp` class provides a good example of overriding the threequals method. In other words,

``` ruby
/c.t/ == "cat"   # => false, because we have two separate objects
/c.t/ === "cat"  # => true,  because the Regexp matches the string
```

The `case` statement evalutes equality in terms of `===`, and so provides yet another context in which overriding threequals might make sense. Likewise, the `Date` class overrides the method to test whether two dates fall on the same day (irrespective of each object being unique).

Finally, we have `eql?`. The `eql?` method is used by the `Hash` class to compare keys. In other words, `Hash` uses `eql?` to test whether two keys are the same. By defining the behavior of `eql?` and the `hash` method in a class, we can implement our own custom behavior for storage in a hash.

Consider the following contrived example. We start with a simple class modeling a key.

``` ruby
class Key
  attr_reader :type
  def initialize(type)
    @type = type
  end
end
```

And then we create two instances of that key and store one in a hash called `doors`.

``` ruby
key_one = Key.new('skeleton')
key_two = Key.new('skeleton')

doors = {}
doors[key_one] = :secret_room
```

As it stands now, if we lookup `key_two` in our `doors` hash, we will not get the same result as `key_one` even though both keys are equally skeleton keys.

``` ruby
doors[key_two] # => nil
```

To achieve our desired behavior, we need to do two things. First, we need to override `eql?` which currently tests keys with only their `object_id`. Second, we need to override the `hash` method, which currently hashes an object based on its `object_id`.

``` ruby
class Key
  def eql?(other)
    type == other.type
  end

  def hash
    type.hash
  end
end
```

Now when we recreate our `doors` hash and perform the same assignment and lookup, we will get our desired behavior. All skeleton keys will gain access to the secret room.

``` ruby
doors = {}
doors[key_one] = :secret_room
doors[key_two] # => :secret_room
```

Aside from `equal?`, the equality methods are built into Ruby to provide custom behavior and are meant to be overrided. Out of the box, the equality methods are all implemented in terms of `equal?`, aside from a number of core classes whose behavior is easy to take for granted (e.g., `"a" == "a"`, or `1 == 1.0`).
