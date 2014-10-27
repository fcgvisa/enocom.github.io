---
layout: post
title: "Assigning Functions to Variables"
categories: functional-programming
---

One of the coolest (and powerful) features of JavaScript is the ability to assign functions to variables. We start with a simple function which simply logs a short message to the console.

``` javascript
function sayHello(name) {
  console.log("Hello," + " " + name);
}
```

Having defined the function we can then assign it to a variable. The variable becomes a stand-in for the function which can be called just as the function would be.

``` javascript
var x = sayHello;

x("Alice"); // "Hello, Alice"
```

Python allows the same behavior. Any function can be assigned to a variable and then called.

``` python
def say_hello(name):
  print "Hello, ", name

x = say_hello
x("Alice") # "Hello, Alice"
```

In the case of Ruby, though, once a function is defined it cannot be assigned to a variable.

``` ruby
def say_hello(name)
  puts "Hello, #{name}"
end
```

Given the function above, we cannot assign ```say_hello``` to a variable. Of course, similiar behavior can be achieved by using a ```lambda``` or ```Proc```.

``` ruby
x = -> name { puts "Hello, #{name}" }
x.call("Alice") # "Hello, Alice"
```

I find it useful to keep these differences in mind, especially when thinking about functional and imperative languages. Granted the discussion here is rudimentary. Nonetheless, the ability to assign a function to a variable is remarkably powerful and will likely be the subject of future posts as I learn my way further into Haskell and the functional aspects of JavaScript.
