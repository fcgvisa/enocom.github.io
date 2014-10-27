---
layout: post
title: "Redefining Splat (A Rubeque Problem)"
categories: operators
---

There are a number of great simple, but challenging problems over at [Rubeque](http://rubeque.com/problems). As such, going through all the Rubeque problems is a great way to learn some of the less-known ins-and-outs of Ruby.

Just recently, I have been working to solve the [Redefining Splat](http://rubeque.com/problems/redefining-splat) problem. The splat operator, otherwise known as the asterisk, is one of the cooler little details about Ruby (and Python).

In its most common usage, the splat operator mops up extra arguments passed to a function into an array. For example:

``` ruby
def function(arg1, arg2, *other_ags)
  # => arg1, arg2, [other_args]
end
```

In its opposite usage, the splat operator explodes an array into its individual elements. For example, with parallel assignment:

``` ruby
a, b, c = *[1, 2, 3] # => a = 1, b = 2, c = 3
```

To the unexperience eye, this all looks like magic, which makes the "Redefining Splat" problem a little mystifying at first. The challenge is to make the following code pass by redefining the behavior of the splat operator.

``` ruby
assert_equal "FOO", *"foo"
*x = "BAR"
assert_equal ["bar"], x
```

To start it's worth noting that the problem expects two distinct behaviors from the splat operator. First, ```*"foo"``` should return an array of ```"FOO"```. I assume ```assert_equal``` works similarly to ```puts``` in that it pulls whatever elements are in an array out before testing them, hence the first line above where ```"FOO"``` does not appear within an array. Second, we need to redefine the way ```*``` works in assigning strings to variables. The assignment in line two above should return a new string within an array and with the case swapped.

Defining these two behaviors is actually quite simple. The trick is to recognize that in the first case above the ```*``` is synonymous with a call to ```to_a```. If we override the behavior of ```to_a``` within the ```String``` class, we'll have half the answer.

``` ruby
class String
  def to_a
    [self.swapcase]
  end
end
```

While it might seem that ```to_a``` handles both cases above, it actually does not. The other half of the problem depends upon redefining ```to_ary``` which is called when the ```*``` is used on a variable in an assignment. Since we can use the ```swapcase``` method on both cases, there is no need to repeat ourselves. So for ```to_ary```, we can write:

```ruby
class String
  alias to_ary to_a
end
```

The keyword ```alias``` simply sends all calls to ```to_ary``` right along to ```to_a```. And voila! We have the answer. We can now use the splat operator in front of a string and a variable with the desired behavior.