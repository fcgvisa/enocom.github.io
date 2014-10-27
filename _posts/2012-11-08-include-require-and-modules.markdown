---
layout: post
title: "A Short Guide on How to Use Include, Require, and Modules in Ruby"
categories: ruby modules
---

Having become accustomed to thinking of ```include``` as a preprocessor directive as in C (i.e., ```#include```), I have been a little confused about Ruby's own use of the statement. In addition, the ```require``` keyword is also common in Ruby. From a C perspective, this all seems quite strange. At least that was how I felt until I finally got fed up with not understanding the difference between ```include``` and ```require``` and finally made sense of it all.

The explanation from [Programming Ruby 1.9](http://www.ruby-doc.org/docs/ProgrammingRuby/html/tut_modules.html) (i.e., the "pickaxe book") is especially good for those coming to Ruby from C:

>[The ```include``` statement] has nothing to do with files. C programmers use a preprocessor directive called ```#include``` to insert the contents of one file into another during compilation. The Ruby ```include``` statement simply makes a reference to a module. If that module is in a separate file, you must use ```require``` (or its less commonly used cousin, ```load```) to drag that file in before using ```include```.

Let's look at a basic example. If we have a file called ```hello.rb``` which simply defines a method ```say_hello```, then we can gain access to the function in a second file by using the following line. Note that the file must exist within Ruby's ```$LOAD_PATH```.

```ruby
require 'hello'
```

In the second file, we can call the function directly with ```say_hello``` as we would in the original file. Of course, this is an ugly way to bring in our function because it pollutes the global namespace.

The answer to this problem is to wrap up our function into a module. Let's call it ```Greeter```. Now when we require our original file, we can call the ```say_hello``` function by invoking it through the module name: ```Greeter.say_hello```.

The ```include``` statement lets us take things one step further. Let's say we have a ```Person``` class and want to include the ```say_hello``` function from the ```Greeter``` module. Basically, we want to be able to call the ```say_hello``` function on an instance of ```Person``` without bothering to define a new method. The ```include``` handles all this for us.

```ruby
class Person
  include Greeter
end
```

In effect, the ```include``` statement pulls in all the methods defined in ```Greeter```, which at this point consists of only the ```say_hello``` function. Now we can call the method directly on an instance of ```Person``` as ```bob.say_hello```. 

As a side note, there is a minor difference in syntax in the module depending on how it will be used. To use methods outside of a class, each method within the module must be defined on the module, e.g., ```Greeter.say_hello```. Otherwise, if the methods within a module will be included within a class, then methods should be defined without any prefix, e.g., ```say_hello```.

Sytanx details aside, let's return to look at Ruby itself now with a sense of how modules and ```include``` works.

There are a number of "built-in" modules which are included on the foundational ```Object``` class. The ```Kernel``` module is a good example. It provides all sorts of methods used throughout Ruby. For instance, ```Kernel``` defines ```puts``` and major classes like ```Array``` and ```String```. Because ```Object``` brings in the ```Kernel``` module using ```include``` (that's the effect, at least), we can simply call ```puts``` directly without any prefix.

Meanwhile, in addition to the built-in modules, Ruby likewise ships with the Standard Library which includes more than one hundred additional modules. A particular libray module is loaded in the same way as seen above. For example, to use ```FileUtils```, we first pull it in with a ```require``` statement. The file name should be all lowercase:

```ruby
require 'fileutils'
```

Then, if we don't want to prefix each method call with the module name, we can use ```include``` in the appropriate place and all the module's methods will be available locally.

In short, ```require``` and ```include``` are quite simple: ```require``` pulls in a file's modules and ```include``` localizes the module to the current namespace. It's stuff like this that makes Ruby so much fun.
