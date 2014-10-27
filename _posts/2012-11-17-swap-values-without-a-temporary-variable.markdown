---
layout: post
title: "Swap Values without a Temporary Variable"
categories: algorithms bitwise-operators
---

In the world of basic programming problems, [FizzBuzz](http://www.codinghorror.com/blog/2007/02/why-cant-programmers-program.html) is probably the best known. Recently, I happened upon a second problem: how to swap the values of two variables without using a third temporary variable. The solution certainly ranks as one of those interesting little details involved with writing software. And so I'll add my own short explanation of the solution here.

Let's start by assuming that each variable holds a numerical value, as opposed to a string of text.

Given two variables (```x``` and ```y```), swapping them is easy when using a temporary value-holder. For example:

```ruby
temp = y # temp holds the intitial value of y
y = x    # y now holds the initial value of x
x = temp # and now x holds the initial value of y
```

Doing the same swap without that ```temp``` variable is also easy:

```ruby
x = x + y # assign the sum of both values to x
y = x - y # subtract the value of y from the sum (leaving the initial x)
x = x - y # and subtract the value of y (initial x) from the sum to leave initial y
```

In the case above, swapping the variables is simply a matter of basic addition and subtraction. It's an uncomplicated solution.

Another possible solution, which is more interesting, involves [bitwise operators](https://en.wikipedia.org/wiki/Bitwise_operation). More specifically, the solution uses the [exclusive-or operator](http://en.wikipedia.org/wiki/Exclusive_disjunction#Computer_science) (```^``` in Ruby) three times.

Let's work through a basic example again with ```x``` and ```y```. We'll start by setting up the initial values.

```ruby
x = 10 # assign a value (1010 in binary) to x
y = 3  # assign a value (0011 in binary) to y
```

Swapping the variabes using the bitwise operator is done as follows:

```ruby
x = x ^ y # 1010 ^ 0011 = 1001 (a temporary x value)
y = x ^ y # 0011 ^ 1001 = 1010 (the initial x value!)
x = x ^ y # 1001 ^ 1010 = 0011 (the initial y value!)
```

The last solution here may be a little intimidating to the novice for its direct operation on the bits of the variable. Wikipedia has a [proof](http://en.wikipedia.org/wiki/XOR_swap_algorithm#Proof_of_correctness) of the method in case the reader requires a sanity check.

In production code, the last solution is probably not a good idea given its purpose is not immediately obvious. In any case, using the exclusive-or operator is quite elegant in its stark simplicity and is a super cool piece of trivia in the software world.
