---
layout: post
title: "Test Driving Flatten in Javascript with Grunt"
categories: grunt javascript
---

As a way to improve my understanding of JavaScript, I have been gradually re-implementing all the functions provided by [Underscore.js](http://underscorejs.org/). For an added challenge, I have been doing it using test-driven development. It's an excellent exercise and has been a good way to learn about the exciting new world of [Grunt](http://gruntjs.com/). My copy-cat library is called [doublescore.js](https://github.com/enocom/doublescore.js). In effect, writing the library is a more engaged reading of Underscore's super-awesome [annotated source](http://underscorejs.org/docs/underscore.html).

In particular, I have been using a combination of [grunt-contrib-jasmine](https://github.com/gruntjs/grunt-contrib-jasmine) (run headlessly on the command line) and [grunt-contrib-watch](https://github.com/gruntjs/grunt-contrib-watch). For good measure, I am also using [grunt-contrib-jshint](https://github.com/gruntjs/grunt-contrib-jshint) to catch all the stupid syntax mistakes that so easily creep into JavaScript code. My setup is otherwise minimal as I like to understand how all the pieces fit together and keep things simple. A Gruntfile for the project may be found [here](https://github.com/enocom/doublescore.js/blob/master/Gruntfile.js).

With [node](http://nodejs.org/) and npm installed (easily done through Homebrew), as well as all the project's dependencies installed (using `npm install`), we can start watching the test suite and code for changes with `grunt watch`. From there, we are ready to start writing a test.

Before we write `flatten`, let's start with a basic function which will come in handy: `each`.

To start, we will write a test in `tests/doublescore_spec.js`.

``` javascript
describe("__.each", function() {
  it("iterates over an array calling a function for each item", function() {
    var arr = [1, 2, 3],
        memo = [],
        iterator = function(value) { memo.push(value); };

    __.each(arr, iterator);

    expect(memo).toEqual([1, 2, 3]);
  });
});
```

We start by declaring an array of values and an empty memo array. Finally, we also declare an iterator, which will simply push whatever value it receives onto the memo object. From there, we call the `__.each` method passing our array and the iterator. We then assert that our memo did in fact receive all three values from the original array.

Once we save our test, Grunt will complain that `undefined` is not a function. In other words, there is no such `each` function. Let's fix that by writing an implementation of `each` in `lib/doublescore.js`. (Note that I am eliding the steps of only writing enough code to get a new error, which is what a proper practice of TDD includes.)

``` javascript
__.each = function(coll, iterator) {
  var index, length;
  for (index = 0, length = coll.length; index < length; index++) {
    iterator.call({}, coll[index]);
  }
};

```

The most interesting detail in the function above is the use of `call`. I will leave it to [Mozilla](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/call) to give a full explanation. Suffice it to say that we are invoking our function with each value in our array.

Now with `each` in hand, we are ready to tackle `flatten`. Let's start with a test:

``` javascript
describe("__.flatten", function() {
  it("flattens a nested array", function() {
    var nested = [[1, [2, [3, [4]]]], 5, [6, 7]],
        expected = [1, 2, 3, 4, 5, 6, 7];

    expect(__.flatten(nested)).toEqual(expected);
  });
});

```

The test is fairly simple. We pass a nested array to `flatten` and expect it to come out without any sub-arrays. The implementation is much more interesting.

``` javascript
__.flatten = function(coll) {
  return flatten(coll, []);
};
```

We will start with the main interface which in turn delegates to a function which we can use recursively. The private method is where all the action happens:

``` javascript
var flatten = function(coll, memo) {
  each(coll, function(value) {
    if (value instanceof Array) {
      flatten(value, memo);
    } else {
      memo.push(value);
    }
  });

  return memo;
};
```

With `each` already in place, we can write a clean implementation of `flatten`. Basically, we iterate over each index in the initial array, checking if it's an array using the `instanceof` keyword. If the value is a non-array, we simply push it onto our memo array. Otherwise, we make a recursive call to `flatten`, passing the sub-array and the memo object. And just like that, we have a working implementation of `flatten`.

Granted, our version of `flatten` isn't nearly as sophisticated as that of Underscore.js. Nonetheless, we have a starting point from which to formulate a more robust algorithm.
