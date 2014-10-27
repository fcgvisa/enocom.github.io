---
layout: post
title: "Lamdas and Procs"
categories: python ruby lambda proc
---

One of my favorite little details from Ruby is its use of ```Proc``` and ```lambda```, two cool syntactic features which create anonymous functions for simple tasks. For instance, to create a basic function that calculates square roots (as if ```Math.sqrt()``` weren't easy enough), you first create a new ```Proc``` while handing it a block. Then, calling that ```Proc``` is simply a matter of sending the ```call``` method with however many arguments are required.

``` ruby
pr = Proc.new { |x| x**x }
pr.call(2) # => 4
```

Incidentally, a ```Proc``` is not picky about the number of arguments. If you pass ```pr``` above more than one argument, it will happily ignore everything after the first argument.

Meanwhile, Ruby offers a similar feature with ```lambda```:

``` ruby
la = lambda { |x| x**x }
la.call(2) # => 4
```

The ```lambda``` feature also allows for a slightly different syntax as well, using ```->``` in place of ```lambda``` and requiring arguments within parentheses before the block.

``` ruby
lam = ->(x) { x**x }
lam.call(2) # => 4
```

One key difference with ```Proc```, though, is that ```lambda``` will complain about extra arguments, yielding an ```ArgumentError```. Yet another difference is the way ```lambda``` handles ```return```. Whereas ```Proc``` will return immediately from a function, ```lambda``` will only stop its execution of a block. The example from [The Well Grounded Rubyist](http://www.manning.com/black2/) is as follows:

``` ruby
def return_test
  lam = lambda { return }
  lam.call
  puts "Still here!"
  pro = Proc.new { return }
  pro.call
  puts "You won't see this message!"
end
```

Interestingly, calling a ```Proc``` which simply returns when not within a function will create a ```LocalJumpError```.

As a side note, there is a ```lambda``` function in Python. When doing something simple like in the examples above, the functionality seems nearly identical:

``` python
al = lambda x: x**x
al(2) # 4
```

However, as many comparisons between Ruby and Python point out, only Ruby can have multi-line blocks (using ```do``` and ```end```). In Python, ```lambda``` is always one line.