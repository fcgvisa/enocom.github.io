---
layout: post
title: "Self and Attributes"
categories: oop ruby
---

Consider the following code:

```ruby
class Person
  attr_accesor :age

  def initialize(age)
    @age = age
  end
end
```

Suppose we wanted to refer to the ```age``` attribute within the ```Person``` class. What is the difference between ```@age``` and ```self.age```?

There is a subtle but important difference here. I discovered it firsthand this week when a mentor posed the question to me. On first brush with the question, I almost disregarded its value because the question seemed inconsequential. But there is a big difference.

In order to make the difference clear, let's start by forgoing the use of ```attr_accessor``` and write our own setters and getters. It might be tempting to write something like the following:

```ruby
class Person
  def age=(new_age)
    self.age = new_age
  end

  def age
    self.age
  end
end
```

The code above doesn't work.

I found myself writing the code above and wondering at first why Ruby was complaining at me. I was stuck in the mindset of Objective-C, my first objective-oriented langauge. In Objective-C, the code might look something like this:

```objective-c
@interface Person : NSObject
{
  PersonAge age;
}

- (PersonAge) age;
- (void) setAge: (PersonAge) newAge;
@end

@implementation
- (PersonAge) age
{
  return self.age;
}

- (void) setAge: (PersonAge) age
{
  self.age = age;
}
@end
```

Unlike the Ruby code above, the code here doesn't raise any errors. The ```age``` attribute is stored in the interface and the setters and getters for ```age``` can safely reference ```self.age```.

Let's go back to Ruby. Because attributes are accesible on an object through the ```self``` keyword in Objective-C, I was assuming that Ruby behaved the same way. Of course, as with all learning, I wasn't immediately aware of my assumptions.

Just as instance variables are declared within the ```@interface``` of an object in Objective-C, so too in Ruby are instance variables (often) declared in the ```initialize``` method. Instead of the erroneous Ruby code above, a more correct way would be to write the following:

```ruby
class Person
  def initialize(age)
    @age = age
  end

  def age
    @age
  end

  def age=(new_age)
    @age = new_age
  end
end
```

This code will work. By using the ```@``` symbol, we're referencing the ```age``` instance variable. In other words, the code above is synonymous with the Objective-C code further up. The question is when to use the ```self``` keyword instead of ```@age```.

Let's assume we have another method, ```fake_age``` so a person can act as if they're ten years older. The code would look something like this:

```ruby
class Person
  attr_accessor :age

  def initialize(age)
    @age = age
  end

  def fake_age
    self.age + 10
  end
end
```

The body of ```fake_age``` will return the same value as adding ten to ```@age```. Why then do we use ```self.age``` instead? Granted, at this point, we've wandered into matters of style. Nonetheless, by using ```self.age``` instead of ```@age```, we are "eating our own dogfood." In other words, ```attr_accessor``` has created a getter and setter for the ```@age``` instance variable, and so rather than deal with the instance variable directly, we're going to call our getter for that instance variable. If users of our ```Person``` class are going to be using that getter or setter, why shouldn't we do the same?

At the bottom of all this is understanding exactly what ```attr_accessor``` and its kin are doing when given a symbol name of an instance variable. Inside the setters and getters created by the ```attr``` family, there will be references directly on the particular attribute. And if we need to ever write our own setters or getters, we do the same. Outside of setters and getters, though, the dog-food principle suggests we will be best off by using the ```self``` keyword.
