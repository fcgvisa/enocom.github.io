---
layout: post
title: "A Cheatsheet for Multi-line Strings in Ruby"
categories: ruby
---

Now and then, I need a multi-line string in Ruby. Sometimes it's for a long error message. Other times, it's for a custom SQL query. I often forget, though, the various options. So here's a quick cheatsheet for multi-line strings in Ruby 2.1.

The simplest solution is something like this:

``` ruby
[
  "This is the first line.",
  "This is the second line.",
  "1 + 1 is #{1 + 1}."
].join("\n") + "\n"

# => "This is the first line.\nThis is the second line.\n1+1 is 2.\n"
```

Using a docstring:

``` ruby
my_str = <<-FOO
This is the first line.
This is the second line.
1 + 1 is #{1 + 1}.
FOO

# => "This is the first line.\nThis is the second line.\n1+1 is 2.\n"
# Note the *absence* of a leading newline character.
```

Ruby also supports a shorthand syntax. For no interpolation, use `%q`.

``` ruby
my_str = %q(
This is the first line.
This is the second line.
1 + 1 is #{1 + 1}.
)

# => "\nThis is the first line.\nThis is the second line\n1 + 1 is \#{1 + 1}.\n"
# Note the absence of interpolation.
```

Capital `q` provides interpolation.

``` ruby
my_str = %Q(
This is the first line.
This is the second line.
1 + 1 is #{1 + 1}.
)

# => "\nThis is the first line.\nThis is the second line.\n1 + 1 is 2.\n"
```

Note that there is a variant syntax for `%Q`, which is as follows:

``` ruby
my_str = %(
This is the first line.
This is the second line.
1 + 1 is #{1 + 1}.
)

# => "\nThis is the first line.\nThis is the second line.\n1 + 1 is 2.\n"
```
