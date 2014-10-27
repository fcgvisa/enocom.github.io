---
layout: post
title: "ActiveRecord and Serialization Revisited"
categories: activerecord
---

Not long after posting about how ActiveRecord [serializes a Ruby object](http://www.commandercoriander.net/blog/2013/02/25/how-activerecord-serializes-a-ruby-object/), a friend showed me the `serialize` method. Although there are likely cases where converting a Ruby object to YAML has its uses, ActiveRecord in fact supports storing objects within a database for later retrieval.

For example, assume we have a `User` class with a `preferences` attribute. (Note: the following example comes from this [page](http://api.rubyonrails.org/classes/ActiveRecord/Base.html) in the Ruby on Rails API.)

``` ruby
class User
  attr_accessible :preferences
end
```

We then create a user, storing a hash within `preferences` (a string column in the `users` table):

``` ruby
u = User.create :preferences => { :background => "black" }
```

When we look up `preferences`, it will come back to us as YAML.

``` ruby
u.reload
u.preferences # => "---\n:background: black\n"
```

While we could run this through the YAML `load` function to recreate our hash, there is a nicer way. Enter `serialize`.

``` ruby
class User
  attr_accessible :preferences
  serialize :preferences
end
```

By calling the `serialize` method with whatever attributes we wish to serialize, we're telling ActiveRecord to store and return Ruby objects, in particular arrays and hashes. Observe.

``` ruby
u = User.create :preferences => { :background => "black" }
u.reload
u.preferences # => { :background => "black" }
```

We create our object specifying a Ruby hash, store it in the database, and then upon reloading, that object returns to us as we inserted it. Beautiful. This works for "arrays, hashes, and other non-mappable objects" according to the API.

So clever tricks aside, why bother? In my mind, the `serialize` function is an excellent alternative to creating a new table. While we could create a `preferences` table and associate it with the `user` table through a `belongs_to` -- `has_many` relationship, it certainly seems that an additional table is overkill, especially since in the example above we will only be storing a couple of key-value pairs.