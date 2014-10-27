---
layout: post
title: "Keeping Track of Details"
categories: functions syntax
---

This little blog will be an online notebook to keep track of details as I learn my way around Ruby and Ruby on Rails. From time to time, for the sake of comparison I will likely write about my other favorite languages, Python, C, and Objective-C. So here we go.

One of Ruby's better known features (or quirks, depending on your view) is its optional parentheses. As a result, given an arbitrary function which takes a number of arguments, Ruby's syntax allows for both of the following:

``` ruby
print_args(1, 2, 3, 4)
print_args 1, 2, 3, 4
```

Incidentally, if one forgoes the use of parentheses, then one may be liberal with spacing.

``` ruby
print_args   1,      2,   3, 4 # no problems here
```

However, a function which uses parentheses must have its opening prend immediately following the function name, although subsequent spacing may be loose:

``` ruby
print_args(1, 2, 3, 4)        # like this
print_args (1, 2, 3, 4)       # => SyntaxError
print_args(1,    2,   3, 4)   # no problems here
```

It seems the syntax of line 2 above should be correct, but it throws a ```SyntaxError```.

Another interesting syntax detail is in the use of blocks as arguments to a function. To declare a function which takes a block as one of its arguments, Ruby requires the use of an ampersand. For example:

``` ruby
def takes_block(*args, &block)
  args.map { |i| block.call(i) }
end
```

The use of the ampersand in short tells Ruby to expect a block rather than a more usual argument. The block must be named last in the list of arguments. As an aside, the splat operator before ```args``` gathers a variable number of arguments into an array -- the subject for another post.

To call the function with a block, though, Ruby's syntax requires 1) parentheses around the arguments and 2) the block outside the parentheses.

``` ruby
takes_block(1, 2, 3, 4) { |i| i * 2 } # => [2, 4, 6, 8]
```

Without the parentheses around the arguments -- what might seem acceptable syntax -- Ruby will throw a ```SyntaxError```.

``` ruby
takes_block 1, 2, 3, 4 { |i| i * 2 } # => SyntaxError
```

Having started programming by learning C, I actually prefer the use of parentheses. They help the eye quickly parse a line. As a result, I wouldn't argue in favor of "legalizing" the syntax errors above. Nonetheless, it's interesting to see just how far one can push the syntax of optional parentheses.
