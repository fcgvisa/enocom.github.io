---
layout: post
title: "Test Driving a JSON API in Rails"
categories: rails api tdd
---

Eventually, anyone working with Rails will need to build a RESTful API, whether it be for a single-page application or for a mobile client. The exercise is simple enough, but is worth doing on one's own for the practice. It's also nice to have a sandbox against which to practice HTTP requests with cURL -- my own motivation behind the exercise.

For this post, we will be test-driving a basic JSON API. For those who might like to jump ahead and just peruse the code, the repo may be found on [Github](https://github.com/enocom/json-api/tree/simple-api). When we are done, we will support the following routes for managing movie records:

```
GET    /movies
POST   /movies
GET    /movies/:id
PUT    /movies/:id
DELETE /movies/:id
```

After creating a new Rails app and adding RSpec and FactoryGirl to the Gemfile, we are ready to write our first request spec.

``` ruby
# spec/requests/movies_spec.rb
describe "Movies API" do
  describe "GET /movies" do
    it "returns all the movies" do
      FactoryGirl.create :movie, name: "The Lord of the Rings"
      FactoryGirl.create :movie, name: "The Two Towers"

      get "/movies", {}, { "Accept" => "application/json" }

      expect(response.status).to eq 200

      body = JSON.parse(response.body)
      movie_titles = body.map { |m| m["title"] }

      expect(movie_titles).to match_array(["The Lord of the Rings",
                                           "The Two Towers"])
    end
  end
end
```

Before we make this test pass, there are a few details worth discussing. First is the way we are simulating a `GET` request against the API, i.e.,

``` ruby
get "/movies", {}, { "Accept" => "application/json" }
```

We could have likewise used the `xhr` method like so:

``` ruby
xhr :get, "/movies"
```

For the tests here, we will use the more verbose `get` method for the practice. The `get` method takes three arguments: a path, a set of HTTP parameters, and any additional headers to be included in the request. In the case here, we aren't sending any parameters with our request. We do, however, need to specify the `Accept` header to indicate that we want JSON back from the server.

Another detail worth noting is the `response` variable which will be set following our request. We can then parse the body of the response using `JSON`. From there, we have access to whatever may come back in the body, such as the titles of our movies.

To get this test to pass, we have a few things to do. We need to define a movie factory using FactoryGirl. The factory will in turn require us to create a movie model, which will need a migration to create a movies table. For now, our movie model will need only a title attribute. Since this is all fairly standard, I will leave it to the curious reader to look through the code on Github for the details.

Once we have our movie factory and `Movie` model created, our next test run will complain about a missing route. Let's add a route:

``` ruby
# config/routes.rb
JsonRails::Application.routes.draw do
  resources :movies, only: [:index]
end
```

With a route defined, we get one error closer to a passing test. Running the test again now complains about a missing controller. So we create our `MoviesController`. At each step, we do only enough to reach a new error. After getting yet another error about a missing index action, we end up with the following controller.

``` ruby
# app/controllers/movies_controller.rb
class MoviesController < ApplicationController
  def index
  end
end
```

Running the test once more complains about a missing template since Rails assumes we want to render an HTML page. Instead, we tell Rails to send back JSON representing all our movies.

``` ruby
class MoviesController < ApplicationController
  def index
    render json: Movies.all
  end
end
```

And with that, we have our first passing test. Now that we have a basic pattern to follow, the remaining routes are fairly easy in comparison.

The next endpoint will be `/movies/:id`.

``` ruby
describe "Movies API" do
  # ...
  describe "GET /movies/:id" do
    it "returns a requested movie" do
      m = FactoryGirl.create :movie, title: "2001: A Space Odyssey"

      get "/movies/#{m.id}", {}, { "Accept" => "application/json" }

      expect(response.status).to eq 200

      body = JSON.parse(response.body)
      expect(body["title"]).to eq "2001: A Space Odyssey"
    end
  end
end
```

The code to make the test pass isn't much different from that above. After adding the `show` method to the list of routes, we have the following in our controller:

``` ruby
# app/controllers/movies_controller.rb
class MoviesController < ApplicationController
  # ...
  def show
    render json: Movie.find(params[:id])
  end
end
```

So far whenever we've simulated a request against our API, we have only had to specify the `Accept` header and have otherwise ignored the parameters passed along with the request. When it comes to creating movie resources, though, we naturally have to pass up whatever information will be stored in the movies table.

``` ruby
describe "Movies API" do
  # ...
  describe "POST /movies" do
    it "creates a movie" do
      movie_params = {
        "movie" => {
          "title" => "Indiana Jones and the Temple of Doom"
        }
      }.to_json

      request_headers = {
        "Accept" => "application/json",
        "Content-Type" => "application/json"
      }

      post "/movies", movie_params, request_headers

      expect(response.status).to eq 201 # created
      expect(Movie.first.title).to eq "Indiana Jones and the Temple of Doom"
    end
  end
end
```

Aside from the `movie_params` that we pass along with our `POST` request, we also have to specify the `Content-Type` header, which tells the server the MIME type of the body of our request. As usual, `Accept` tells the server that we want JSON back.

Since the code for `DELETE` and `PUT` is quite similar to `GET` and `POST`, I won't introduce it here. Again, the curious reader may look at the repo on Github for the details. The spec is [here](https://github.com/enocom/json-rails/blob/master/spec/requests/movies_spec.rb) and the controller is [here](https://github.com/enocom/json-rails/blob/master/app/controllers/movies_controller.rb).

Ultimately, it's a simple exercise, building a RESTful JSON API. Nonetheless, in the process of test-driving the API, one can learn quite a bit about the mundane, but still important details of the HTTP request cycle. Finally, after building the API, we now have a toy interface against which to experiment with cURL.

## Update

A previous version of this post suggested using `respond_to` and `respond_with`. I've updated the code examples to use the much simpler and much preferred `render json: {}` instead.
