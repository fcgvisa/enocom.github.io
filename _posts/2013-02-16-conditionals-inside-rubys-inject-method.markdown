---
layout: post
title: "Conditionals inside Ruby's Inject Method"
categories: syntax ruby
---

The Ruby method ```inject```, AKA ```reduce```, is a super cool method worth spending some time with if it's not already a familiar friend.

For example, to quickly sum the first five natural numbers, we might write:

``` ruby
first_five = [1, 2, 3, 4, 5]

# shorthand
first_five.inject(&:+) # => 15

# longhand
first_five.inject { |sum, num| sum + num } # => 15
```

What happens if we want to sum only the even numbers among the first five natural numbers? It would be tempting to write:

``` ruby
first_five.inject do |sum, num|
  if num.even?
    sum + num
  end
end
```

With the code above, Ruby will raise a ```NoMethodError``` with the message: "undefined method '+' for nil:NilClass." During this week at [DevBootcamp](http://devbootcamp.com), my pair and I found ourselves needing to do a more complicated, yet nonetheless similiar operation and were totally puzzled when ```inject``` was not behaving as expected.

After twenty minutes of sanity checks, we discovered the obvious: ```inject``` passes on the value of the memo object ```sum``` from one operation to the next. If we lock ```sum``` up within a conditional that only fires in some cases, then we're effectively breaking the chain and the memo object is not passed along to the next operation.

In other words, whenver one has the need for a conditional within a block passed to ```inject```, the memo object must be passed on to the next operation one way or another. Something like the following will do the job:

``` ruby
first_five.inject(0) { |sum, num| num.even? ? sum + num : sum }
```

If the number in question is even, the number plus the memo object will be passed on. Otherwise, it will be just the sum which is given to the next operation.

It's worth noting in the case here that we must explicity pass ```0``` to ```inject``` to prevent our first number, i.e., ```1```, from being included in the sum.
