---
layout: post
title: "Testing for Fibonacci Numbers"
categories: algorithms fibonacci
---

After working out how to check if a number falls within the Fibonacci sequence, I felt assured of the simplicity of my solution:

- use a recursive function to generate a Fibonacci number
- create an array of all Fibonacci numbers up to the candidate number
- stop generating Fibonacci numbers once we either reach or pass the candidate

With the help of memoization, the above solution can easily deal with even large numbers.

After submitting my own solution, though, I had the chance to look at a number of other solutions. Looking back now, I see that the solution above is actually excessively verbose and unnecessarily expensive.

Instead, the following code does a much better job of testing numbers for membership in the Fibonacci sequence without the expense of recursion.

``` ruby
def is_fibonacci?(num)
  arr = [0, 1]
  while num > arr.last
    x, y = arr.pop(2)
    arr.push(x, y, x + y)
  end

  return true if num == arr.last
  return false
end
```

In effect, the function above is a non-recursive Fibonacci number generator. It also checks the most recent Fibonacci number against the candidate number until it can confirm whether the candidate falls within the series.

Since I've been thinking lots about recursive functions recently, I automatically assumed recursion was the best way to solve the problem. It  makes me think of the saying "To someone who only knows how to use a hammer, all problems look like nails."
