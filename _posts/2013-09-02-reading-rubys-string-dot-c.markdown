---
layout: post
title: "Reading Ruby's string.c"
categories: ruby-source ruby
published: false
---

While playing around with some Ruby exercises at [exercism.io](http://exercism.io/), I happened upon a surprising method called against a string:

``` ruby
"foobar"[/foo/] # => "foo"
```

Rather than the usual index lookup (e.g., `"foo"[1]`), we have a regular expression passed to the `[]` method. In effect, the above line is a shorthand version of:

``` ruby
"foobar".match(/foo/) # => #<MatchData "foo">
```

Now a sane person might consult the documentation to understand the details, but I wanted to see how one method (i.e., `[]`) could have two distinct and drastically different behaviors.

Enter `string.c` (found [here](https://github.com/ruby/ruby/blob/trunk/string.c)).

Before we go to C-land, here are a few things to keep in mind while digging around the Ruby source:

- Many of the base Ruby classes (e.g., `String`, `Hash`, `Range`, etc.) are found in a corresponding `.c` file in the top-level of the Ruby repo on Github.

- After opening one of these `.c` files, a good way to get started is to skip all the way to the file's end where there will be an initialization method in which the class in question is created along with all its methods. For example, in `string.c`, we'll find `Init_String` on [line 8657](https://github.com/ruby/ruby/blob/trunk/string.c#L8657). Below, all the methods available to `String` are defined.

- A method is defined using the `rb_define_method`. A typical usaage is something like the example below. We have three arguments: 1) the Ruby class onto which the method will be added; 2) the actual string which will be the method signature; 3) the corresponding C function which does all the work; 4) and finally the arity.

``` ruby
rb_define_method(rb_cString, "==", rb_str_equal, 1);
```

With the three points above in mind, we are ready to jump into `string.c` and see just how the `[]` method can do two separate things.

For starters, we have the method definition on [line 8677](https://github.com/ruby/ruby/blob/trunk/string.c#L8677):

``` ruby
rb_define_method(rb_cString, "[]", rb_str_aref_m, -1);
```

See [here](https://github.com/ruby/ruby/blob/trunk/class.c#L1323) for an explanation of what a negative one means for the arity.

https://github.com/ruby/ruby/blob/trunk/string.c#L3499

static VALUE
rb_str_aref_m(int argc, VALUE *argv, VALUE str)
{
    if (argc == 2) {
    if (RB_TYPE_P(argv[0], T_REGEXP)) {
      return rb_str_subpat(str, argv[0], argv[1]);
    }
    return rb_str_substr(str, NUM2LONG(argv[0]), NUM2LONG(argv[1]));
    }
    rb_check_arity(argc, 1, 2);
    return rb_str_aref(str, argv[0]);
}
