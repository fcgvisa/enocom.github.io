---
layout: post
title: "Testing Angular Services with Dependencies"
categories: angularjs
---

One of the nicest features of AngularJS is its inherent testability.

Testing functions with dependencies is easy. One simply passes in mock objects to the function in question in a test. Likewise, testing controllers with dependencies is also fairly straightforward. One simply instantiates a controller, giving it whatever mocked dependencies are necessary, and then proceeds with the test. If we skip the details of the setup (see [here](http://commandercoriander.net/blog/2014/10/19/testing-angularjs-from-the-view-model-to-the-network/) for those details), newing up the controller looks something like the following:

``` javascript
MyCoolCtrl = $controller("MyCoolCtrl", {
  MyCoolService: MyMockedService
});
```

We can mock the service object in whatever way we like and then hand that mocked object over to the controller. Then after invoking some number of controller functions, we can easily verify the mock is used in some way or another.

What then does a similar test of a service with dependencies look like?

Say, for instance, we want to write a simple service to wrap `localStorage`. We know that we cannot pass in the real `window` object in the test as it does not exist in that context. Hence, `$window` -- Angular's convenient wrapper around the `window` object. But how do we pass `$window` to our service? Unlike the `$controller` method above, we do not have a similar method for instantiating service objects without short-circuiting Angular's dependency injection.

Enter `$provide`. We can swap out `$window` for a mock window object whose interface matches the real `$window`, at least insofar as our object under test in concerned.

Let's walk through a test for our envisioned `LocalStorageService`. First, we can assure that when an object asks for `$window`, it will instead get a mock window, which we control.

``` javascript
describe("LocalStorageService", function () {
  // ... mockWindow definition goes here

  beforeEach(module("myApp", function ($provide) {
    $provide.value("$window", mockWindow);
  }));

  // ... tests go here
```

In the lines above, we are configuring the `$window` provider and telling it to instead hand over our `mockWindow` object. What does `mockWindow` look like?

Since we know the interface of `localStorage` (see [here](https://developer.mozilla.org/en-US/docs/Web/API/Storage/LocalStorage)), we could could create a `mockWindow` which would look something like the following:

``` javascript
describe("LocalStorageService", function () {
  var LocalStorageService,  // a local variable for our object under test
    mockWindow = {          // <---------- This is our fake window object
      localStorage: {
        _storage: {},
        getItem: function (k) {
          return this._storage[k];
        },
        setItem: function (k, v) {
          this._storage[k] = v;
        },
        removeItem: function (k) {
          delete this._storage[k];
        }
      }
    };

  // ... provider config from above

  // ... our tests will go here
});
```

Even though JavaScript does not make it explicit, our `mockWindow` object has a `localStorage` which, implementation aside, has all the relevant methods we require of the true `localStorage` object. In effect, we have built a fake window object for use in test that looks exactly like the real `$window` object.

From there, it's trivial to write our tests:

``` javascript
describe("LocalStorageService", function () {
  // ... LocalStorageService and mockWindow declaration

  // ... configuration of the $window provider

  // we inject our service and store it off
  // in the local variable declared above
  beforeEach(inject(function (_LocalStorageService_) {
    LocalStorageService = _LocalStorageService_;
  }));

  it("returns a value for a key", function () {
    mockWindow.localStorage.setItem("a key", "a value");

    expect(LocalStorageService.getKey("a key")).toEqual("a value");
  });

  it("stores a value for a key", function () {
    LocalStorageService.setKey("my key", "my value");

    expect(mockWindow.localStorage.getItem("my key")).toEqual("my value");
  });

  it("removes a value for a given key", function () {
    mockWindow.localStorage.setItem("bad key", "bad value");

    LocalStorageService.remove("bad key");

    expect(mockWindow.localStorage.getItem("bad key")).toEqual(undefined);
  });
});
```

And with that, we have a clean test with our service's dependency on `$window` replaced by a mock object which we can easily verify.

For those curious, the implementation of `LocalStorageService` might look like the following:

``` javascript
(function () {
  "use strict";

  function LocalStorageService($window) {
    function getKey(k) {
      return $window.localStorage.getItem(k);
    }

    function setKey(k, v) {
      $window.localStorage.setItem(k, v);
    }

    function remove(k) {
      $window.localStorage.removeItem(k);
    }

    // we return the public interface of our object
    return {
      getKey: getKey,
      setKey: setKey,
      remove: remove
    };
  }

  angular
    .module("myApp")
    .service("LocalStorageService", LocalStorageService);
})();
```
