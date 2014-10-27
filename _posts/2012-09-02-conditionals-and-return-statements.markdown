---
layout: post
title: "Conditionals and Return Statements"
categories: algorithms
---

When writing functions which use both conditionals and return statements, there is a common beginner's mistake to be avoided. Often, a new programmer will write a function like the following one:

``` ruby
def some_function(value)
  if value
    return True
  else
    return False
  end
end
```

In the course of improving my code, at first I found myself making the mistake above countless times. Given the use of ```return``` statements within both branches of the conditional, the ```else``` statement may seem a nice balance to the ```if``` statement. But, in fact, the use of else here is superfluous. Instead, the following is more elegant.

``` ruby
def some_function(value)
  return True if value
  return False
end
```

In the case that value evaluates to ```True``` in the function above, the control flow will never reach the second ```return``` statement. As a result, there is no need to use an ```else``` branch when using ```return``` statements.
