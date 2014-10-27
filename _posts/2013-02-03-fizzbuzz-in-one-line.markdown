---
layout: post
title: "FizzBuzz in One Line"
categories: interview-questions
---

Over the past month, I've relocated to San Francisco and started as part of the spring cohort at [DevBootcamp](http://www.devbootcamp.com/). As a result, I have had less time to add new posts here.

Among the many highlights of last week at DBC, I learned a one-line implementation of FizzBuzz in Ruby. It's a clever piece of code and worth taking apart.

First, to review, the requirements of the classic [FizzBuzz](http://www.codinghorror.com/blog/2007/02/why-cant-programmers-program.html) program are as follows: print out the numbers one to one hundred. If a number is divisible by three, print out "Fizz" instead of the number. Likewise, if a number is divisible by five, print out "Buzz." Finally, if a number is divisible by both three and five, print out "FizzBuzz."

Here's how to do all that using just one line in Ruby.

``` ruby
puts (1..100).map { |i| (fb = [["Fizz"][i % 3], ["Buzz"][i % 5]].compact.join).empty? ? i : fb }
```

Naturally, the code above is far from transparent. Let's take it apart piece by piece.

We start by calling ```puts``` on an array, which will be the return value of the ```map``` function. The nice thing about ```puts``` is that it will iterate over an array, printing an element and then a new line each time.

Next comes the hairy part. We call ```map``` on one through one hundred. For each number, we assign a particular value to the ```fb``` local variable. Understanding that particular value is at the heart of this one-liner.

Let's start by assuming that we've passed three into the block, represented by ```i```. The expression ```i % 3``` becomes ```3 % 3``` which evaulates to zero. Therefore, we'll have ```["Fizz"][0]```. The zeroeth index of a one member array is the first member, and so the result will be ```"Fizz"```. Second, the expression ```i % 5``` will become ```3 % 5``` which will evaluate to ```5```. And this is the most important trick. In Ruby, when we ask for an index on an array which is larger than the array, e.g., ```["a", "b", "c"][10]```, we won't get an error. Instead, we'll simply get ```nil```. As a result, the expression ```["Buzz"][3 % 5]``` will become ```["Buzz"][5]``` which will finally become simply ```nil```.

So, when ```i``` is three, the resulting array will be:

``` ruby
# step one
[["Fizz"][3 % 3], ["Buzz"][3 % 5]]
# step two
[["Fizz"][0], ["Buzz"][5]]
# step three
["Fizz", nil]
```

In other words, every time we have a number which is evenly divisible by three, i.e., ```i % 3 == 0```, we'll get a ```"Fizz"``` in our array. And every time we get a number which is evenly divisible by five, i.e., ```i % 5 == 0```, we'll get a ```"Buzz"```. If both conditions are satisfied, e.g., as with ```15```, we'll get an array with both ```"Fizz"``` and ```"Buzz"```, e.g., ```["Fizz", "Buzz"]```.

With that down, we simply call ```compact``` on the resulting array, removing any ```nil``` members, and join the two members together into a string. If we have a ```"Fizz"``` and a ```nil```, the ```join``` method is smart enough to drop the ```nil```, and just leave ```"Fizz"```. A ```"Fizz"``` and a ```"Buzz"``` will become ```"FizzBuzz```. If we have two ```nil``` members, ```join``` will return an empty string.

We then assign the result of ```compact``` and ```join``` to a variable local to the block, i.e., ```fb```.

The rest of the line is simply a ternary expression. If our ```fb``` string is an empty string, we return ```i```, otherwise we return whatever has been assigned to ```fb```.

And that is the FizzBuzz problem in one line. Just please don't write code like this in production.
