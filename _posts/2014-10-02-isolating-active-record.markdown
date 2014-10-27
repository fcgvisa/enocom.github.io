---
layout: post
title: "Isolating Active Record"
categories: domain-driven-design
---

In building large Rails apps with test-driven development, I have often struggled with ways to minimize the presence of ActiveRecord objects in the tests. In an ideal world, unit tests would rarely if ever hit the database. And yet in common practice in a Rails app, every level of the [testing pyramid](http://martinfowler.com/bliki/TestPyramid.html) reads from and writes to the database. The inevitable, resulting slowdown derives almost exclusively from the way Rails encourages the use of ActiveRecord objects, as if they were the only tenable currency of the realm.

Now that "fat model, skinny controller" has lost its appeal -- if you don't believe me, ask any developer who has to maintain an app with models whose line count numbers in the hundreds if not the thousands -- the question remains, what can we do to isolate ActiveRecord?

The world of [domain driven design](http://en.wikipedia.org/wiki/Domain-driven_design) and hexagonal architecture provides some exciting alternatives. For those new to the subject, Uncle Bob has a great [blog post](http://blog.8thlight.com/uncle-bob/2012/08/13/the-clean-architecture.html) on the subject and a [recent talk](https://www.youtube.com/watch?v=_rbF97T4480) at RailsConf also gives a well-presented rundown. See also, Eric Evan's [book](http://www.amazon.com/Domain-Driven-Design-Tackling-Complexity-Software/dp/0321125215) by the same name.

Among all the ideas of domain driven design, the repository pattern is especially relevant. Rather than mix our business logic and persistence needs in a single object as ActiveRecord encourages, the repository pattern forces the separation of business logic from persistence. The repository operates on entities, distinct representations of ideas from a business domain, and is charged only with persistence.

Let's talk about some code. As usual, skip to [GitHub](https://github.com/enocom/json-api/tree/control-flow-repo) for the TL;DR. What follows are annotations to my attempt at writing a proper repository.

To start, let's assume we want to build an application to store movies. The usual approach would be to create a `Movie` class in the `models` directory. Instead, we will create a `daos` (i.e., database access objects) directory to keep the ActiveRecord objects which know how to persist themselves to a SQL database. Here's what our `MovieDao` looks like:

``` ruby
require "active_record"

class MovieDao < ActiveRecord::Base
  self.table_name = :movies

  validates_presence_of :title, :director
end
```

Granted, one could argue that validation is a separate concern, but overlooking that little detail, our `MovieDao` follows the single responsibility principle. It is charged with writing data to the database.

Before we discuss the repository, let's take a quick look at the entity object -- what will represent the domain object. The entity will be the currency of the application, and anyone who wants to do anything with movies will operate on entities. Any changes to an entity, however, will have to pass through the repo if there are to be persisted. The entity is, in effect, nothing more than a bag of data:

``` ruby
require "active_model"

class MovieEntity

  include ActiveModel::Serializers::JSON

  attr_reader :id, :title, :director

  def initialize(attributes)
    @id        = attributes[:id]
    @title     = attributes[:title]
    @director  = attributes[:director]
  end

  def attributes
    instance_values.symbolize_keys
  end

  def persisted?
    !id.nil?
  end

end
```

There is some boilerplate here which makes the entity play well with serialization (e.g., the JSON module) and which makes accessing the data easy for ActiveRecord (e.g., the `#attributes` method). Otherwise, the entity holds only three values. Note that once we set the attributes in `#initialize`, we expose them on a read-only basis.

And, finally, for the interesting part: the repository. The full code for the repo can be found [here](https://github.com/enocom/json-api/blob/control-flow-repo/app/repositories/movie_repository.rb). The repo supports all the CRUD actions, but we will look only at the `#add` function. Once `#add` makes sense, the rest of the methods are straightforward and easy to understand.

This is what `#add` looks like:

``` ruby
require_relative "../daos/movie_dao"
require_relative "../entities/movie_entity"
require_relative "../shared/store_result"
require_relative "../shared/error_factory"

class MovieRepository
  def initialize(factory = MovieFactory, dao = MovieDao)
    @factory = factory
    @dao = dao
  end

  def add(entity)
    record = dao.new(entity.attributes)

    if record.save
      return successful_result(record)
    end

    failed_result_with_errors(record.errors)
  end

  # ...

  private

  attr_reader :factory, :dao

  def successful_result(record = nil)
    entity = record ? factory.create(record) : nil

    StoreResult.new(
      entity: entity,
      success: true,
      errors: nil
    )
  end

  def failed_result_with_errors(errors)
    StoreResult.new(
      entity: nil,
      success: false,
      errors: ErrorFactory.create(errors)
    )
  end

end
```

Let's start with `#initialize`.

The repo takes two dependencies: 1) a factory to inflate entities from ActiveRecord objects (see [here](https://github.com/enocom/json-api/blob/control-flow-repo/app/factories/movie_factory.rb)), and 2) a database access object, otherwise known as our familiar ActiveRecord `MovieDao` object from above.

Let's walk through the `#add` method in detail. First, it's worth noting that the clients of the repo are charged with passing in already inflated entities. The repo expects movie entities with a `title` and `director` attribute already set. From there, we build an ActiveRecord object in memory with the entity's attributes and then attempt to save it.

It's at this moment that the repo does something interesting. If the attempt to save is successful, we _do_ _not_ pass back the ActiveRecord object. And for that matter, we do not pass back a raw entity, either. In order to pass back some information about the save attempt, we return a `StoreResult` object.

A `StoreResult` is an extremely simple object:

``` ruby
class StoreResult

  attr_reader :entity, :errors

  def initialize(entity:, success:, errors:)
    @entity = entity
    @success = success
    @errors = errors
  end

  def success?
    success
  end

  private

  attr_reader :success

end
```

By wrapping our return value in a `StoreResult` object, we have a clean way to return the entity, the result of the store attempt (e.g., true or false), and any errors which may be attached to the ActiveRecord object. Using `StoreResult` we keep any kind of ActiveRecord object isolated to the repo layer.

This is the most important point of the repository. The ActiveRecord object is nothing more than a private implementation detail of the repository and we strictly enforce the encapsulation of that detail. By returning a `StoreResult` object, we ensure that consumers of the repo receive nothing more than the entity in question, some information about the result of the attempt to persist the entity, and, if necessary, any errors related to the validity of the entity.

In producing a `StoreResult` object which wraps our entity, the repo will alternatively make a call to `#successful_result` or `#failed_result_with_errors`. Both methods create a `StoreResult`. In case our attempt at saving the ActiveRecord object fails, we'll pass the associated errors to the `#failed_result_with_errors` method which will create a basic errors object to be consumed elsewhere in the system.

One obvious benefit to the repository pattern is the way persistence becomes an implementation detail. If in the future, we have to persist our data across an HTTP request, we have to rewrite only our repository.

Another big win with using the repository is the way it frees up our need to connect to a database in test. By swapping our repository out with a fake repository which honors the same interface but otherwise lives only in memory, we can completely remove the need to hit the database in test at the unit level.

More importantly, though, by using entities and repositories in place of a single ActiveRecord object, we have honored the single responsibility principle. Persistence is the repository's responsibility, and our entity's responsibility is simply to model the  behavior inherent to the domain object it represents.

Note: I'm grateful to [Joe Rodriguez](https://github.com/joerodriguez) whose idea for a `StoreResult` object I use in my repo. See his original implementation [here](https://github.com/joerodriguez/athletes-api-rmr-2014).
