---
layout: post
title: "Using Migrations to Change Columns"
categories: activerecord
---

As an exercise to improve my knowledge of Rails and test-driven development, I'm currently building a clone of [Hacker News](https://github.com/CommanderCoriander/hnews). I have a barebones site put together with minimal styling. At the moment, I'm working on the ```user``` model while using migrations to gradually improve it. Along the way, I'm writing tests in ```rspec``` first and then writing code to make the tests pass. The [Rails Tutorial](http://ruby.railstutorial.org) continues to serve as a critical reference as I progress.

For a passing moment, I thought I might need to change a database column. Whereas my ```user``` model had an attribute ```name```, I considered updating the model to include a ```username``` attribute. Of course, from the method ```user.username``` alone, it's self-evident why that's a bad idea. Instead of touching the model and the database, it's better to just change the value within a view. In other words, when a user goes to the signup page, instead of a ```name``` field, there will be a ```username``` field. The database will simply store that value as ```name``` in the ```user``` table.

Should the opportunity arise to change a column name in a database, though, Rails provides an easy way to do it. Instead of mucking around in past migrations and potentially breaking things in a messy way, it's possible to simply generate a migration which changes the column's name.

```ruby
rails generate migration ChangeOldColumnToNewColumn
```

The body of the migration is equally simple (as explained in a post from [stackoverflow](http://stackoverflow.com/questions/1992019/how-can-i-rename-a-database-column-in-a-rails-migration)):

```ruby
rename_column :table, :old_column, :new_column
```

By staying within the migration workflow, it's easy to avoid any messy problems which would almost certainly be introduced by manually editing past code.
