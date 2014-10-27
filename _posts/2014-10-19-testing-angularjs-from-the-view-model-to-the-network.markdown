---
layout: post
title: "Testing AngularJS from the View-Model to the Network"
---

Recently, I have been building a single page application in AngularJS which consumes a Rails API. Now that some of the basic concepts have sunk in, I find it a joy to work with the client-side framework. In particular, testing is especially painless. It helps, though, to have a few basic examples to work from and so my interest in this post is to walk through a simple testing arc starting with the view-model and going all the way to the API.

So that we can skip all the mundane setup details, here is the [client repo](https://github.com/enocom/angular_client). I'm using Grunt as a build tool and running tests with Karma. Note that the client lives entirely separate from its API. If were were to deploy this app, we might deliver the static assets from an S3 bucket (see [here](http://www.nickdobie.com/blog/hosting-angularjs-with-amazon-s3/) for an example) and host the Rails API on Heroku.

Let's start by assuming we have a set of API endpoints already written. I'll be using my [json-api](https://github.com/enocom/json-api) app here which supports basic CRUD actions for movie records. In particular, we have an endpoint at `/api/movies` which responds to `GET` requests.

Our first step is to wire up the view. For now, we'll just have a list of all movies available through the API endpoint:

``` html
<div ng-controller="MovieListCtrl as list">
  <ul ng-repeat="movie in list.movies">
    <li>{% raw %}
      <span>{{movie.title}} - Directed By {{movie.director}}</span>{% endraw %}
    </li>
  </ul>
</div>
```

This little piece of HTML will be the only item we have on our barebones page. I've omitted the usual HTML boilerplate (e.g., `html`, `head`, and the `body` tags). Note that I'm using the "controller as" syntax which allows me to put all controller data on the controller itself. Whereas we might refer to the list of movies as `$scope.movies`, here we're just using `list.movies`.

The next step is to write a test for our presently non-existent `MovieListCtrl`. The main responsibility of this `MovieListCtrl` is that as soon as Angular instantiates the controller, we want to immediately go to the API and retrieve all the movies. In other words, we will have to stub the service which retrieves the movies and pass that stubbed service into the controller as one of its dependencies. Here's the necessary setup:

``` javascript
"use strict";

describe("MovieListCtrl", function () {
  var MovieListCtrl,
    $scope,
    moviesDeferred,
    MovieService,
    $q;

  beforeEach(module("client"));

  beforeEach(inject(function ($controller, $rootScope, _$q_, _MovieService_) {
    $scope = $rootScope.$new();
    MovieService = _MovieService_;
    $q = _$q_;

    moviesDeferred = $q.defer();

    spyOn(MovieService, "getList").and.returnValue(moviesDeferred.promise);

    MovieListCtrl = $controller("MovieListCtrl", {
      MovieService: MovieService
    });
  }));

  // assertions go here

});

```

Aside from gaining reference to our module with `module("client")`, we have a `beforeEach` block which creates our controller with all the necessary dependencies. We'll inject the `$controller` object to create our controller. We'll inject the `$rootScope` object to have a reference to `$scope` (for reasons we'll see below). We'll also ask for the `$q` object and our currently non-existent `MovieService` whose responsibility will be to perform all network requests.

With all the necessary dependencies injected and referenced with local variables, we have two important tasks remaining. First, we have to spy on the actual method call made by the service to retrieve the movies. In addition to stubbing that method and preventing a network request in test (which we never want), we'll ensure that the call returns a promise, helpfully created for us using the `$q` object. We then new up the `MovieListCtrl` using the `$controller` object passing in whatever dependencies the controller might have.

As a quick aside, it's worth noting that when passing arguments to Angular's `inject` function, we refer to `$q` and the `MovieService` with underscores before and after the object. From Angular's perspective, the underscores are absolutely meaningless and are otherwise ignored. For our own purposes, though, by wrapping the objects with underscores, we can still use variables in the test file whose name lines up with the actual objects. Hence the `$q` object is referred to as `$q`, rather than `_$q_`.

With all the pieces ready to go, we're can finish writing our first test. Here's the actual assertion:

``` javascript
it("populates a list of movies", function () {
  MovieService.movies = [
    {id: 1, title: "The Shining", director: "Stanley Kubrick"},
    {id: 2, title: "A Clockwork Orange", director: "Stanley Kubrick"}
  ];
  moviesDeferred.resolve();
  $scope.$apply();

  expect(MovieListCtrl.movies).toEqual([
    {id: 1, title: "The Shining", director: "Stanley Kubrick"},
    {id: 2, title: "A Clockwork Orange", director: "Stanley Kubrick"}
  ]);
});
```

Knowing that our instantiated controller will have already made a call to the `MovieService` at this point, we simulate a successful `GET` request on the service's part and assign some movies onto the service object, we then resolve our promise (which will bring us through a `.then` chained function) and call `$scope.$apply()` to update our controller object. And then (finally!), we're ready to make an assertion that the `movies` property on the controller will be populated with whatever movies were present on the service object.

When we run the tests, Jasmine is the first to complain about there being no such `MovieService`. Let's quickly make one without adding any behavior to it:

``` javascript
(function () {
  "use strict";

  function MovieService () {
    var service = {};

    service.getList = function () {};

    return service;
  }

  angular
    .module("client")
    .service("MovieService", MovieService);
})();
```

We're just defining the `getList` function so Jasmine has something on which to spy. Otherwise, nothing to see here yet.

With that our first test should be giving a more meaningful failure: "Expected undefined to equal [our list of movies]." So we're ready to start writing the controller.

``` javascript
(function () {
  "use strict";

  function MovieListCtrl () {
    var vm = this;
  }

  angular
    .module('client')
    .controller('MovieListCtrl', MovieListCtrl)
})();
```

The basic skeleton in wrapped inside an immediately invoked function expression. Also, note that for readability we defined a named function first and then pass it to Angular's controller function. Also, another convenience is storing off a reference to the controller immediately as `vm` or "view-model." We'll update the controller to take `MovieService` as a dependency and immediately make a call to it.

``` javascript
function MovieListCtrl (MovieService) {
  var vm = this;

  MovieService.getList()
    .then(function () {
      vm.movies = MovieService.movies;
    });
}
```

As soon as the controller is instantiated we make a call to `getList` and once the call resolves successfully, we define what should happen by chaining a `then` function onto the promise implicitly returned by the `MovieService`. See the [documentation](https://docs.angularjs.org/api/ng/service/$q#!) on `$q` for more details of the promise API. Once the `MovieService` returns from its network call, we ask the service for all the movies and store a reference to them on the controller.

With that our controller test is passing. But our app is far from working. To finish the job, we have to write the necessary code for the `MovieService`.

Let's start with a test:

``` javascript
"use strict";

describe("MovieService", function () {
  var MovieService,
    $httpBackend;

  beforeEach(module("client"));

  beforeEach(inject(function (_$httpBackend_, _MovieService_) {
    $httpBackend = _$httpBackend_;
    MovieService = _MovieService_;
  }));

  // actual assertions
});
```

The setup for the `MovieService` is much easier in comparison. We inject `$httpBackend` for mocking out the network calls, and we inject our object under test.

``` javascript
it("GETs all the movies", function () {
  $httpBackend
    .expectGET(/\/api\/movies/)
    .respond([
      {id: 1, title: "The Shining", director: "Stanley Kubrick"}
    ]);

  MovieService.getList().then(function () {
    expect(MovieService.movies).toEqual([
      {id: 1, title: "The Shining", director: "Stanley Kubrick"}
    ]);
  });

  $httpBackend.flush();
});
```

The test is made of up three parts. First, we make an expectation for the actual network activity. Here we're going to expect a `GET` request to `/api/movies` and we'll return a sample bit of movie data as a response.

Second, we actually invoke our service object by calling `getList`. When that function resolves, we make another expectation that the service has stored off a reference to whatever the network response might have been. This particular step ensures that we're extracting the data attribute from the response and storing it off.

Finally, we make a call to `$httpBackend.flush()` to flush all the pending network requests to ensure our object only performed exactly the network request we expected.

The test fails and now we reach the last step of our testing arc:

``` javascript
(function () {
  "use strict";

  function MovieService ($http) {

    var service = {},
      endpoint = "http://localhost:3000/api/movies/";

    service.movies = [];

    service.getList = function () {
      return $http
        .get(endpoint)
        .then(function (response) {
          service.movies = response.data;
        });
    };


    return service;
  }

  angular
    .module("client")
    .factory("MovieService", MovieService);
})();
```

Within the service, we start by declaring an object onto which the public interface will go. There will be a `movies` collection and a `getList` function. The `getList` function simply delegates its heavy lifting to the injected `$http` object. Once `$http` returns successfully from its network call, we strip the `data` off the `response` object and assign it to the `movies` collection.

I've glossed over handling different endpoint URLs here and instead hard-code the Rails API server's address. If you're curious about how to better handle URLs which change between development and production, I recommend looking at [ng-constant](https://www.npmjs.org/package/grunt-ng-constant).

With that our tests are passing, and our barebones app should now display whatever movies we have in the API service.
