---
layout: post
title: "Testing around Random Numbers in JavaScript"
categories: javascript tdd
---

Recently, while test-driving [doublescore.js](https://github.com/enocom/doublescore.js), my Underscore.js copycat library, I came across an interesting problem. Suppose we want to write `__.sample`, a method which takes a collection and returns a random element. If the return value is always random, how will we ever write a meaningful assertion in our tests?

Although it might sound like a vexing problem, Jasmine gives us a nice solution. First, here's the interface that we want:

``` javascript
__.sample([1, 2, 3]) // should return a random selection of 1, 2, or 3
```

So how do we write a test which will always have the same return value? This is where Jasmine's `spyOn` method comes in handy. Granted, by using `spyOn`, our test will become somewhat aware of how `__.sample` works under the hood, but since we're not passing our random number generator in as a dependency (another option here), we'll just have to live with that concession.

In short, we'll use `spyOn` to guarantee that any call to `Math.random()` returns the same value each time. From there, it's fairly easy to assert that `__.sample` will return the same value, as well.

``` javascript
describe("__.sample", function() {
  it("return a random element from a collection", function() {
    // note that I'm using Jasmine 2.0 here
    spyOn(Math, "random").and.returnValue(0.5);

    expect(__.sample([1, 2, 3])).toEqual(2);
  });
});
```

As long as `__.sample` uses `Math.random()` in its generation of a random index from which to pull from the passed collection, our assertion will pass. In the interest of keeping this post short, I'll only link to the implementation of `__.sample` [here](https://github.com/enocom/doublescore.js/blob/c37d4a2e241118a1924dbc0f21166ed8c6847986/lib/doublescore.js#L295).

Where this gets interesting is in our next test. Suppose we wanted to test that `__.sample` takes an optional argument to designate the sample size, e.g.,

``` javascript
__.sample([1, 2, 3], 2) // should return an array of two random elements
```

Whereas our previous test stubbed `Math.random()` to always return one value, we now want to return two distinct values. How does that work?

This is where Jasmine shines:

``` javascript
describe("__.sample", function() {
  it("takes an optional sample size", function() {
    var callCount    = 0,
        firstNumber  = 0.5,
        secondNumber = 0.1,
        numberGenerator = function() {
          if (callCount++ === 0) return firstNumber;
          return secondNumber;
        };
    spyOn(Math, "random").and.callFake(numberGenerator);
    expect(__.sample([1, 2, 3], 2)).toEqual([2, 1]);
  });
});
```

Now, we have set up our test such that any call to `Math.random()` will instead call our own number generator. The number generator simply returns one of two numbers. If the call count is zero, it returns the first number. And once the call count reaches one, it returns the second number. Note that the postfix `++` operator will increment `callCount` only after the boolean expression is evaluated.

Again, the test knows a little bit too much about the implementation -- namely that `Math.random()` is used at all. Nonetheless, unless we refactor `__.sample` to accept a number generator as an argument, this kind of stubbing will have to do.
