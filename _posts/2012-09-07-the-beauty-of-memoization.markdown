---
layout: post
title: "The Beauty of Memoization"
categories: algorithms memoization fibonacci
---

Having gone through a handful of basic programming problems several times now, I have memorized the recursive algorithm to solve for the nth number of the Fibonacci sequence.

```ruby
def fibonacci(num)
  return num if [0, 1].include? num
  fibonacci(num - 2) + fibonacci(num - 1)
end
```

As part of the [Dev Bootcamp](http://www.devbootcamp.com/) preparation materials, though, I ran across a twist in this problem which was not as easy to solve. Instead of finding the nth number of the Fibonacci sequence, the problem is to write an algorithm to test whether a certain number falls within the sequence. To complicate things, the algorithm must likewise be able to deal with massive numbers.

A quick jump over to Wikipedia reveals this [jewel](http://en.wikipedia.org/wiki/Fibonacci_numbers#Recognizing_Fibonacci_numbers). In short, given ```n``` one and only one (hence ```XOR```) of the follow equations must be a perfect square:

```
5n^2 + 4
5n^2 - 4
```

Why not just turn these two equations into a function to solve the problem? Something like the following might spring to mind:

``` ruby
def is_fibonacci?(num)
  x = 5 * num**2 + 4
  y = 5 * num**2 - 4

  if (Math.sqrt(x).floor**2 == x) ^ (Math.sqrt(y).floor**2 == y)
    return true
  end

  return false
end
```

Though the function above works fine for small numbers, once the candidate number becomes adequately large, the function breaks and no longer accurately evaluates its argument. What then is to be done?

Why not just generate a sequence of Fibonacci numbers up to the candidate number, testing to see if we ever hit the candidate number? If we go past the number without hitting it, then we know the candidate is not a Fibonacci number. For example:

``` ruby
def is_fibonacci?(i)
  f_nums = []
  j = 0

  while true
    f_nums.push(fibonacci(j))
    j += 1
    if f_nums.last >= i
      break
    end
  end

  if f_nums.last == i
    return true
  end

  return false

end
```

But we have another problem. Generating Fibonacci numbers up to a certain massive number is itself going to be impossibly slow. The major slowdown comes from using recursion within the ```fibonacci``` function. Since we'll be calling the ```fibonacci``` function repeatedly, keeping track of the values we've already calculated will provided for a major speed increase.

Enter memoization. If we use a global hash to keep track of the sequence number and the value (e.g., the first number is 0, the second is 1, the third is 1, the fourth is 2, etc.), we can skip recalculating the values we've already calculated once. Our ```fibonacci``` function will first check the hash before proceeding any further. And if we find a new value, we'll enter it in the hash:

``` ruby
$memo = { 0 => 0, 1 => 1 }

def fibonacci(num)
  return $memo[num] if $memo.include?(num)
  res = fibonacci(num - 2) + fibonacci(num - 1)
  $memo[num] = res
  return res
end
```

All of a sudden, the function above speeds up considerably and we can test even massive numbers.
