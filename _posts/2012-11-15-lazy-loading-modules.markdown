---
layout: post
title: "Lazy Loading Modules"
categories: ruby lazy-loading
---

Working through [Rubeque](http://www.rubeque.com) has taught me lots of little Ruby tricks. One of the cooler things I've learned recently is how to use lazy loading with modules.

First, for the problem. Given the following code, what should go in place of the underscores to make the code pass?

```ruby
module ChildModule
  ___
end

module ParentModule
  def parent_module_method
    'This should get called'
  end
end

class TestClass
  include ChildModule
end

assert_equal TestClass.new.parent_module_method, 'This should get called'
```

At first glance, it's easy to think that simply using ```include ParentModule``` within ```ChildModule``` will solve the problem. It doesn't. Ruby throws a ```NameError``` and complains about an uninitialized constant, ```ChildModule::ParentModule```. If we had defined ```ParentModule``` before ```ChildModule```, this wouldn't be a problem. But, for whatever reason, ```ChildModule``` comes first in the problem above. How is this solved, then?

The answer depends upon a callback. When some other module or class includes a module, that module receives the message ```included``` with a reference to the including object. By overriding this method, we can lazily load ```ParentModule``` in ```ChildModule``` only when ```TestClass``` includes ```ChildModule```. The code is perfectly clear:

```ruby
module ChildModule
  def self.included(base)
    base.send :include, ParentModule
  end
end
```

By using the ```send``` method in the callback function ```included```, we make sure that Ruby won't load ```ParentModule``` until the moment ```ChildModule``` is needed. And that's the solution.
