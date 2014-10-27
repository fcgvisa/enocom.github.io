---
layout: post
title: "How ActiveRecord Serializes a Ruby Object"
categories: activerecord ruby yaml
---

Let's assume we have a table named ```posts```. The table has only one column, ```title```, which will store just that: a post's title. For this hypothetical exercise, we'll be using ```ActiveRecord``` as our ORM.

Let's also assume that we're using the handy ```Faker::Lorem.words()``` function to generate some test data. We'll pass the number three to the ```words``` function to get what will stand in as a passable title for testing purposes. Finally, let's assume that we forget that ```words``` returns an array of words, rather than a string of three words.

What happens when we do the following?

``` ruby
post = Post.create(:title => Faker::Loream.words(3))
```

Will ```ActiveRecord``` throw an error about storing a Ruby object in the database? Or, will it do something else?

This past week, I found myself overlooking this detail and getting some strange behavior, which upon closer inspection makes sense. When I retrieved the test post's title from the database, look what came back:

``` yaml
  --- -lorem -ipsum -dolor
```

It's Yaml! Instead of complaining about receiving a non-string, ```ActiveRecord``` will serialize a Ruby object using Yaml. What strange and unexpected behavior.
