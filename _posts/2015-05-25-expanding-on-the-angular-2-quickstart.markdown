---
layout: post
title: "Expanding on the Angular 2 Quickstart"
categories: angular
---

Not so long ago the Angular 2 team released a [quickstart tutorial](https://angular.io/docs/js/latest/quickstart.html) to give people a chance to play with the new version of the framework. The tutorial itself is pretty skinny, likely because the framework is still subject to change in big ways. Nonetheless, there have been [screencasts](https://egghead.io/lessons/angularjs-angular-2-hello-world-es5), [more screencasts](http://www.infoq.com/presentations/angularjs-2), [tutorials](http://blog.thoughtram.io/angular/2015/03/27/building-a-zippy-component-in-angular-2.html), and [more tutorials](https://www.airpair.com/angularjs/posts/creating-components-p3-angular2-directives) covering this or that part of the framework.

All the same, though, when I set out to build my own basic Angular 2 app, it was hard to find one blog post that answered all my questions -- stuff like how to inject services into components, how to assign attributes to components, the basics of the [template syntax](https://egghead.io/lessons/angularjs-angular-2-template-syntax), as well as what a TypeScript workflow looked like. So in the interest of answering these questions for others who might be interested, I will walk through a basic guestbook app. If reading the code straightaway is more helpful, you can find it [here](https://github.com/enocom/angular2-guestbook). In short, this post is meant to be a minor expansion on the Angular 2 quickstart.

Before we get into the code, I found [WebStorm 10](https://www.jetbrains.com/webstorm/) to work nicely with TypeScript. Even though the IDE offers to compile `.ts` files, I instead ran the TypeScript compiler from the command line. The [Atom](https://atom.io/) editor from GitHub paired with [atom-typescript](https://github.com/TypeStrong/atom-typescript) looks like a nice alternative.

Aside from an editor, you will also need a few command line tools: `tsc`, a TypeScript compiler, `tsd`, a TypeScript definition manager, and `http-server`, a simple server which we'll use to view our app. To install these tools, run:

``` bash
npm install -g tsd
npm install -g typescript
npm install -g http-server
```

To start, let's create a working directory and fetch the Angular 2 type definitions:

``` bash
mkdir guestbook && cd guestbook
tsd query angular2 --action install
```

This will create a `typings` directory with an `angular/angular2.d.ts` file, which we will use in a moment. Next, we need to create our application files:

``` bash
touch Guestbook.ts index.html
```

And with that, we are ready to start compiling our TypeScript:

``` bash
tsc --watch -m commonjs -t es5 --emitDecoratorMetadata Guestbook.ts
```

Note that whenever we create a new file, we will need to restart the `tsc` process for it to pick up the new file.

We now need to specify the TypeScript definitions for Angular at the top of `Guestbook.ts`:

``` javascript
/// <reference path="typings/angular2/angular2.d.ts" />
```

The last involved part of setting up Angular 2 is bootstrapping its dependencies in the `index.html`. Note that we're using JSPM to load our `Guestbook` code onto the page.

``` html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8"/>
  <title>Angular 2 Quickstart Expanded</title>
  <script src="https://github.jspm.io/jmcriffey/bower-traceur-runtime@0.0.87/traceur-runtime.js"></script>
  <script src="https://jspm.io/system@0.16.js"></script>
  <script src="https://code.angularjs.org/2.0.0-alpha.22/angular2.dev.js"></script>
</head>
<body>

  <!-- we'll write this component below -->
  <guestbook></guestbook>

  <script>System.import('Guestbook');</script>
</body>
</html>
```

As a side note, at the time of this post, I tried bumping to the newest Angular 2 (version 2.0.0-alpha.25), but ran into some trouble with injecting services. So we'll stick with the version suggested by the Angular team on their quickstart.

And with that, we are ready to start writing an Angular 2 app. Let's start by getting a basic component on the page. We'll import the `Component`, `View`, and `bootstrap` functions first before defining the rough outline of our guestbook component.

``` javascript
import {
  Component,
  View,
  bootstrap
} from 'angular2/angular2';

@Component({
  selector: "guestbook"
})
@View({
  template: "<h1>My Cool Guestbook</h1>"
})
class Guestbook {

}

bootstrap(Guestbook);
```

If we launch our `http-server` from our working directory and visit `localhost:8080`, we should see that everything is working. As we go, keep an eye on the `tsc` process, as it will report any errors.

And that's about where the Angular 2 quickstart leaves off. Let's make this component more interesting by injecting a service into it.

Any guestbook will have a list of names, so let's update our component to show that list. First, we'll update the template and pull it out into a separate file. We'll need to change the `View` annotation to point to a template URL. If this were Angular 1, we would want to add an `ng-repeat` somewhere for that list of names. In Angular 2, `ng-repeat` has become `For`.

``` javascript
import {
  Component,
  View,
  For,        // <---- This is how we bring in For
  bootstrap
} from 'angular2/angular2';


@View({
  templateUrl: "Guestbook", // <--- point to a template URL
  directives: [For] // <---- And this is how we tell the view, we'll use `For`
})
```

Then create a `Guestbook.html` file in the same directory with the following:

``` html
<div>
  <h1>My Cool Guestbook</h1>

  <ul>
    <li *for="#guest of guests">{{guest}}</li>
  </ul>
</div>
```

Assuming we have a collection called `guests` on our `Guestbook` component, the template simply loops over them and create an `li` for every guest name. The syntax looks a little strange at first, but in effect we're just using the Angular 2 version of `ng-repeat='guest in guests'` The asterix denotes an Angular directive and the pound sign, which we'll see again below, creates a data binding.

Now that we have our template wired up correctly, let's add the `guests` field to our component:

``` javascript
class Guestbook {
  guests: Array<string>;

  constructor() {
    this.guests = ["Alice", "Bob"];
  }
}
```

Now, if we return to `localhost`, we will see our list of guests on the page.

To make this more interesting, let's suppose we have an API endpoint which will return guest names, so we'll write a service to fetch those names and inject it into the `Guestbook` component.

First, for the service. Make a new file called `GuestService.ts` with the following code and then restart `tsc`:

``` javascript
class GuestService {
  guests: Array<string>;

  constructor() {
    // hard coding guest names for now
    this.guests = ["Alice", "Bob"];
  }
}

export { GuestService };
```

There is nothing special about this service. It's a plain JavaScript object. We assign names to the `guests` array in the constructor and we export the service for importing elsewhere.

Now, back in the `Guestbook` component, we can import the service. Just below the Angular 2 imports, add the following line:

``` javascript
import {GuestService} from "./GuestService";
```

With the service now available to us, we have to tell Angular to inject it into the `Guestbook` component for us. To do that, we'll update the `@Component` annotation.

``` javascript
@Component({
  selector: 'guestbook',
  injectables: [GuestService] // <--- here we configure our injectable objects
})
```

And then in the `Guestbook` constructor, we specify the `GuestService` as an argument:

``` javascript
class Guestbook {
  guests: Array<string>;

  constructor(service: GuestService) {
    this.guests = service.guests;
  }
}
```

Now, reload `localhost` (watching for possible errors in the `tsc` process), and our guest names are now being delivered by the service. Our app otherwise looks exactly the same. From here, it would be trivial to fetch the guests across a network call, rather than hardcoding them, but we'll leave that exercise for another day.

To take our guestbook features on step further, let's allow users to add new guests to the guestbook.

Best practice would require us to use a form, but for the sake of simplicity, let's sidestep dealing with forms. For the curious reader, [here](https://youtu.be/fRJIJU-K6o8) is a good talk on forms in Angular 2. For our purposes here, we will use just an `input` and a `button`.

First, we will update the `Guestbook.html` template:

``` html
<div>
  <h1>My Cool Guestbook</h1>

  <!-- new stuff follows -->

  <input type="text" #new-guest />

  <button (click)="addToGuestbook(newGuest.value)">
    Add to Guestbook
  </button>

  <!-- end new stuff -->

  <ul>
    <li *for="#guest of guests">{{guest}}</li>
  </ul>
</div>
```

The changes here jump right out to a developer versed in Angular 1. There is no `ng-model`. Instead we have `#new-guest` with the pound sign indicating a data binding. Also, there is no `ng-click`. Instead, it's `(click)` (which is shorthand for `bind-click`). Otherwise, the idea is the same. We bind user input to a `new-guest` object and pass the value of that object (`value` being a property that Angular gives us) to an `addToGuestbook` function. Note the difference between `#new-guest` and `newGuest`. Angular makes the conversion between kebab case and camel case automatically to ensure valid JavaScript variable names. This is not a new feature.

To make this work with our component, we need to add the `addToGuestbook` function as well as extend our GuestService to accept newly registered guests:

``` javascript
class Guestbook {
  guests: Array<string>;
  service: GuestService;

  constructor(service: GuestService) {
    this.service = service;
    this.guests = service.guests;
  }

  addToGuestbook(guestName: string) {
    this.service.register(guestName);
  }
}
```

The `register` function on the `GuestService` is plain and simple:

``` javascript
class GuestService {
  guests: Array<string>;

  constructor() {
    this.guests = ["Alice", "Bob"];
  }

  // a simple way to register guests
  register(guestName: string) {
    this.guests.push(guestName);
  }
}

export { GuestService };
```

Now, when we reload the page, we can now add guests to our list. Note that the input field remains populated after successfully adding a guest, but I will leave this problem as an exercise for the reader.

As a finishing touch to our awesome guestbook, let's add a toggleable message showing the number of guests in our guestbook. Rather than write the toggleable message into our `Guestbook` component, we will make it generalizable and reusable. In other words, let's make a new component.

Our component will take the form of a button with some custom text. When the user clicks that button, a message will appear on the page. If the user clicks the button a second time, the message will disappear. We'll call our new component "toggle-message."

Let's start by updating the `Guestbook.html` template to include our new `toggle-message` component:

``` html
<div>
  <h1>My Cool Guestbook</h1>

  <input type="text" #newGuest />

  <button (click)="addToGuestbook(newGuest)">
    Add to Guestbook
  </button>

  <ul>
    <li *for="#guest of guests">{{guest}}</li>
  </ul>

  <!-- here's the new component -->
  <toggle-message
    message="{{currentGuestCount()}}"
    button-text="Guest Count">
  </toggle-message>
</div>
```

We'll make the message and the button text configurable. The message will be the return value of a new function on our `Guestbook` component. Let's write that first:

``` javascript
class Guestbook {
  // ...

  currentGuestCount(): number {
    return this.service.currentCount();
  }
}
```

The new method on the `GuestService` is also trivial:

``` javascript
class GuestService {
  // ...
  currentCount() {
    return this.guests.length;
  }
}
```

While we're updating the `Guestbook` component, we also need to tell Angular about the component's dependency on `toggle-message`. To do that, we'll import the `toggle-message` component and update the `@View` annotation:

``` javascript
// ...
import {ToggleMessage} from "./ToggleMessage";

//...
@View({
  templateUrl: 'Guestbook',
  directives: [For, ToggleMessage] // <--- adding ToggleMessage to the list
})
```

Now with all the configuration out the way, we're ready to write the `toggle-message` component. First, create a template `ToggleMessage.html` with the following:

``` html
<div>
  <button (click)="toggleMessage()">{{buttonText}}</button>
  <h1 [hidden]="!toggle">{{message}}</h1>
</div>
```

Again, we have a click handler which calls a function, this time `toggleMessage()`. We interpolate the `buttonText` and `message` as passed in by the parent component. What's new about this example is the use of `[hidden]`, which sets the hidden property of the `h1` DOM element and sets it to the negated value of the `toggle` variable. Again, see [this screencast](https://egghead.io/lessons/angularjs-angular-2-template-syntax) for more detail on the square bracket notation.

With the template done, let's now write the actual component and complete this app. Create a new file called `ToggleMessage.ts` and add the following:

``` javascript
import {Component, View} from "angular2/angular2";

@Component({
  selector: 'toggle-message',
  properties: {                   // <---- this is the interesting part
    'message': 'message',
    'buttonText': 'button-text'
  }
})
@View({
  templateUrl: 'ToggleMessage'
})
export class ToggleMessage {
  toggle: boolean;

  constructor() {
    this.toggle = false;
  }

  toggleMessage() {
    this.toggle = !this.toggle;
  }
}
```

As usual, we import `Component` and `View` to describe the selector and the location of the template. We also implement a basic toggleable field using a boolean `toggle` variable.

The most interesting part is how we tell Angular about the component's properties, i.e., the `properties` key in the `Component` annotation. The keys on the right, e.g., `button-text` will be passed in by the component's parent, while the values on the left will be the local bindings available for us in the template of `toggle-message` itself.

Now restart the `tsc` process for it to compile the `ToggleMessage` component and visit `localhost` to view our amazing guestbook app in all its glory. If we click on the "Guest Count" button, our count will appear. Clicking again will hide the count. If we add a new guest, the count will be updated.

Granted there are still lots of things to explore about Angular 2 and the framework is still undergoing major changes, but the bigger ideas seemed to have settled. In my view, Angular 2 takes the best of component-based frameworks like ReactJS, while maintaining the excellent testability and use of dependency injection. I think Angular 2 is going to be a fantastic upgrade.
