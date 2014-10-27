---
layout: post
title: "How to Validate for Matching Parentheses"
categories: interview-questions ruby
---

In an interview the other day, the follow problem was posed to me: Write a method which validates a string if all the opening parentheses match the closing ones. For an invalid string, the method will return `false`. For a valid string, it will return `true`.

The method should behave as follows:

``` ruby
validate("(((hello))")  # => false
validate("((")          # => false
validate(")(")          # => false
validate("(())")        # => true
```

After attempting a solution using a boolean value, I settled on a simple tally variable. Whenever an opening paren appears, the method increments the tally. When a closing paren appears, the method decrements the tally. Finally, it is just a matter of returning `true` or `false` depending on the final value of the tally. Here is the code:

``` ruby
def validate(str)
  tally = 0

  str.each_char do |char|
    case char
    when "("
      tally += 1
    when ")"
      tally -= 1
    end

    return false if tally < 0
  end

  return tally == 0
end
```

If the `tally` ever becomes negative, we have encountered an unmatched closing right paren and can immediately return false. Note also the implicit fall-through in the `case` statement which will occur when `char` is a non-paren.

How then would we implement a validator which can check for not just parentheses, but also brackets, curly brackets, and angle brackets?

Again, the method should behave as follows:

```
smarter_validate("[(])")   # => false
smarter_validate("[(0)]")  # => true
```

The solution requires a stack and a hash. Whenever we encounter a left paren or member of the left bracket family, we push it onto the stack. When we encounter a closing paren or bracket, we will pop off an item off the stack, look up its expected closing mark, and then compare the expected value with the actual value.

Here is a simple solution:

```ruby
def smarter_validate(str)
  stack  = []
  lookup = { '(' => ')', '[' => ']', '{' => '}', '<' => '>' }
  left   = lookup.keys
  right  = lookup.values

  str.each_char do |char|
    if left.include? char
      stack << char
    elsif right.include? char
      return false if stack.empty? || (lookup[stack.pop] != char)
    end
  end

  return stack.empty?
end
```

First, we initialize a stack which will hold all the left parentheses and brackets. The hash gives us an easy way to associate left parentheses and brackets with their counterparts. We pull out both the keys and the values of the hash for easy reference.

Next, we loop over the string adding left items to the stack and popping them off when we find a right item. If we find a right item and the stack is empty, or if the right item does not match the item sitting on top of the stack, we know we have a mismatch and return false.

Finally, as long as we have taken everything off the stack, we return true.

Granted this solution is hardly ready for industrial use, but it's a working solution, which is always a good first step.
