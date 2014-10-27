---
layout: post
title: "Curling with Rails"
categories: cURL rails
---

Having built a basic [JSON API](http://www.commandercoriander.net/blog/2014/01/04/test-driving-a-json-api-in-rails/), we are now in a great position to understand how Rails handles HTTP requests, especially in terms of cross-site request forgery. Before we worry about authenticity tokens, let's just get a few basic usages of cURL down.

To start, [here](https://github.com/enocom/json-rails/tree/simple-api) is the Rails 4.0 code against which we will be using cURL. In short, we have a basic CRUD JSON API. The key detail worth noting is the only line in `ApplicationController`:

``` ruby
class ApplicationController < ActionController::Base
  # Prevent CSRF attacks by raising an exception.
  # For APIs, you may want to use :null_session instead.
  protect_from_forgery with: :null_session
end
```

By passing `:null_session` instead of the default `:exception`, we need not worry about retrieving or sending an authenticity token along with our requests to the server. For more on what `:null_session` means, see [here](http://alexcoco.com/forgery-protection-strategy/).

With this detail out of the way, we are now ready to use cURL to interact with our server. Let's start by asking for all the movies in the server's database. Once we have our server running (and our postgres database running, as well), we can issue the following command:

``` bash
curl -v localhost:3000/movies # GET localhost:3000/movies
```

Since we are interested in the request and response details, we pass the `-v` flag to cURL for a verbose output of the actual HTTP request and response. There are all sorts of interesting details in the request and the response. For now, though, note that the response body is simply `[]`, an empty JSON array. We don't have any movies in our server database. Let's fix that with another cURL command:

``` bash
curl -v localhost:3000/movies -X POST \
-H "Accept: application/json" \
-H "Content-Type: application/json" \
-d '{"movie": {"title": "Star Wars: A New Hope"}}'
```

Whereas our previous command created an HTTP request willing to accept any MIME type back, this time, we designate that we want JSON back with the `Accept` header. In our case, this is an irrelevant detail simply because our server is coded to return JSON every time. Nonetheless, for the practice, we will request a JSON response. The `Content-Type` header indicates we are sending a POST request with JSON attached. Finally, the `-d` flag marks the payload, the actual movie data.

If all goes well, we will get a 201 status code, and a JSON representation of our record back in the body of the response.

``` bash
{"id": 1, "title": "Star Wars: A New Hope"}
```

If we turn around and send a GET request to `/movies/1`, we will now see that our record is there. Updating the record is simply a matter of submitting a PUT request to `/movies/1` with a data payload which represents the new information. Submitting a DELETE is easy as well.

For good measure, let's see how to use cURL when CSRF protection is enabled across our app's controllers. Returning to our application controller, we now have:

``` ruby
class ApplicationController < ActionController::Base
  protect_from_forgery with: :exception
end
```

This setting is what will require that any POST, PUT, or DELETE request likewise contain an authenticity token which can be verified against the server's session. So with this new setting in place, let's try to POST a new movie record to the server.

``` bash
curl -v localhost:3000/movies -X POST \
-H "Accept: application/json" \
-H "Content-Type: application/json" \
-d '{"movie": {"title": "Star Wars: The Empire Strikes Back"}}'
```

Instead of the previous 201 status code, we now get a 422 and the `ActionController::InvalidAuthenticityToken` error is raised. There is lots to be said about the authenticity token, and [this](http://stackoverflow.com/questions/941594/understand-rails-authenticity-token) question on StackOverflow is a good place to start. For our purposes here, though, suffice it to say that we now need to pass a cookie (the session) and the authenticity token over to Rails to create our new resource. The way cURL handles cookies makes this all fairly simple.

To start, let's send a simple GET request to the root of our app and see how the server responds.

``` bash
curl -v localhost:3000/
```

In particular, we are interested in the headers of the server response. Amongst all sorts of interesting information, there is one header which pertains to cookies, i.e., `Set-Cookie`. The value of this header is our cookie which in the context of a browser would be stored locally and submitted with all POST, PUT, and DELETE requests. Along with that cookie, the authenticity token is likewise submitted. The token, which appears in the `head` tag of each HTML page is also hidden in an invisible `input` in HTML forms.

I am no security expert, and suggest an interested reader head over to the relevant [Rails Guide](http://guides.rubyonrails.org/security.html) for a more in depth discussion. Nonetheless, the cookie and authenticity token from each response go together as a two-part key of sorts. If the cookie and authenticity token match, then Rails lets any POST, PUT, or DELETE request through.

So how do we handle this with cURL? Easy.

First, we send a GET request to the root, but this time store off the cookie for future use.

``` bash
curl --cookie-jar cookie.txt localhost:3000
```

Once the request returns, in addition to telling cURL to save the cookie off as `cookie.txt` (or whatever file name suits the reader), we need to look through the response body and find the authenticity token in the HTML.

``` html
<meta content="6LhpjoKfxxPQzW/1bdtuwHy3QD2+7oAFetHig0U1RoY=" name="csrf-token" />
```

Note that the reader's own token will be a different value. And, in fact, if we make a subsequent request to the root path, we will get different values for the cookie and authenticity token, too.

Now that we have our cookie in `cookie.txt` and we know our `csrf-token`, we can formulate a POST request to our server which will pass the CSRF protection.

``` bash
curl -v localhost:3000/movies -X POST \
--cookie cookie.txt \
-H "X-CSRF-TOKEN: 6LhpjoKfxxPQzW/1bdtuwHy3QD2+7oAFetHig0U1RoY=" \
-H "Content-Type: application/json" \
-d '{"movie": {"title": "Star Wars: The Return of the Jedi"}}'
```

With the `X-CSRF-TOKEN` header and the correct corresponding cookie, Rails is perfectly convinced we aren't trying a CSRF attack and will happily respond with a 201 status code.

Whether handling the CSRF token in our JSON API is good design is another question. All the same, though, we now know how to send HTTP requests to our Rails server and honor its requirements for authenticity tokens if need be, all without leaving the command line.
