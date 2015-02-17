---
layout: post
title: "Selecting Matches across Multiple Columns in Postgres"
categories: postgres
---

Suppose we have a `persons` table with a `first_name` and `last_name` column. Now suppose, we want to query that table based on user input. Imagine a text box in a webclient that allows a user to type a first name and a last name. Some JavaScript code would then submit the user input to a server endpoint, which would then trigger a lookup in Postgres.

A naive solution might look like the following (using ActiveRecord):

``` ruby
Person.where("first_name ILIKE (?) OR last_name ILIKE (?)",
             "%#{params[:query]}%",
             "%#{params[:query]}%")
```

Note that `ILIKE` for those who aren't aware means "case insensitive like."

The problem with this approach is that we assume a user has entered _only_ a first name or _only_ a last name. But we want to allow a user to enter a first name and a last name. What we need to do is to test user input against a concatenation of the `first_name` and `last_name` columns.

Fortunately, and even remarkably, Postgres makes this a breeze. In plain SQL, we might do something like the following:

``` sql
SELECT * FROM persons
WHERE first_name || ' ' || last_name
ILIKE 'jane doe';
```

The peculiar pipes above -- otherwise recognizable as a boolean OR -- are the operators for string concatenation in Postgres. In effect, we are comparing a concatenated `first_name`, a space, and `last_name` against some string (i.e., `jane doe`).

This translates easily into an ActiveRecord `where` query:

``` ruby
Person.where("first_name || ' ' || last_name ILIKE (?)",
             "%#{params[:query]}%")
```

And just like that, we can return records based on matches across two columns.

For more details on string functions and operators in Postgres, see [here](http://www.postgresql.org/docs/current/static/functions-string.html).
